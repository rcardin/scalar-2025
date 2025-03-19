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
import scala.util.Random

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
    - **Deferred execution** & **referential transparency**

The pattern lets us use the **substitution model** again 🚀 🎉

---

# An Effect Example

Effects have the form of a generic type `F[A]`

* The `Option[A]` type models the conditional lack of a value 

```scala 3
val maybeInt: Option[Int] = Some(42)
val maybeString: Option[String] = maybeInt.map(_.toString)
```
* Composing function returning effects is not trivial
  * `F[_]` must be a _monad_ so we can use `flatMap` and `map`
  * Different monads are _hard to compose_ (Monad Transformers)

---

# Effect Systems

An **Effect System** is the implementation of the _Effect Pattern_

* It puts side effects in a _box_
* It replaces side effecting operation in standard libraries
* It provides structures to manage effects

In an effect system, a side effect 👎 becomes an effect 👍

---

# Cats Effect

* Cats Effect uses the `IO[A]` data type to model effects
  * `IO[A]` is a _über effect_ that models any effectful computation that returns a value of type `A` and can fail with a `Throwable`
  * It's a _monad_ so we must use `flatMap` and `map` to compose effectful functions
  * `IO[A]` is **referentially transparent** and **lazy** 
  * Redefines the effectul part of the Standard Library
  * Implements _structured concurrency_

---

# Cats Effect

Let's rewrite the `drunkFlip` function using `IO` effect

```scala 3
def drunkFlip: IO[String] =
  for {
    random <- Random.scalaUtilRandom[IO]
    caught <- random.nextBoolean
    heads <-
      if (caught) random.nextBoolean
      else IO.raiseError(RuntimeException("We dropped the coin"))
  } yield if (heads) "Heads" else "Tails"
```
The `drunkFlip` function returns a _recipe_ of the program

---

# Cats Effect

The library provides many ways to _run_ the effect

```scala 3
object Main extends IOApp.Simple {
  def run: IO[Unit] = drunkFlip.flatMap(result => IO.println(result))
}
```
There are also some _unsafe_ methods to run the effect

```scala 3
val result: String = drunkFlip.unsafeRunSync()
val resultF: Future[String] = drunkFlip.unsafeToFuture()
```
---

# Cats Effect

The `IO[A]` hides the exactly performed side effects. We can make them explicit using _Tagless Final_ syntax

```scala 3
def drunkFlipF[F[_]: Random: [G[_]] =>> MonadError[G, String]]: F[String] =
  for {
    caught <- Random[F].nextBoolean
    heads <-
      if (caught) Random[F].nextBoolean
      else ApplicativeError[F, String].raiseError("We dropped the coin")
  } yield if (heads) "Heads" else "Tails"
```
The cognitive load is higher, here 😱

___

# ZIO
* `ZIO[R, E, A]` introduces the errors type `E` and dependencies `R` in the effect definition
  * It's still a monad on the `A` type (`map` and `flatMap`)
  * It provides a _rich algebra_ on the `ZIO` type to avoid monad transformers
  * It's a _referentially transparent_ and _lazy_ effect
  * It provides _structured concurrency_ primitives
  * ...still a über effect

---

# ZIO

The `drunkFlip` function using `ZIO` effect is the following

```scala 3
def drunkFlip: ZIO[Random, String, String] =
  for {
    caught <- Random.nextBoolean
    heads <-
      if (caught) Random.nextBoolean
      else ZIO.fail("We dropped the coin")
  } yield if (heads) "Heads" else "Tails"
```

**Effects are _explicit_** in the `R` type and we can fail with _custom errors_

---

# ZIO

Running the effect means providing needed dependencies or _layers_ 

```scala 3
object Main extends ZIOAppDefault {
  override def run =
    drunkFlip.flatMap { result =>
      Console.printLine(result)
    }.provideLayer(ZLayer.succeed(RandomLive))
}
```

* We can use intersection type: `Random & Console`
* We must fullfil _all the dependencies_ at once to run the effect

---

# Kyo: Meet Algebraic Effects

What if we can have types _listing Effect separately_ and _handling_ them virtually _once at time_?

**Algebraic Effects and Handlers** do exactly that 🎉
 * The type of the function tells us exactly what effects it uses
 * **Kyo** is a novel library implementing Algebraic Effects

```scala 3
def drunkFlip: String < (IO & Abort[String]) = ???
```

---

# Kyo: Meet Algebraic Effects

* Each effect has its own _rich algebra_ to describe the operations

```scala 3
import kyo.*

def drunkFlip: String < (IO & Abort[String]) = for {
  caught <- Random.nextBoolean
  heads  <- if (caught) Random.nextBoolean else Abort.fail("We dropped the coin")
} yield if (heads) "Heads" else "Tails"
```

* Kyo uses a _monad_ to represent the effectful computation
  * We still have to use `flatMap` and `map` 

---

# Kyo: Meet Algebraic Effects

We can decide to _handle each effect separately_ (no über effect)

```scala 3
val partialResult: Result[String, String] < IO = Abort.run { drunkFlip }
```

* `Abort.run` is called an **Effect Handler**
  * It executes the `Abort` effect. The `IO` effect is _left untouched_

* Virtually, we can define a our own effect handler without changing original recipe
  * For example, for testing purposes
---

# References

* [Essential Effects](https://essentialeffects.dev/), Adam Rosien
* [Effect Oriented Programming](https://effectorientedprogramming.com/), Bill Frasure, Bruce Eckel, James Ward
* [Zionomicon](https://www.zionomicon.com/), John A. De Goes, Adam Fraser, Milad Khajavi
* [Effekt: Capability-passing style for type- and effect-safe, extensible effect handlers in Scala](https://www.cambridge.org/core/journals/journal-of-functional-programming/article/effekt-capabilitypassing-style-for-type-and-effectsafe-extensible-effect-handlers-in-scala/A19680B18FB74AD95F8D83BC4B097D4F), Jonathan Brachthäuser , Philipp Schuster, Klaus Ostermann
* [Kyo](https://getkyo.io/)

