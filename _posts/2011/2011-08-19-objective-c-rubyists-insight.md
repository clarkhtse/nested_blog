---
layout: post
title: "Objective-C: Rubyist's insight"
published: true
---


## Intro
![](http://habrastorage.org/storage1/9e3f6a48/f3ee30bb/18d4ea06/4208aa40.png)

Before I started learning Objective-C, I did some PHP, then Python and
Ruby. I like Ruby most of all. I like it for its simplicity, pithiness,
and at the same time, for its power. About a week later (this post was
originally written in Russian on 26th Jul --
[link](http://habrahabr.ru/blogs/macosxdev/124974/)) I got hackintosh on
my PC up and running (now I own a MacBook Early 2008 Black). It was OS X
Lion GM. I knew that the applications for Macs and iPhones are written
in Objective-C, I even tried to learn Objective-C, but without a Mac
that kind of learning was unpleasant. I installed Xcode 4.2, I have
written several simple console apps. And each time I followed some
tutorial, or just tried to do stuff myself, I kept noticing similarities
between Ruby and Objective-C (and it's very logical -- both languages
are influenced by [Smalltalk](http://en.wikipedia.org/wiki/Smalltalk)),
although they serve different purpose.

![](http://habrastorage.org/storage1/b9449c86/329c5f48/15953b7c/328d6257.png)

So I'll describe a few things that will help Rubyists to understand
Objective-C. These things are mostly theoretical, without the snippets
of code. I apologize if some thing are not explained The C Way, I came
from The World of Ruby. And sorry for my not good enough English.

## 1. Dynamicity

We all know that Ruby is a dynamic programming language. Okay but what's
with Objective-C? Isn't it just a superset of C? Not quite. Objective-C
makes many decisions (e.g. sending a message, etc) at runtime and not at compile time.

## 2. Object model

Everything is an object in Ruby. This isn't always true for Objective-C.
As Objective-C is superset of C, basic data types (like `char`, `float`,
`double`, `int`, `struct`) aren't objects. However Foundation.framework
provides the functional wrappers around them as well as such ordinary
things for Rubyists like Strings, Mutable Arrays, Hashes, etc.

## 3. Sending messages

In Ruby, when we call an object method we actually send a message to the
object:

```ruby
# what you do
object.method args
# what ruby actually does
object.send :method, args
```

That is, the method call is converted to passing a message. The same
thing happens with Objective-C:

```objective-c
# what you do
[object method];
# what actually happens
objc_msgSend(object, @selector(method));
```

This is precisely passing of a message and not a method call, because if
we call a non-existent method we don't get any errors at compile time,
only at runtime (that proves that Objective-C is dynamic too). At
compile time `[object method]` is converted to C-call of
`objc_sendMsg(id object, SEL selector)` function.

If you use [clang3](http://clang.llvm.org/), it can make it throw errors
when compiling.

_BTW, in the terminology of C++, methods in both Ruby and Objective-C are virtual._

## 4. Non-strict typing

Each developer in Ruby knows that they don't have to care about types.
But not everyone knows that with Objective-C they can forget about types
as well. If the type of the object is unknown to us yet, we can just
specify the type of object as `id`, that means that variable can hold
value of any type. `id` is kinda like a pointer to any object.

Okay it's all good. But what with C-types? No, it doesn't work with
C-typesi. However we can use Foundation's wrappers over C types like
`NSNumber`, `NSString`, `NSArray`, and so forth.

## 5. Class declaration, instances of objects, and properties

In Ruby we just define a class, in Objective-C we have to write both
interface and implementation for the class. In Ruby we can define any i-var from any method, in Objective-C we have to declare all i-vars we are going to use in an interface file.
An objects are created easily in both langs though. In Ruby we just send
`new` message to the class:

```ruby
myObject = MyClass.new
```

In Objective-C we need to allocate memory for the object (`alloc`) and
then to initialize it (with `init` or `initWith...` method):

```objective-c
MyClass * myObject = [[MyClass alloc] init];
```

If we want to access the i-vars from the outside, we define getter and
setter methods. In Ruby we just write `attr_accessor :var` inside class
definition, in Objective-C we write `type * variableName` inside i-vars
definition block, `@property(retain) type * variableName` after methods
definition, and `@synthesize variableName` inside class implementation
file.

_As Objective-C is a superset of C, we ought to care about memory. LLVM compiler however provides us with one interesting feature -- Automatic Reference Counting, which takes care about freeing memory. Xcode 4.2 and newer uses LLVM compiler by default, so we can not care about memory. But for deeper Objective-C understanding I highly recommend you to spend some time understanding memory management._

## 6. Protocols and mixins

In Ruby we can define modules and then mix 'em into the classes, in a
such way something like multiple inheritance is realized. In Objective-C
we haven't modules. But we have protocols.

> An informal protocol is a list of methods that a class can opt to implement. It is specified in the documentation, since it has no presence in the language. Informal protocols often include optional methods, which, if implemented, can change the behavior of a class. For example, a text field class might have a delegate that implements an informal protocol with an optional method for performing auto-completion of user-typed text. The text field discovers whether the delegate implements that method (via reflection) and, if so, calls the delegate's method to support the auto-complete feature.

And for example we can implement a method that can take any object that realizes `Serializable` protocol and be sure that the object can respond to `serialize` message.

## 7. Categories and class extension

If we want to extend a Ruby class, we can simply write:

```ruby
class String
  def hello_world
    "Hey"
  end
end
```

That way we added `hello_world` method to `String` class (but we can do
so in many ways). In Objective-C we can do something similar using
categories:

```objective-c
@interface NSString (HelloWorld)
+ (NSString *)helloWorld;
@end

@implementation NSString (HelloWorld)
+ (NSString *)helloWorld
{
  return @"Hey";
}
@end
```

Usually categories are saved in files named like `ClassToBeExtended+CategoryName` -- e.g. `NSString+HelloWorld.h` and `NSString+HelloWorld.m`, and then are imported.

Here are several things that simplified my understanding of
Objective-C. Hope this article is helpful for you. Good luck!
