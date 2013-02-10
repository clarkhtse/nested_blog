---
layout: post
title: "Analytics: Analyzing tracked events"
published: true
hn: http://news.ycombinator.com/item?id=5182625
---

In previous article, I've shown two essential ways the events get tracked. 

Now, I would like to focus on the latter event of tracking — storing all the event data, and analyzing it ad-hoc afterwards. 

It is important to analyze data in the fastest fashion, while preserving the power. That's the main concern of this post. 

We'd explore more about that using the shop tracking system as an example. It would track only one kind of event — orders. To keep the samples simple, we'll only calculate these metrics:

* average purchase total
* top products

The structure of the event document would be as follows:

* total — float order total
* line items — array with elements like:
  * sku — string product sku
  * price — float item price

For the sake of this example, we will perform different analysis methods against a set of 1,000,000 of such order events, with each having 5 line items on average.

We'll use MongoDB as schema-less documents are the fit for storing events and also because of its powerful analytics functionalities. Worth noticing that those features will mainly be described in light of their application to described tasks.

[Sample sources][sample]

### The naive way

The most obvious thing to do is to query all the events and perform our analysis in Ruby.

```ruby
def average_purchase
  all.map(&:total).sum / all.count
end

def top_products
  all.map(&:line_items).flatten.group_by(&:sku).map { |i| [i.first, i.last.count] }.sort_by(&:last).reverse
end
```

While it seems pretty simple, it does in no way seem performant.

| naive        | time      |
| ------------ | --------- |
| avg purchase | 119,775ms |
| top products | *canceled after 2+ hours of waiting* |
| **total**    | **canceled after 2+ hours of waiting** |

And not only it is being slow, it is also being highly RAM-consuming. The overhead of transmitting 1M documents from Mongo to Ruby process and then storing that in RAM is hurting.

### Evaluating JS code on MongoDB server

What was the worst point of previous case? Right, transmitting the data and storing it in memory.

If there only was a way to evaluate arbitrary code on MongoDB side… Well, there is. It's called [db.eval()][mongo-db-eval].

Using it, we can execute any JavaScript as if we were running it inside `mongo`. It effectively reduces the overhead of transmitting and introduces overhead of running a JavaScript VM.

```ruby
def average_purchase
  result = collection.database.command("$eval" => <<-EOJS)
    var orders = db.order_events.find(),
        order_count = orders.count(),
        total_sum = 0;

    orders.forEach(function(order) {
      total_sum += order.total;
    });

    return total_sum / order_count;
  EOJS

  result['retval']
end

def top_products
  result = collection.database.command("$eval" => <<-EOJS)
    var orders = db.order_events.find(),
        products = {};

    orders.forEach(function(order) {
      order.line_items.forEach(function(item) {
        var sku = item.sku;
        if (!products[sku]) products[sku] = 0;
        products[sku] += 1;
      });
    });

    return products;
  EOJS

  result['retval'].to_a.sort_by(&:last).reverse
end
```

It is more of the code, but the results are better:

| db.eval()    | time         |
| ------------ | ------------ |
| avg purchase | 8,228ms      |
| top products | 42,399ms     |
| **total**    | **50,627ms** |

The disadvantage of using it is the write lock being placed while it's running and overhead of executing it in JavaScript VM.

### Map/reduce

[Map/reduce][mapreduce] is a programming model for processing large data sets. One of highlights is distributing processing of computations across the cluster. Not only that, it also has the benefits of `db.eval()`. You can learn more about how MongoDB implements it [here][mongo-mr].

Worth noticing that map/reduce is not always a suitable substitution for `db.eval()`. The tasks we are trying to accomplish, on another hand, are the perfect match for using it.

```ruby
def average_purchase
  map = "function() { emit('avg', this); }"

  reduce = %Q{
    function(key, values) {
      var result = { sum: 0, count: 0 };
      values.forEach(function(value) {
        result.sum += value.total;
        result.count += 1;
      });
      return result;
    }
  }

  finalize = %Q{
    function(key, value) {
      value.avg = value.sum / value.count;
      return value;
    }
  }

  map_reduce(map, reduce).out(inline: 1).finalize(finalize).to_a.first['value']['avg']
end

def top_products
  map = %Q{
    function() {
      this.line_items.forEach(function(item) {
        emit(item.sku, item);
      });
    }
  }

  reduce = %Q{
    function(key, values) {
      var result = { purchases: 0 };
      values.forEach(function(value) {
        result.purchases += 1;
      });
      return result;
    }
  }

  items = map_reduce(map, reduce).out(inline: 1).to_a
  items.map { |doc| [doc['_id'], doc['value']['purchases']] }.sort_by(&:last).reverse
end
```

Speed was generally worse compared to `db.eval()`. It is because  map/reduce splits tasks in two phases. If map/reduce is run on a shard cluster, tasks would be dispatched to each shard, thus making it faster. (Map/reduce wasn't meant to be damn fast anyway.)

| map reduce   | time          |
| ------------ | ------------- |
| avg purchase | 20,498ms      |
| top products | 112,035ms     |
| **total**    | **132,534ms** |

It also suffers from the same thing as `db.eval()` does — map/reducing places the write lock while it runs.

### Aggregation framework

While map/reduce is truly awesome, the amount of code to aggregate simple things is often overwhelming.

Assuming you're familiar with UNIX shell you would also wonder why can't you easily chain different operations in map/reduce? What if you wanted to do something like:

    match_criteria | sort | limit | group

Doing that with map/reduce would be a pain in ass.

Ok, that sucks, but what are the alternatives? Well, MongoDB 2.1 and higher ships with something called ["aggregation framework"][mongo-aggr]. Essentially, it is chainable map/reduce with common use cases implemented and optimized upfront. It is not a tutorial on it, so refer to the docs when you need. I'd just go by showing how neat it is.

```ruby
def average_purchase
  collection.aggregate([
    {"$group" => {"_id" => "avg", "avg" => {"$avg" => "$total"}}}
  ]).first['avg']
end

def top_products
  collection.aggregate([
    {"$unwind" => "$line_items"},
    {"$project" => {"item" => "$line_items"}},
    {"$group" => {"_id" => "$item.sku", "purchases" => {"$sum" => 1}}},
    {"$sort" => {"purchases" => -1}}
  ]).map { |doc| [doc['_id'], doc['purchases']] }
end
```

Running it, we can notice great speed improvement compared to any of previously discussed ways:

| aggregation  | time         |
| ------------ | ------------ |
| avg purchase | 2,511ms      |
| top products | 29,146ms     |
| **total**    | **31,658ms** |

### Possible optimization techniques

Whatever method you end up using, it is wise to apply some optimization to metric calculation.

Computing your metrics on every access to them would be super-dumb. Instead, once calculated, their value should be cached and always returned unless the metric dependencies change.

In our examples, caches should be invalidated only when a new order is tracked.

### Comparison

|     | naive | db.eval() | map/reduce | aggregation |
| --- | ----- | --------- | ---------- | ----------- |
| avg purchase | 119,775ms | 8,228ms | 20,498ms | 2,511ms |
| top products | *canceled after 2+ hours of waiting* | 42,399ms | 112,035ms | 29,146ms |
| total | *canceled after 2+ hours of waiting* | 50,627ms | 132,543ms | 31,658ms |
| speed | exceptionally low | high | medium | super high |
| memory usage | high | low | low | low |
| data transfer overhead | yes | no | no | no |
| JS VM overhead | no | yes | yes | yes |
| write lock | no | yes | yes | yes |
| easily distributed | — | no | yes | yes |
| optimized | — | — | no | yes |
| output limit | — | — | 16Mb | 16Mb |
| chainable | — | — | no | yes |

### Conclusion

Using MongoDB aggregation framework proves to be the fastest option you have. For sure, it is powerful enough to perform simple as well as more advanced analysis of data. It scales well and is simple to write.

If something more sophisticated needed than it's possible to do with aggregation framework, you may use map/reduce, trading off the performance and loosing easy chainability. Scalability still remains the point.

If your task is really simple but is impossible to do with aggregation framework or is very slow under map/reduce, it might be desirable to give `db.eval()` a shot. Cons? Doesn't scale.

Querying all the documents and performing the analysis right in your programming language is the last resort. It's a loose by all means and generally it should be used only when neither of other options leads to desired result.

[Sample sources][sample]

*I am not the expert at analytics, just the curious one. If I got something in a wrong way and/or advice the bad thing, feel free to let me know about that.*

[mapreduce]: http://en.wikipedia.org/wiki/MapReduce
[mongo-db-eval]: http://docs.mongodb.org/manual/reference/method/db.eval/#db.eval
[mongo-mr]: http://docs.mongodb.org/manual/applications/map-reduce/
[mongo-aggr]: http://docs.mongodb.org/manual/applications/aggregation/
[sample]: http://github.com/goshakkk/simple-analytics-demo
