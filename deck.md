---
theme: gaia
_class: lead
paginate: true
backgroundColor: #888
backgroundImage: url('./assets/background.jpg')
author: Riccardo Cardin
lang: en
color: #063970
footer: "Riccardo Cardin - 2025"

marp: true
---

<style>
  header,footer {
    color: #063970;
  }
</style>

![bg left:40% 80%](./assets/logo.png)

# Yo Dawg, Heard You Want To FlatMap Your Direct-Style
Effect system in Scala using capability passing style 

---

# Agenda

* Who Am I?
* References

---

# Who Am I?

---

# Why We ❤️ Functional Programming

* We have the **substitution model** for reasoning about programs
```scala 3
def plusOne(i: Int): Int = i + 1
def timesTwo(i: Int): Int = plusOne(plusOne(i))
```
* The substitution model enables **local reasoning** and **referential transparency**
  * We don't need to look at the implementation
  * Original program and the substituted program are *equivalent*
* We call these functions *pure* functions
---

# We Live in an Imperfect World

> Model a coin toss, but with a twist: the gambler might be too drunk and lose the coin

```scala 3
def drunkFlip(): String = {
  val caught = Random.nextBoolean()
  val heads =
    if (caught) Random.nextBoolean()
    else throw new Exception("We dropped the coin")
  if (heads) "Heads" else "Tails"
}
```

---

# We Live in an Imperfect World

* We can't use the substitution model for all programs
  * If the `drunkFlip` function throws an _exception_, the substitution model breaks

* Programs that interact with a context outside the function
  * The result of the `drunkFlip` function depends on the state of the world

* Multiple calls to `drunkFlip` can return different results

---

# Side Effects

* **Side Effect**: An unpredictable change in the state of the world
  * *Unmanaged*, they just happen

```scala 3
// What happens if b is equal to zero?
def divide(a: Int, b: Int): Int = a / b
```

* We call `divide` an *impure* function
* The best we can do is to *track* and push them to the _boundaries_ of our system

---

# The Effect Pattern

When a side effect is _tracked_ and _controlled_ we call it an **effect**
  1. The _type_ of the function should tell us what effects it can perform and what's the type of the result
    - The `drunkFlip` deals with _non-determinism_ and _errors_
  2. We must separe the _description_ from making it happen
    - We want a _recipe_ of the program.
    - **Deferred execution**

The pattern lets us use the **substitution model** again 🚀 🎉

---

# Effect Systems

An **Effect System** is the implementation of the _Effect Pattern_

* It puts side effects in a _box_
* It replaces side effecting operation in standard libraries
* It provides structures to manage effects

In an effect system, a side effect 👎 becomes an effect 👍

---

# References

* [Essential Effects](https://essentialeffects.dev/), Adam Rosien
* [Effect Oriented Programming](https://effectorientedprogramming.com/), Bill Frasure, Bruce Eckel, James Ward
* [Effekt: Capability-passing style for type- and effect-safe, extensible effect handlers in Scala](https://www.cambridge.org/core/journals/journal-of-functional-programming/article/effekt-capabilitypassing-style-for-type-and-effectsafe-extensible-effect-handlers-in-scala/A19680B18FB74AD95F8D83BC4B097D4F), Jonathan Brachthäuser , Philipp Schuster, Klaus Ostermann

