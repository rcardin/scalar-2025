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
* The substitution model enables **local reasoning**
  * No need to look at the implementation
* ...and **referential transparency**
  * Original program and the substituted program are *equivalent*

---

# We Live in an Imperfect World

* We can't use the substitution model for all programs

```scala 3
// We can't substitute `println` with a call to ()
val x: Unit = println("Hello World")
// Every execution of Random.nextInt() is different
val y: Int = Random.nextInt() 
```

* Programs that interact with a context outside the function
```scala 3
def foo(): Unit = ??? // Does it launch a rocket? 😱
```
---

# Side Effects

* **Side Effect**: An unpredictable change in the state of the world
  * *Unmanaged*, they just happen

```scala 3
// What happens if b is equal to zero?
def divide(a: Int, b: Int): Int = a / b
```

* We call `divide` as an *impure* function
* The best we can do is to *track* them and push them to the boundaries of our system

---

# References

* [Essential Effects](https://essentialeffects.dev/), Adam Rosien
* [Effect Oriented Programming](https://effectorientedprogramming.com/), Bill Frasure, Bruce Eckel, James Ward

