---

layout: layout
title: "Trimming Swift generics"
author: Krzysztof Siejkowski
date: 2016-04-21
keywords: swift, generics, swift generics, swift language, closures, programming 
description: Swift generics are a great tool, but sometimes it's hard to keep them encapsulated. See some simple ways how to do that.

---
# [{{ page.title }}]({{ page.url }}) 

Swift generics are a great tool when applied appropriately. They are immensely useful both in an *object form*, as generic parameter for `class`, `enum` or `struct` type, and in *constraint form*, as `associatedtype` and `Self` requirements in `protocol`. The standard library and many community libraries put generics to work for code reduction and simplification. 

The power of generics, however, doesn't come without drawbacks. The client of generic-driven API is expected to do a little bit of heavy lifting. It needs to either know how to specialize the generic type (hopefully just by compiler interfering) to or become generic itself. At the same time, the good practice of preferring composition over inheritance increases the number of places in which the object must be injected or passed to method. The more those places, the more tempting it becomes to generalize the client. Iterating over this process leads to the *escalation of generics*: once you add generic parameter to one class, soon a huge part of your program becomes generic over this parameter.

How to identify and defend against the escalation? Let's see three simple techniques for trimming the generics. After all, as the responsible [software gardeners](https://culturalcoder.com/inclusivity-of-programming-metaphors/), we don't want our types to wither under the shadow of unweeded generalizations!

## Technique 1: when client uses API that doesn't need generics

Let's say we've got a struct that is parametrized. It might be parametrized for a valid reason, such as need to store a property of generic type. The client of the struct, however, uses only a subset of its functionality. What's more important, that subset doesn't touch the stored property of generic type. From the client perspective, there's no correlation between what type the struct is specialized with and the used API:

```swift
struct Problem1<T> {
    let prop : T? = nil
    func increment(counter: Int) -> Int {
        return self.dynamicType.increment(counter)
    }
    static func increment(counter: Int) -> Int {
        return counter + 1
    }
}

// I would like to write:
//
// let problem1Instance = Problem1()
//
// because I won't use any functionality concerning T.
// However, I must specify type to use the initializer:
let problem1Instance = Problem1<String>()
let incrementedFromProblem1Instance = problem1Instance.increment(41)

// I'd also like to write:
//
//let incremented = Something.increment(2) 
//
// but then I get the error:
// `Generic parameter 'T' could not be inferred`
// Even though I don't care about what T is,
// I must specify it to use static method:
let incrementedFromProblem1Type = Problem1<Any>.increment(41)
```

Whether the functionality is exposed via instance or static method, the client must specify the `T` type. But it doesn't matter what `T` is! Just ignore it, can we?

Well, we can, although with a little bit of additional work. Let's split the `Problem1` struct into two parts: the part that makes use of `T` and the part that doesn't care. Then let's extract the second part into a separate protocol and possibly provide the default implementation. Now we can make the original `Problem1` to conform to that protocol. Also, we can create a separate companion struct that will provide the functionality without the generic baggage. This way both the clients that care about `T` and those who doesn't will be able to use the functionality the same way.

```swift
// this used to be Problem1<T>
struct Solution1<T> {
    let prop : T? = nil
}

// extract signatures of non-generic functionalities
protocol Incrementing {
    func increment(counter: Int) -> Int
    static func increment(counter: Int) -> Int
}

// provide default implementations
extension Incrementing {
    func increment(counter: Int) -> Int {
        return self.dynamicType.increment(counter)
    }
    static func increment(counter: Int) -> Int {
        return counter + 1
    }
}

// add the functionalities back to original source
extension Solution1 : Incrementing {}

// expose the functionalities without generics
struct Solution1Companion : Incrementing {}

// now I can use the functionalities without specifying type
let solution1Instance = Solution1Companion()
let incrementedFromSolution1Instance = solution1Instance.increment(41)
let incrementedFromSolution1Type = Solution1Companion.increment(41)
```

Is it more work? Yes. Does it make it easier for the client to use desired API? Also yes. Doesn't it make design somewhat convoluted? Yes again. However, if the functionalities were not using the property that made the struct generic, shouldn't they be extracted from the beginning? After all, object is an encapsulated package of data and logic operating on that data. The additional methods are a code smell. While I can easily understand why it might be useful to expose them via struct, I also think that extracting separate protocol just explicitly states the design costs.

## Technique 2: downgrade generics from type signature to method signatures

Other time you might have an object that delegates its work to some other object. That other object is parametrized. Let's see the example situation:

```swift
struct Incrementer<T : IntegerType> {
    func increment(counter: T) -> T {
        return counter + 1
    }
}

struct Problem2<T : IntegerType> {
    func increment(counter: T) -> T {
        return self.dynamicType.increment(counter)
    }
    static func increment(counter: T) -> T {
        return Incrementer<T>().increment(counter)
    }
}

let problem2Instance = Problem2<Int>()
let incrementedFromProblem2Instance = problem2Instance.increment(41)
let incrementedFromProblem2type = Problem2<Int>.increment(41)
```

`Problem2` doesn't do much work itself, it just delegates it to generic `Incrementer`. What's wrong with this example? Firstly, just as in the first technique, the client has to care about specifying the generic parameter. As before, we'd rather want to write:

```swift
// doesn't compile - Problem2 requires generic parameter
let problem2Instance = Problem2()
let incrementedFromProblem2Instance = problem2Instance.increment(41)
let incrementedFromProblem2type = Problem2.increment(41)
```

There's also another problem. Placing the generic parameter in a type signature creates a correlation between the provided type (`<Int>` in our simple case) and the actual functionality. In other words, our `problem2Instance` will not work for any other `IntegerType`:

```swift
// doesn't compile - problem2Instance is only for Int, not for UInt64
let otherType : UInt64 = 41
let incrementedOtherType = problem2Instance.increment(otherType)
```

However, should there be one? Looking at the implementation of `Problem2`, there's nothing that justifies the specialization. All the work is delegated to one-time instance of `Incrementer` struct. This instance is created each time the incrementation is executed, so why not specialize it with type that is required at the moment?

The answer is easy. Let's move the generics from type signature to method signatures:

```swift
struct Solution2 {
    func increment<T : IntegerType>(counter: T) -> T {
        return self.dynamicType.increment(counter)
    }
    static func increment<T : IntegerType>(counter: T) -> T {
        return Incrementer<T>().increment(counter)
    }
}

let solution2Instance = Solution2()
let incrementedFromSolution2Instance = solution2Instance.increment(41)
let incrementedFromSolution2Type = Solution2.increment(41)
let test: UInt64 = 41
let incrementedOtherType = solution2Instance.increment(41)
```

"Why have you added the generalization to type signature in the first place?", you might ask. "The methods should be parametrized from the start". I agree. Adding generic parameter to type signature was a mistake. 

It is an easy mistake to make, though. When you see an object with, say, 6 methods, each of these methods parametrized, possibly with additional constraints (`where` clause), you might easily fall into thinking that moving the generics to type signature is a way of cleaning the code. Which is partially true: removing generics from methods will make them more readable. More important, once you need to change the generic constraints (for example, you generalize over error and someone renamed `ErrorType` to `ErrorProtocol`), you need to do it in one place only. It's *Don't Repeat Yourself 101*, right?

Well, this time repeating yourself might be a good idea. The maintenance cost of your API will rise, but your object becomes both easier to use and more universal.

## Technique 3: catch the generic object in closure 

One of the first hoops to jump through for new Swift users is `Protocol '...' can only be used as a generic constraint because it has Self or associated type requirements`. The common situation for this error is when you want to create a property of protocol type, and this protocol is a generic. There's no way of specializing the protocol in the property declaration. The type must become parametrized with constraints that match the property expected type:

```swift
// generic protocol
protocol Decrementable {
    associatedtype T : IntegerType
    func decrement(counter: T) -> T
}

extension Decrementable where T : IntegerType {
    func decrement(counter: T) -> T {
        return counter - 1
    }
}

struct Decrementer : Decrementable {
    typealias T = Int
}

// generalization escalated to client signature
struct Problem3<D : Decrementable> {

    let decrementable: D
    
    init(decrementable: D) {
        self.decrementable = decrementable
    }
    
    func decrement(counter: D.T) -> D.T {
        return decrementable.decrement(counter)
    }
}

let decrementer = Decrementer()
let problem3 = Problem3(decrementable: decrementer)
let decrementedWithProblem3 = problem3.decrement(43)
```

Isn't that bad, right? The API is clean, no generics leaking through inference. Let's try to use it in a slightly different way:

```swift
var problem3Optional: Problem3<Decrementer>? = nil
problem3Optional = Problem3(decrementable: decrementer)
let decrementedWithProblem3Optional = problem3Optional?.decrement(43)
```

Oh no! When working with `Problem3` type in different contexts, like enclosing it in an `Optional`, we need to explicitly specify its generic parameter, `Decrementer`. Also, `decrement` signature doesn't communicate to the client that the method is working only with `Int`. Not before one tries to use it with different type (say, `UInt64`) one discovers that `Cannot convert value of type 'UInt64' to expected argument type 'T' (aka 'Int')`. It's not very readable for the users of `Problem3` instance.

What we would like to do is to specify from the beginning that `Problem3` works only on `Int`:

```swift
struct Problem3Wannabe {

    // we want Problem3 to operate only on Ints,
    // but you cannot move protocol specification 
    // into property declaration. not supported.
    let decrementable: Decrementable where D.T == Int

    init<D : Decrementable where D.T == Int>(decrementable: D) {
        self.decrementable = decrementable
    }

    func decrement(counter: Int) -> Int {
        return decrementable.decrement(counter)
    }
}
```

Can we do better? Well, kind of yes! If you expect the object that implements a generic protocol (and, by necessity, specifies its `associatedtype`) and you want to catch its functionality without escalating the generics to type signature, you might catch it in closure:

```swift
struct Solution3 {
    
    // the closure is specified with Int
    let decrementing: Int -> Int
    
    init<D : Decrementable where D.T == Int>(decrementable: D) {
        self.decrementing = { counter in decrementable.decrement(counter) }
    }
    
    // decrement is now explicitely expecting Int
    func decrement(counter: Int) -> Int {
        return self.decrementing(counter)
    }
}

let decrementerSolution = Decrementer()
let solution3 = Solution3(decrementable: decrementerSolution)
let decrementedWithSolution3 = solution3.decrement(43)

var solution3Optional: Solution3? = nil
solution3Optional = Solution3(decrementable: decrementer)
let decrementedWithSolution3Optional = solution3Optional?.decrement(43)
```

This technique is not always possible, and might require significant overhead, especially when the generic protocol is huge. Then you'd need to create separate closure for each of its methods or write small routing to control what function should be invoked inside a closure. For small protocols, however, storing a closure will help you encapsulate the generics and prevent them from escalation. Client of `Solution3` doesn't even need to know that there's any generic type involved.

## Trim the generics whenever you can

All of the techniques share common idea: **each time you add a generic parameter to type signature, think whether it's unavoidable**. Often it isn't. It might be completely unnecessary (*technique 1*), it might be easily avoided (*technique 2*) or encapsulated in an ad-hoc object like closure (*technique 3*). Either way, it always leads to *escalation of generics*.

And when it's not necessary to add generics to your type signature, please don't. A good way of thinking would be to ask yourself: *does the client of my object should care about the generic type?* If the answer is *no*, than don't make it care. It'll make the future refactoring easier, and will also make the fellow programmers joining your project happier.

I'm more than happy to discuss this or any other topic related to software development. Contact me [@\_siejkowski](https://twitter.com/_siejkowski). 

### {{ page.date | date: "%Y-%m-%d" }}
