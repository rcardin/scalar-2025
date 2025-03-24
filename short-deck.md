---
theme: gaia
_class: lead
paginate: true
backgroundColor: #888
backgroundImage: url('./assets/unicorn.jpg')
author: Riccardo Cardin
lang: en
color: #063970
footer: "Riccardo Cardin - Scalar 2025"

marp: true
---

<style>
  header,footer {
    color: #063970;
  }
</style>

![bg right:40% 80%](./assets/logo.png)

<!-- _paginate: false -->
<!-- _footer: "" -->

# Yo Dawg, Heard You Want to Flatmap Your Direct-style
Effect System in Scala Using Capability Passing Style

---

# Agenda

1. Who Am I?
2. What are Effects & Side Effects?
3. Scala Effect Systems (Cats Effect, ZIO, Kyo)
4. Building Our Own Effect System
5. Conclusions

---

# Who Am I?

* Hello there 👋, I'm **Riccardo Cardin**, 
  * An Enthusiastic Scala Lover 💯

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![w:300 h:300](./assets/github-qr.jpeg)&nbsp;&nbsp;&nbsp;&nbsp;![w:300 h:300](./assets/linkedin-qr.jpeg)&nbsp;&nbsp;&nbsp; ![w:300 h:300](./assets/blog-qr.jpeg)

---

## Why We ❤️ Functional Programming  
- **Substitution model** → Safe, predictable reasoning.
  * _Referential transparency_: We don't need to look at the implementation  
  * _Composition_
- **Pure functions** → Same input = same output, no external state.

```scala 3
def plusOne(i: Int): Int = i + 1
def timesTwo(i: Int): Int = plusOne(plusOne(i))
```

--- 

# We Live in an Imperfect World 💔

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

# The Problem: Side Effects  
❌ **Side effects** break substitution:  
- **Randomness** (`Random.nextBoolean()`)
  - The result depends on a previous state
- **Exceptions** (`divide(1, 0)`)
  - Completely break the substitution model

✔️ **Solution: Effect Systems** track & control them!  

---

# The Effect Pattern

When a side effect is _tracked_ and _controlled_ we call it an **effect**
  1. The _type_ of the function should tell us what effects it can perform and what's the type of the result
  2. We must separate the _description_ from making it happen
    - **Deferred execution** & **referential transparency**

The pattern lets us use the **substitution model** again 🚀 🎉

---

# Effect Systems

An **Effect System** is the implementation of the _Effect Pattern_

* It puts side effects in a _box_, `F[A]` 
* It replaces side-effecting operations in standard libraries
* It provides structures to manage effects

In an effect system, a side effect 👎 becomes an effect 👍


---

# Effect Systems in Scala  

### ✅ **Cats Effect (`IO[A]`)**  
### ✅ **ZIO (`ZIO[R, E, A]`)**  
### ✅ **Kyo (Algebraic Effects)**  

Each provides structured ways to manage effects.

---

# Cats Effect: Handling Side Effects  
```scala
def drunkFlip: IO[String] = for {
  caught <- IO(Random.nextBoolean())
  heads  <- if (caught) IO(Random.nextBoolean())
            else IO.raiseError(new RuntimeException("Coin lost"))
} yield if (heads) "Heads" else "Tails"
```
✔️ `IO[A]` is an _über effect_ that models any effectful computation that can fail with a `Throwable`

✔️ **Referentially transparent**, explicit error handling.  

---

# ZIO: More Type Safety  

```scala
def drunkFlip: ZIO[Random, String, String] = for {
  caught <- Random.nextBoolean
  heads  <- if (caught) Random.nextBoolean else ZIO.fail("Coin lost")
} yield if (heads) "Heads" else "Tails"
```
`ZIO[R, E, A]` is still an über effect on `A`

✔️ **Effects are _explicit_** in the `R` type, and we can **fail with _custom errors_ in `E`**
✔️ _Rich algebra_ on the `ZIO` type to avoid _monad transformers_

---

# Kyo: Algebraic Effects  

What if we can have types _listing Effect separately_ and _handling_ them virtually _once at a time_?

```scala
def drunkFlip: String < (IO & Abort[String]) = for {
  caught <- Random.nextBoolean
  heads  <- if (caught) Random.nextBoolean else Abort.fail("Coin lost")
} yield if (heads) "Heads" else "Tails"
```
✔️ **Effects are explicit in the type.**  
✔️ Handlers execute effects **separately**.  

---

# Building Our Own Effect System  

What if we could create an effect system that _doesn't rely on monads_, but almost preserves _referential transparency_?

# 😱 😱 😱 😱 😱 😱 

🔧 **Key idea**: Use **Context Functions** instead of monads.  

---

# Step 1: Define Effects  

We'll focus on the `drunkFlip` example. We need effects that model non-determinism (`Random`), errors (`Raise`), and output (`Output`)

```scala
trait Random { def nextBoolean: Boolean }
trait Raise[-E] { def raise(error: => E): Nothing }
```
✔️ **Each effect models an operation.**  

---

# Step 2: Provide Access

We need now to wrap the standard library with the effects

```scala 3
object Random {
  private val unsafe = new Random {
    override def nextBoolean: Boolean = scala.util.Random.nextBoolean()
  }
}
```
→ We call the variable `unsafe` ☣️ because it gives _direct_, _uncontrolled_ access to the side effect 

___

# Step 3: Provide Safe Access

```scala
object Random {
  def nextBoolean(using r: Random): Boolean = r.nextBoolean
}
```
✔️ Requires a **capability** (`using r: Random`) to execute.  


✔️ Calling `Random.nextBoolean` produces a _recipe_ for the program

---

# Step 3: Rewrite `drunkFlip`  
```scala
def drunkFlip(using Random, Raise[String]): String = {
  val caught = Random.nextBoolean
  val heads = if (caught) Random.nextBoolean else Raise.raise("Coin lost")
  if (heads) "Heads" else "Tails"
}
```
✔️ No explicit monads, **only capabilities**.  

🪄 Variables `caught` and `heads` are treated as `Boolean`!!


---

# Step 4: Handle Effects  
```scala
object Raise {
  def run[E, A](program: Raise[E] ?=> A): E | A =
    boundary {
      given r: Raise[E] = new Raise[E] { def raise(error: => E): Nothing = break(error) }
      program
    }
}
```
✔️ Converts effectful computations into **values**.  

---

# Step 5: Run the Effect  
```scala
val result: Either[String, String] = Random.run { Raise.either { drunkFlip } }
```
✔️ **Stackable handlers** → Each effect handled separately.  

---

# Bonus: Adding Monadic Operations  
```scala
extension [F, A](eff: Effect[F] ?=> A) {
  inline def flatMap[B](inline f: A => Effect[F] ?=> B): Effect[F] ?=> B = f(eff)
  inline def map[B](inline f: A => B): Effect[F] ?=> B = eff.flatMap(a => f(a))
}
```
✔️ We can still use **`map` and `flatMap`**!  

---

# Conclusions  
✔️ Effects allow us to **track and manage side effects**.  
✔️ Existing solutions: **Cats Effect, ZIO, Kyo**.  
✔️ Our custom **Effect System** avoids monads while preserving **referential transparency**.  

---

# So Long, and Thanks for All the Fish 🐠!  
💡 **YÆS** – _Yet Another Effect System_ is a real library implementing these ideas!  

---

## Final Thoughts?  
🙋‍♂️ Happy to take questions after the talk!  
