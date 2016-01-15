# Pattern Matching Partial Function

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): [Swift Developer](https://github.com/swiftdev)
* Status: **Awaiting review**
* Review manager: TBD

## Introduction

Provide the ability for defining partial functions using familiar `case`/`default` pattern matching syntax.  The proposal includes the ability to use the existing `map` function to provide the same functionality as a switch-expression.  All basic types such as `Int`, `Float`, `Bool`, `Optional`, `tuple`, `String` must support the `map` function so that the same `map` using these same partial functions will provide the ability to use it as a switch-expression.  Functions on arrays, dictionaries or other collections such as `reduce` or `filter` would accept these partial functions.  Use of `case`/`default` partial functions must always be exhaustive providing a total function.

Swift-evolution thread: [link to the discussion thread for that proposal](https://lists.swift.org/pipermail/swift-evolution)

## Motivation

The proprosal grew out of the desire to be able to be able to have a switch-expression for pattern matching or a like expression for cases where there are more than two potential results.

Discussions about multi-case ternary solutions lead to the idea that the same functionality could be provided in a more generalized solution that would work for both a value or a collection of values.  Swift already has a function `map` that provides the ability to map a collection to a new collection using a function.  If the ability to define partial functions to map values from one collection to another for a subset of the values, a total function could be composed out of passing multiple partial functions.  Using existing `case`/`default` syntax for defining these partial functions will make it more intuitive.

This proposal provides a generalized solution to the commonly proposed requests:

1. `switch` expression.
3. Replacement of ternary operator which is considered magical and terse by some.  This proposal does not propose any changes to the ternary operator.

## Proposed solution

Any function which accepts a closure would also be able to accept a closure composed of partial functions defined using `case`/`default` syntax used in switch-case.  Each `case` or `default`is considered a partial function since it defines the closure for a subset of values.  

```
public enum Trade {
    case Buy(quantity: Double, price: Double)
    case Sell(quantity: Double, price: Double)
}

let commissions = trades.map {
    case .Buy(let quantity, let price) where quantity * price > 10000:
        let vipCommissionRate = calculateCommissionRate(...)
        quantity * price * vipCommissionRate / 100
    case .Buy(let quantity, let price):
        let commissionRate = calculateCommissionRate(...)
        quantity * price * commissionRate / 100
    case .Sell(let quantity, let price) where quantity * price > 10000:
        let vipCommissionRate = calculateCommissionRate(...)
        quantity * price * vipCommissionRate / 100
    case .Sell(let quantity, let price):
        let commissionRate = calculateCommissionRate(...)
        quantity * price * commissionRate / 100
}
	         
```




Alternative grammar `cases`/`default` which is a specialized version that can be used for the simplest and the most consise use case.  If the use case is a simple definition of mapping of values then `cases` can be used to define multiple  case clauses. `cases` clause may not be used with a `where` clause.  The purpose of allowing `cases` as syntatic sugar for multiple `case` clauses is to allow a less verbose option for use cases where the developer would use a multi-case ternary expression.


```
let col = [1,5,7,9]

let newCol = col.map {
	cases 1: "one", 2: "two",   3: "three", 4: "four", 5: "five",
	      6: "six", 7: "seven", 8: "eight", 9: "nine", 0: "zero"
	default: ""
}
	         
```

To be consistent and allow for match expressions on values such as `Int`, `Float`, `Bool`, `Optional`, `tuple`, `String` should support mapping that one value to another value.

```
let n = 5

let n.map {
	cases 1: "one", 2: "two",   3: "three", 4: "four", 5: "five",
	      6: "six", 7: "seven", 8: "eight", 9: "nine", 0: "zero"
	default: ""
}


```

Functions such as `reduce` that receive two or more values must be enclosed in brackets `(x, y)` to otherwise the parser would likely have trouble distinquishing between comma delimited lists of values which are currently allowed as a single case.

```
public enum Troy {
    case Pound(Int)
    case Ounce(Int)
    case Pennyweight(Int)
    case Grain(Int)
}

let weightTroy = [Troy.Pound(5), Troy.Ounce(4), Troy.Pennyweight(6), Troy.Grain(9)]

let weightKg = weightTroy.reduce(0.00) {
    case (acc, Troy.Pound(let quantity)):
        acc + Double(quantity) * 0.373
    case (acc, Troy.Ounce(let quantity)):
        acc + Double(quantity) * 0.031103
    case (acc, Troy.Pennyweight(let quantity)):
        acc + Double(quantity) * 0.001555
    case (acc, Troy.Grain(let quantity)):
        acc + Double(quantity) * 0.0000648
    }
}
	         
```

## Detailed design

Any function which accepts a closure would also be able to accept a closure composed of partial functions defined using `case`/`default` syntax used in switch-case.  Each `case` or `default`is considered a partial function since it defines the closure for a subset of values.  

**GRAMMAR OF A CLOSURE COMPOSED OF PARTIAL FUNCTIONS**

closure → ­ {­switch-cases­<sub>opt</sub>­}­  
switch-cases → switch-case­ switch-cases­<sub>opt</sub>  
switch-case → case-label­ statements­ | default-label­ statements  
case-label → **case­** case-item-list­ :­  
case-item-list → pattern­ where-clause­<sub>opt</sub>­ | pattern ­where-clause­<sub>opt</sub>­ , ­case-item-list­  
default-label → **default**­ :­  
where-clause → **where**­ where-expression­  
where-expression → expression­




Alternative grammar `cases`/`default` which is a specialized version that can be used for the simplest and the most consise use case.  If the use case is a simple definition of mapping of values then `cases` can be used to define multiple  case clauses. `cases` clause may not be used with a `where` clause.  The purpose of allowing `cases` as syntatic sugar for multiple `case` clauses is to allow a less verbose option for use cases where the developer would use a multi-case ternary expression.


**NEW GRAMMAR FOR CONSISE FORM OF CASES**

closure → ­ {­switch-cases­<sub>opt</sub>­}­  
switch-cases → cases-label­ statements­ | default-label­ statements  
cases-label → **cases**­ case-item-map­  
case-item-map → pattern­ : value | pattern : value ­ , ­case-item-list­  
default-label → **default**­:­  



## Impact on existing code

This mechanism is opt-in, so existing code won't be affected by this change.

## Alternatives considered

There were various specialized multi-ternary suggestions made as a replacement/expansion of ternary which supported more than two possible examples, but are more of a specialized case that this proposal should eliminate the need for.  

The only other alternative considered that was a generalized version similar to the proposal above was basically the same but using `in` instead of `:` because we were defining partial functions and not a `switch` command.  The overwelming sentiment was in favour of using exactly the same syntax as the `switch` command where possible.

```
public enum Troy {
    case Pound(Int)
    case Ounce(Int)
    case Pennyweight(Int)
    case Grain(Int)
}

let weightTroy = [Troy.Pound(5), Troy.Ounce(4), Troy.Pennyweight(6), Troy.Grain(9)]

let weightKg = weightTroy.reduce(0.00) {
    case (acc, Troy.Pound(let quantity)) in
        acc + Double(quantity) * 0.373
    case (acc, Troy.Ounce(let quantity)) in
        acc + Double(quantity) * 0.031103
    case (acc, Troy.Pennyweight(let quantity)) in
        acc + Double(quantity) * 0.001555
    case (acc, Troy.Grain(let quantity)) in
        acc + Double(quantity) * 0.0000648
    }
}
	         
```

