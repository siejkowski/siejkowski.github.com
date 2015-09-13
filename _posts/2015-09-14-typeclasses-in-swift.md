---

layout: layout
title: "Road to Mobilization: Typeclasses in Swift, Haskell and Scala"
author: Krzysztof Siejkowski
date: 2015-09-14
keywords: mobilization, mobilization 2015, swift, typeclasses, haskell, scala, typeclasses in swift
description: An article describing what are typeclasses and how can we implement them in Haskell, Scala and Swift. 

---
# [{{ page.title }}]({{ page.url }}) 

Few days ago I've got an email confirming that my presentation proposal at [Mobilization 2015](http://2015.mobilization.pl/) conference was accepted.

I am so excited.

I've co-organized two conferences ([Codepot 2015](https://codepot.pl) and [Warsjawa 2014](http://warsjawa.pl)) and volunteered at some others, but I've never given a conference talk. I've always wanted to. For one, there's something in the presentation format that makes it easy to break the ice with a topic. Two, good presentations are about storytelling, and I love good stories. Three, it gives me an excuse to play with [Deckset](http://www.decksetapp.com/), which is one of my favourite apps. Three is enough to get me sold!

I'll talk about how to apply a few ides from functional programming to the everyday iOS development. It's a wide and popular topic that most people are already tired of, so I'm gonna pick and choose just my favourite concepts from Scala and Haskell and verify whether they apply in the world of Cocoa, Objective-C and Swift. There's a lot of potentially fruitful ideas, but only a few will make it to the actual presentation. It's gonna be a long road full of constant learning, occasional epiphanies, random bursts of euphoria and long periods of research and doubts. I can't wait, let's hit it!

Here comes the first encounter on the road to [Mobilization](http://2015.mobilization.pl/). Please, meet a typeclass. 

## What is a typeclass?

Typeclass is a Haskell way of creating the type composition in the world without inheritance. It allows to define a desired behavior in a form of function signatures. The concrete implementation is provided separately for all the required types. In other words, it splits the familiar object-oriented encapsulation (data and functionality together) in two separate parts: data and functionality. At the same time typeclass defines a contract that we can build upon. It's like defining the same function multiple times - each time the only thing that differs is the type in the signature - and making it possible to use this function in some other place without specifying which one of its variants should be used. The compiler guesses it for us.

Doesn't it sound like Swift or Objective-C protocols? Well, it does. It's no surprise, because they're all fueled by same basic idea. This is the first and arguably the most important thing to know about typeclasses - although they do have a word _class_ in their name, they are more similar to protocols. Actually, they are far enough from classes so they can independently coexist with them, orthogonal to each other.

Typeclasses, being protocol cousins, are used in similar fashion: to express a feature that spreads across multiple types. In Haskell there are typeclasses like `Eq`, expressing that things are equatable, or `Ord`, expressing that they are sortable, or `Functor`, expressing that they can be mapped over.

If you've seen [WWDC 2015 session "Protocol-Oriented Programming in Swift"](https://developer.apple.com/videos/wwdc/2015/?id=408), you're gonna feel at home (or run away screaming, depending on how you liked it). One thing to notice: while in a more strict functional language, namely Haskell, typeclass is a part of its type semantics, in Swift or Scala typeclass is more of a design pattern. We're using their native type semantics to achieve similar effects.

Enough with introduction. Let's define a typeclass so we can more easily grasp what's going on. 

## _Things that can be encoded in Ceasar cipher_

Have you heard of [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher)? It is a very basic cryptography method: we express anything as a string and than we shift each letter by a fixed number of places in the alphabet. So, for 3-letter Ceasar cipher we write D instead of A, E instead of B, F instead of C and so on.

Our typeclass is gonna describe the ability to be expressed in a Ceasar cipher form. It's gonna be based on position of particular character in the ASCII table. For the sake of simplicity I'll ignore the fact that in the ASCII table there are some special characters after the last letters of the alphabet. No one is actually sending messages to Roman legions anymore, so no one is gonna get surprised by some `%$#`. 

Here is the `Ceasarable` typeclass defined in Haskell:

{% highlight haskell %}

    class Ceasarable c where 
        toCeasar :: c -> String

{% endhighlight %}

Just to mess with object-oriented minds, it uses keyword `class` to kick off its definition. Then it declares one function signature: `toCeasar`. The function takes one argument of any type and returns a string, presumably with the cipher shift applied. This is our desired behavior. It must be implemented (with the actual type instead of `c`) by the typeclass instances. 

How does it gonna look like in Scala? We're gonna use Scala's type semantics. The most obvious way is to use trait:

{% highlight scala %}
 
    trait Ceasarable[T] {
      def toCeasar(input: T): String
    }

{% endhighlight %}

The translation is straightforward. Any type in Haskell becomes a generic parameter in Scala. The signature is the same. 

In Swift the closest thing to traits/interfaces are protocols, so let's search no further:

{% highlight swift %}

    protocol Ceasarable {
      typealias T
      static func toCeasar(input: T) -> String
    }

{% endhighlight %}

Apart from minor syntax differences, like associated type instead of generic parameter, it's the same as in Scala. Not surprising, as those two languages share a lot of similarities (and I mean like, a lot). One thing to notice is the use of `static` method. Why is it static? Because we want to emulate the split between the data and behavior. If the method is not static, than it can use the instance data and there is (in our simple case) no need to pass `input` at all. An instance method declaration would make the typeclass a little more object-oriented, and there's nothing wrong with that, but for now let's stick to the original idea. We're providing the implementation for a type, not for an instance. 

## When I grow up I wanna be a typeclass!

Once we've defined what we expect, it'd be nice to provide some actual implementations for chosen types. This way we'd be able to use the behavior. Let's choose two simple types to work with: strings and integers. In Haskell, the implementation is provided by defining the concrete typeclass instance:

{% highlight haskell %}

    {-# LANGUAGE TypeSynonymInstances, FlexibleInstances #-}
    
    instance Ceasarable String where
        toCeasar x = [toEnum $ fromEnum c + 3 | c <- x]
    
    instance Ceasarable Integer where
        toCeasar x = [toEnum $ fromEnum c + 3 | c <- show x]

{% endhighlight %}

Keyword `instance` means that the implementation is coming. For String, we map over characters, get their ASCII numbers with `fromEnum`, add three and than encode again with `toEnum`. For Integer we just express it as String using `show` and we do exactly the same. 

In Scala things get a little weird, so feel free to skip over the details. The `Ceasarable` behavior is enclosed in the `object` and marked as implicit. This way it can be implicitly passed to the place we want to use it:

{% highlight scala %}

    object Ceasarable {
    
      implicit object CeasarableInt extends Ceasarable[Int] {
        override def toCeasar(input: Int): String = {
          s"$input".map(_ + 3).map(_.toChar).mkString
        }
      }
    
      implicit object CeasarableString extends Ceasarable[String] {
        override def toCeasar(input: String): String = {
          input.map(_ + 3).map(_.toChar).mkString
        }
      }
    
    }

{% endhighlight %}

The `object` scope and `implicit` passing are part of Scala peculiarities, no need to dive deeper in them. If you really want to, [look here](http://danielwestheide.com/blog/2013/02/06/the-neophytes-guide-to-scala-part-12-type-classes.html). What matters is that we've created separated objects with `Ceasarable` implementations for `String` and `Int` types and we enclosed them in static-like objects (`object` is as close as you can get to `static` in Scala). Those types know nothing about their ability to be expressed in Ceasar cipher. 

Let's try the same approach in Swift:

{% highlight swift %}

    struct CeasarableInt : Ceasarable {
       typealias T = Int
       static func toCeasar(input: Int) -> String {
           return "\(input)".unicodeScalars.reduce("") { 
               (acc, char) in
               return acc + String(UnicodeScalar(char.value + 3))
           }
       }
    }

    struct CeasarableString : Ceasarable {
       typealias T = String
       static func toCeasar(input: String) -> String {
           return input.unicodeScalars.reduce("") { 
               (acc, char) in
               return acc + String(UnicodeScalar(char.value + 3))
           }
       }
    }

{% endhighlight %}

Looks valid. This way we've defined the ability to be encoded in Ceasar cipher for strings and integers in each language we consider. 

Now we can work with `Ceasarable` objects just as with any other group of objects sharing common characteristics, i.e. type. We can declare that we expect it as function parameter, we can return it in a function result and so on. Let's see the example usages. In Haskell:

{% highlight haskell %}

    encodeInCeasar :: (Ceasarable c) => c -> String
    encodeInCeasar = toCeasar
    
    encodeInCeasar 1234 -- "4567"
    
    encodeInCeasar "ABCabc" -- "DEFdef"

{% endhighlight %}

We are using the typeclass just like we'd use the protocol - to define a contract without explicitly defining what object are gonna conform to this contract.

How about Scala?

{% highlight scala %}

    def encoderInCeasar[T: Ceasarable](c: T) = {
      println(implicitly[Ceasarable[T]].toCeasar(c))
    }

    encodeInCeasar(1234) // "4567"

    encodeInCeasar("ABCabc") // "DEFdef"

{% endhighlight %}

The sky, once again, gets a little bit cloudy. Instead of requiring the protocol confirmation, we're explicitly asking for the proper implementation using `implicitly`. Implicitly needs to have the implementations passed inside the function, and the enclosing scope is passed via a mechanism called context bound. `T: Ceasarable` is a syntax for context bound. It might sound confusing, but it's fine, actually. This way we can easily see that we're using a typeclass. In Swift, however, we encounter a problem:

{% highlight swift %}

    func encodeInCeasar<C : Ceasarable>(c: C.T) -> String {
        return C.toCeasar(c)
    }

    encodeInCeasar(c: 1234) // Compiler error: Cannot invoke 'encodeInCeasar' with an argument list of type '(Int)'

{% endhighlight %}

Swift compiler cannot infer the generic parameter. There is a struct that does exactly what we want: conforms to `Ceasarable` and defines `Int` as its associated type. However, it cannot be found automatically. Swift doesn't have semantics for Scala-like context bound. However, we've got the second best thing... Wait! It's actually the first best thing, only Swift is 2.0. Protocol extensions.

## Swift typeclasses defined with protocol extensions

In Swift we can use the `extension` keyword to provide implementations for already existing types. The beauty of extension lays in its two properties: universality and ability to be constraint. By universality I mean that you can extend all the Swift types: protocols, classes, structs and enums. The ability to be constraint let us express what we want to extend in a great detail - greater than allowed by protocol confirmation or class inheritance alone.

Did I mention that if you've watched ["Protocol-Oriented Programming in Swift"](https://developer.apple.com/videos/wwdc/2015/?id=408) you'll feel at home? Our better implementation of typeclasses starts with a slight change to the `Ceasarable` definition: 

{% highlight swift %}

    protocol Ceasarable {
        static func toCeasar(input: Self) -> String
    }

{% endhighlight %}

Instead of requiring the associated type in protocol, we can add a `Self` requirement. This way we're expressing that for whatever type we're providing the typeclass implementation, it requires the value of that type as the parameter. It stays closer to the original Haskell definition, because the typeclass doesn't need to be generic. It is just like a template for multiple function definitions that differ only by the type in signature. `Self` expresses exactly that, therefore the actual implementations become easier to write and more readable: 

{% highlight swift %}

    extension Int : Ceasarable {
        static func toCeasar(input: Int) -> String {
            return "\(input)".unicodeScalars.reduce("") { 
                (acc, char) in
                return acc + String(UnicodeScalar(char.value + 3))
            }
        }
    }

    extension String : Ceasarable {
        static func toCeasar(input: String) -> String {
            return input.unicodeScalars.reduce("") { 
                (acc, char) in
                return acc + String(UnicodeScalar(char.value + 3))
            }
        }
    }

{% endhighlight %}

It looks like a straightforward protocol confirmation and it's just what we need. Having that, the usage get simpler as well: 

{% highlight swift %}

    func encodeInCeasar<T : Ceasarable>(c: T) -> String {
        return T.toCeasar(c)
    }

    encodeInCeasar(1234) // "4567"

    encodeInCeasar("ABCabc") // "DEFdef"

{% endhighlight %}

This is what we tried to achieve. At the same time we're providing behavior separate from data (since it's static method of `T`) and expressing the common functionality (since `T` must be `Ceasarable`). By using protocol extensions, we've enabled the second dimension, somewhat orthogonal to inheritance, in which we can compose our functionalities.

## What are Swift typeclasses, then? 

A typeclass in Swift is a pattern build using the protocols and extensions. It's simple and there's nothing new, really, as we've been already using those concepts extensively. As a side note, the process of learning functional programming is very often like that: concepts we used for a long time, but differently named, generalized and ready to build upon. 

Typeclasses are a way of providing a behavior for the type separately from the type and at the same time defining a contract that the type conforms to. It might be used to add functionalities and build composition without inheritance. 

That's all for the first installment of the "Road to [Mobilization](http://2015.mobilization.pl/)" series. The following part will take the concept of typeclasses a little bit further and show some examples of how it can be refactored, modified and used. 

I'm more than happy to discuss above article or any other topic related to iOS development on Twitter. I'm [@_siejkowski](https://twitter.com/_siejkowski) there. 

### {{ page.date | date: "%Y-%m-%d" }} 
