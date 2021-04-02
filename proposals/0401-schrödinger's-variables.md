# Schrödinger's Variables

* Proposal: [SE-0401](0401-schrödinger's-variables.md)
* Authors: [冀卓疌](https://github.com/WowbaggersLiquidLunch)
* Review Manager: TBD until observed
* Status: **Pitched**
* Implementation: [not a pull request](https://github.com/WowbaggersLiquidLunch/Quantum)
* Previous Revision: [`Never`](https://forums.swift.org/404)
* Previous Proposal:[`Never`](https://forums.swift.org/404)

> Note: The author has received no formal education in quantum mechanics. His understanding of modern physics goes only as far as Einstein's special relativity.

## Introduction

Much to the dismay of my imaginary cat, most<sup>[_[citation needed](https://en.wikipedia.org/wiki/Wikipedia:Citation_needed)_]</sup> Swift programs are deterministic finite-state machines. That is, with enough information, a person can predict the state of the program at any point in time. Given enough motivation, another person can step through the compiled binary instructions to verify it. This proposal provides an easy path for variables to behave quantum mechanically, and thus make programs unpredictable and totally cool.

Swift-evolution thread: [Discussion thread topic for that proposal](https://forums.swift.org/)

## Motivation

[There](https://forums.swift.org/t/deprecate-and-make-never-the-bottom-type/35517) have [been](https://forums.swift.org/t/moving-toward-deprecating-force-unwrap-from-swift/43455) many [threads](https://forums.swift.org/t/pitch-soft-unwrapping-of-optionals/2555) on the Swift forums pitching to deprecate or replace force-unwrapping of optionals. Discussions in those threads were always divisive, with those against it finding values in force-unwrapping where the variable is known to be non-`nil`. For example:

```swift
var optionalNumber: Int? = nil
optionalNumber = 1
let number = optionalNumber! // number == 1
```

Now what if we can find a way to make it impossible for anyone to predict whether an optional value can be force-unwrapped successfully? It won't invalidate the opinion that force-unwrapping is useful, but it should be interesting though. 

<details>
    <summary>
        omitted section
    </summary>
    <br>
    I was going to write a short quantam mechanics primer, but didn't find enough time for it.
    <br>
    <br>
    Well, I had time to write it, but I've been playing too much Minecraft lately...
</details>

## Proposed solution

We can throw an optional variable into the quantum realm by attaching a `@Quantum` attribute to its declaration. This way, no one knows if it is `nil` until they're unwrapped. For example:

```swift
@Quantum
var optionalNumber: Int? = nil // `optionalNumber` is initialised with an initial state of `nil`
optionalNumber = 1 // `optionalNumber` now has a 50% chance of being `nil`, and 50% being 1
let number = optionalNumber! // The force-unwrapping has a 50% chance of being successful
```

Although the motivation is in the context of optional values, this works with non-`Optional` typed variables too:

```swift
@Quantum
var schrödingersText = "Happy" // `schrödingersText` is initialised with an initial state of `"Happy"`
schrödingersText = "April" // `schrödingersText` now has a 50% chance of being `"Happy"`, and 50% being `"April"`
schrödingersText = "Fools" // `schrödingersText` now has a 25% chance of being `"Happy"`, 25% being `"April"`, and 50% being `"Fools"`
```

When a variable is declared with the `@Quantum` attribute, the value it's initialised to becomes its initial state, and all succeeding mutations contribute to the probability distribution of its next state when measured. Now each mutation takes on a new meaning of "whatever the value the variable might evaluate to, superpose a new value onto it". However, it is possible to restrict the superposition to a specific possible value that the variable may evaluate to:

```swift
@Quantum
var schrödingersNumber = 0 // `schrödingersNumber` is initialised with an initial state of 0
schrödingersNumber = 1 // `schrödingersNumber` now has a 50% chance of being `0`, and 50% being 1
$schrödingersNumber.superpose(1, on: 2) // `schrödingersNumber` now has a 50% chance of being 0, 25% being 1, and 25% being 2
print($schrödingersNumber.outcomeProbability) // prints "[0: 0.5, 1: 0.25, 2: 0.25]"
```

## Detailed design

`Quantum` is a property wrapper that treats each attempt of mutating the wrapped variable as a quantum event, which is an event that might or might not have happened to move the `@Quantum`-wrapped variable from one state to the next. An observed quantum event is known to have happened, and an unobserved event isn't. Because the arrow of time goes unidirectionally, all quantum events connect into a tree as the backing storage of the wrapper. When the variable is measured, the tree is traversed with a coin toss at each level/depth to determine which quantum event is observed, with its unobserved siblings discarded, until no unobserved quantum event is left. 

## Alternatives considered

### Model each quantum variable as a complex vector space

It could be more efficient to model each `@Quantum`-annotated variable as a complex vector space, iff there is a finite and small number of possible states the variable can be in. In the standard library, only 3 types: `Bool`, `Void`, and `Never` fit the requirement. Other types such as `Int` and `String` model infinitely many integer values and uncountably infinite combination of characters, respectively, although their representable values are finite in number.

### Model each mutation of a quantum variable as a unitary transformation

With the given tools in the standard library (`Double`), sin and cos used in unitary transformation does not provide the probability distribution of a measurement's outcome as precisely as the simple division by 2 used in a quantum event tree.

### Support quantum entanglement

With quantum entanglement supported, we can achieve something like this:

```swift
@Quantum
var number = 1
number = -1
print($number.outcomeProbability) // prints "[1: 0.5, -1: 0.5]"
@Quantum
var numberIsPositive = number > 0 // entangles `numberIsPositive` with `number`
print($numberIsPositive.outcomeProbability) // prints "[true: 0.5, false: 0.5]"
print(numberIsPositive) // `numberIsPositive` is measured here
                        // if `numberIsPositive` is measured to be `true`, then `number` is simultaneously measured to be 1;
                        // if `numberIsPositive` is measured to be `false`, then `number` is simultaneously measured to be -1
```

Currently as proposed, all `@Quantum`-wrapped variables are measured as soon as they're read. If we want to support quantum entanglement, then we need to define precisely when reading a variable measures it and when reading a variable doesn't. In addition, multiple values of different types can be entangled together, so it probably needs some language-level magic.

