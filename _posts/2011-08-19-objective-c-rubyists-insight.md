---
layout: post
title: "Objective-C: Rubyist's insight"
published: false
---


## Intro
![](http://habrastorage.org/storage1/9e3f6a48/f3ee30bb/18d4ea06/4208aa40.png)

Before I started learning Objective-C, I was coding in PHP, then in Python and Ruby. I like Ruby most of all. I like it for its simplicity, pithiness, and in the same time, for its powerfulness. About a week later (this post was originally written in Russian on 26th Jul -- [link](http://habrahabr.ru/blogs/macosxdev/124974/)) I made hackintosh on my PC up and running (now I own a MacBook Early 2008 Black). It was OS X Lion GM. I knew that the applications for Macs and iPhones are written in Objective-C, I even tried to learn Objective-C, but without a Mac this learning was unpleasant. I installed Xcode 4.2, I have written several simple console apps. And each time I followed some tutorial, or just tried to write the code myself, I noticed that Ruby and Objective-C have many common parts (and it's very logical -- both languages are influenced by [Smalltalk](http://en.wikipedia.org/wiki/Smalltalk)), although these langs have different purpose. 

![](http://habrastorage.org/storage1/b9449c86/329c5f48/15953b7c/328d6257.png)

So I'll describe some things that will help Rubyists to understand Objective-C. These things are mostly theoretical, without the snippets of code. Apologize if some thing are not explained The C Way, I came from The World of Ruby :D
And sorry for my not good enough English.

## 1. The Dynamicity
We all know that Ruby is a dynamic programming language. Okay but what's with Objective-C? Isn't it just a superset of C? Not quite. Objective-C makes many decisions (e.g. sending a message, etc) at runtime, not at compile time.
## 2. The Object Model
Everything is an object in Ruby. This isn't always true for Objective-C. As Objective-C is superset of C, base data types (like `char`, `float`, `double`, `int`, `struct`) aren't objects. However Foundation.framework provides the functional wrappers over them and such ordinary for Rubyists things like Strings, Mutable Arrays, Hashes, etc.
## 3. Sending messages
In Ruby, when we call an object method we actually send a message to the object

    object.send :method, args

That is, the method call is converted to sending a message. The same thing happens with Objective-C

    [object method];

This is exactly sending of a message, not a method call, 'cause if we call a non-existent method we don't get any errors at compile time, only at runtime (that proves that Objective-C is dynamic too). At compile time `[object method]` is converted to C-call of `objc_sendMsg(id object, SEL selector)` function.
If you use [clang3](http://clang.llvm.org/), it can return errors at compile time.

_BTW, in the terminology of C++, methods in both Ruby and Objective-C are virtual._
## 4. Non-strict typing
Each developer in Ruby knows that with Ruby he doesn't care about types of vars. But not each knows that with Objective-C he doesn't care about types of vars too. If we don't know the type of an object, or we just don't wanna care about it, we can just specify the type of object as `id`, that means that variable can hold value of any type. `id` is kinda like a pointer to any object.
Okay it's well. But what with C-types? No, it doesn't work for C-types, only for Objective-C objects. However we can use Foundation's wrappers over C types like `NSNumber`, `NSString`, `NSArray`, etc.
## 5. Class declaration, instances of objects and properties
In Ruby we just define a class, in Objective-C we have to write both interface and realization for our class. In Ruby we can define any i-var from any method, in Objective-C we have to declare all i-vars we are going to use in an interface file.
An objects are created easily in both langs. In Ruby we just send `new` message to the class

    myObject = MyClass.new

In Objective-C we need to allocate memory for the object (`alloc`) and then to initialize it (with `init` or `initWith...` method)

    MyClass * myObject = [[MyClass alloc] init];

If we wanna get access i-vars we define getter and setter methods. In Ruby we just write `attr_accessor :var` inside class definition, in Objective-C we write `type * variableName` inside i-vars definition block, `@property(retain) type * variableName` after methods definition, and `@synthesize variableName` inside class realization file.

_As Objective-C is a superset of C, we ought to care about memory. LLVM compiler however provides us with one interesting feature -- Automatic Reference Counting, which takes care about freeing memory. Xcode 4.2 and newer uses LLVM compiler by default, so we can not care about memory. But for deeper Objective-C understanding I highly recommend you to spend some time understanding memory management._
## 6. Protocols and mixins
In Ruby we can define modules and then mix 'em into our classes, in a such way something like multiple inheritance is realized. In Objective-C we haven't modules. But we have protocols.

> An informal protocol is a list of methods that a class can opt to implement. It is specified in the documentation, since it has no presence in the language. Informal protocols often include optional methods, which, if implemented, can change the behavior of a class. For example, a text field class might have a delegate that implements an informal protocol with an optional method for performing auto-completion of user-typed text. The text field discovers whether the delegate implements that method (via reflection) and, if so, calls the delegate's method to support the auto-complete feature.

And for example we can implement a method that can take any object that realizes `Serializable` protocol and be sure that the object can respond to `serialize` message.
## 7. Categories and class extension
If we wanna extend a Ruby class, we can simply write

    class String
      def hello_world
        "Hey"
      end
    end

That way we added `hello_world` method to `String` class (but we can do so in many ways). In Objective-C we can do something similar using categories

    @interface NSString (HelloWorld)
    + (NSString *)helloWorld;
    @end
    @implementation NSString (HelloWorld)
    + (NSString *)helloWorld
    {
      return @"Hey";
    }
    @end

Usually categories are saved in files named like `ClassToBeExtended+CategoryName` -- e.g. `NSString+HelloWorld.h` and `NSString+HelloWorld.m`, and then are imported.

__Here are several things that simplified my understanding of Objective-C. Hope this article is helpful for you. Good luck!__
