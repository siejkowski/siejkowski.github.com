---

layout: layout
title: "Road to Mobilization #2: Swift typeclasses refactored, modified and used"
author: Krzysztof Siejkowski
date: 2015-09-21
keywords: mobilization, mobilization 2015, swift, typeclasses, haskell, scala, typeclasses in swift
description: 

---

# outline:

## refactor:
* default implementations in extensions can help avoid code repetition
* can we express typeclasses for protocols? (not really, it seams)
* `ceasarify` and boxing

## modify:
* object-oriented typeclasses: no longer `static`,  methods not functions
* we're leaving the boundaries of the design pattern, but we get some benefits out of that
* can we use non-public, private `input` data inside `static` method?

## use:
* classes we have no control over
* classes we cannot inherit from
* classes we do not want to inherit from

## examples:
* typeclass in object-oriented world is an equivalent of Objective-C `respondsToSelector:`
* typeclass can bring together common interface, like `map` method for different types

Going back to the code, one pretty obvious criticism that comes to mind when looking at implementations provided for strings and integers is: it doesn't scale! What if we'd like to make many more object encodable in Ceasar cipher? Do we need to provide very similar implementations for all of them? 

The answer is somewhat more language-specific than previous examples, so let's drop Haskell and Scala versions and focus on Swift.

What we would like to do is to add the ability to be expressed in the Ceasar cipher for all the object that can be transformed to String, just as we did for Integer. Ideally, we'd be able to write something like:

    extension CustomStringConvertible : Ceasarable {
        func toCeasar() -> String {
            return "\(self)".unicodeScalars.reduce("") { (acc, char) in
                return acc + String(UnicodeScalar(char.value + 3))
            }
	}
    }

    print(123.0.toCeasar()) // this is Double, which is also CustomStringConvertible

This construction would both provide the `Ceasarable` implementation for all the types that conform to `CustomStringConvertible` and at the same time make all those types conform to `Ceasarable`. Unfortunately, it's not possible in Swift right now. What we can do instead is to provide the default implementation of `toCeasar` or those types conforming to `Ceasarable` that also conform to `CustomStringConvertible`:

    extension Ceasarable where Self : CustomStringConvertible {
        func toCeasar() -> String {
            return "\(self)".unicodeScalars.reduce("") { (acc, char) in
                return acc + String(UnicodeScalar(char.value + 3))
            }
        }
    }

Well, it's not ideal, because it still makes us express the `Ceasarable` confirmation explicitly for each type:

    extension Int : Ceasarable {}
    
    extension String : Ceasarable {}

At least thanks to the default implementation it's as short as possible! We can also do something different: instead of writing that `Int` conforms to `Ceasarable`, we can provide a method for all the `CustomStringConvertible` to become `Ceasarable`:

    extension CustomStringConvertible {
        func ceasarify() -> Ceasarable {
            return CeasarableBox.Value(convertible: self)
        }
    }
    
    enum CeasarableBox : Ceasarable {
        case Value(convertible: CustomStringConvertible)
        func toCeasar() -> String {
            switch self {
            case .Value(let convertible):
                return "\(convertible)".unicodeScalars.reduce("") { (acc, char) in
                    return acc + String(UnicodeScalar(char.value + 3))
                }
            }
        }
    }

    print(123.ceasarify().toCeasar()) // "456"

This method is not without its drawbacks, the biggest of which is the need to explicitly un-box the values each time we need to use them in `Ceasarable` context. However, sometimes it's the only possibility. We might be given the `CustomStringConvertible` value from the external API and we cannot assume what force-conversion (`as!`) won't crash our app. Frankly, I'd never assume that, as anything _force_ or _unconditional_ is exactly what we're trying to avoid with static types.

But what it we'd like to provide separate implementation for a type that is serializable to string? Than we do have two matching implementations and the compiler must choose the one that is more appropriate:

    extension Ceasarable where Self : CustomStringConvertible {
        func toCeasar() -> String {
            return "\(self)".unicodeScalars.reduce("") { (acc, char) in
                return acc + String(UnicodeScalar(char.value + 3))
            }
        }
    }
    
    extension Int : Ceasarable {
        func toCeasar() -> String {
            return "\(self)".unicodeScalars.reduce("") { (acc, char) in
                return acc + String(UnicodeScalar(char.value + 6))
            }
        }
    }
    
    print(123.toCeasar()) // 789

The more direct implementation was used, specific one instead of default one. If there's any problem with that implementation to choose, Swift compiler will let us know with _Ambigous use of toCeasar()_ error.


