# ??? Advanced Scala: Day 1

## Introductions

Noel: Scala for 10 years. Interests: teaching, programming languages, machine learning.
Faraz: 2 years @ ???, dist. sys., programming languages C, assembly, Java. Interests: beat the laws of physics
Dmytro: Automation: spin up pipelines, end to end testing, 4 years at ???, Physics background
Michael: Fraud detection, C, C++, assembly (x86, x86_64, ARM), Python. Little bit of Java, Scala. Information theory, compression, multimedia.
Steven: Web services. Javascript, Go, Python. 1 year Scala.
Matthew: New graduate. 1 year at ???. Python and Java background. Automation, code quality.
Jeffrey: ML engineer, fraud detection. Background: file system drivers. C, Asm. Computer graphics, ML. 5 years of Scala.
Ajay: ~1 years, ML platform. Previously AWS. Distributed systems. Yahoo e-commerce analytics. Understand more advanced Scala topics. 
Azeem: 1 year in current team. Python and Go. Java close 3rd. Little Scala. Demystify Scala. Implicits.
Russell: Fraud detection. Python, Scala (3-4 years), Spark. Reflection.


## Admin

1. Cameras / mics on please, unless you have a good reason
2. Pets (and small children) are welcome to join us, but you should introduce them to everyone else. Older children, friends, housemates, partners, etc. are also welcome.
3. You are ultimately responsible for your learning. Three hints:
   1. Ask questions at any time. "I don't understand this and I don't know how to form a clearer question" is a great question!
   2. Collaborate with other students. Explain what you are doing to them. Explain what you're thinking. "I don't know what to do right now" is a great thing to say.
   3. Take notes.

Goals
1. Overall goal: learn how to design and build systems in Scala
2. Basic principles of functional programming and their application in system design. The importance of semantics.
3. Interpreters and their implementation techniques
4. Type classes in general, and specific type classes that are useful in data pipelines

Admin
- Our curriculum is not fixed. We have a rough plan but it will change depending on your interests.
- We have 5 3.5 hr sessions. We'll have a 30 min break about half way through

What is streamming? Have we used streaming data pipelines before? (Hopefully you have.)
- We're going to build such a system

What functionality / methods should this system have? This will help you think about the problem and help me see what kind of functionality is important for your use cases.

Choose some very (very!) basic functionality and implement it.
- Basic ~Stream~: map, emit, foldLeft. Interpreters. Reification


## Ideas & Notes

Discoverable lambda architecture.
Testability / building / dependency management / deployment / configuration
- best practices

These are the library authors.

Understandability / usability of API.
- E.g. avoiding excessive implicits

Kh..... is a DSL abstracts over data pipelines.
- serialization and deserialization using implicits
- leaky abstraction. E.g. read and write consistency.

Low magic.

Check Azeem's invite

Talk about foldLeft

Send Essential Scala to newcomers


## Features

Reusability.
Replayability.

Joining streams

Windows: sliding, tumbling

Latency sensitivity

Issues with eventual consistency.
- E.g. aggregation over a window, gives a window for malicious actors to operate.

Issues with distributed systems
- parallelism / aggregation


## Algebraic Data Type

Represent data in terms of logical ands and logical ors.
Implementation in Scala:
- sealed trait or sealed abstract class
- (final) case class


```scala
// A is a B or C
sealed trait A
final case class B() extends A
final case class C() extends A
// case object C extends A
```

```scala
// A is B and C
final case class A(theB: B, theC: C)
```

Exhaustivity checking in pattern matching.


## Problem 1

Implement `Stream` with the following API:

- emit: Iterator[A] = Stream[A]
- map: Stream[A] (A => B) = Stream[B]
- foldLeft: Stream[A] (B) ((B, A) => B) = B


```scala
sealed trait Stream[A] {
  import Stream._
  // Combinator
  def map[B](f: A => B): Stream[B] =
    Map(this, f)
    
  // Interpreter
  def foldLeft[B](z: B)(f: (B, A) => B): B =
    ??? 
}
object Stream {
  final case class Map[A,B](source: Stream[A], f: A => B) extends Stream[B]

  // Constructor
  def emit[A](values: Iterator[A]): Stream[A] =
    ???
}
```

Algebra:
1. constructors / introduction form  G[A] => F[A] e.g. Iterator[A] => Stream[A]
2. combinators  F[A] combine G = F[A] e.g. map
3. interpreter / elimination form  F[A] => B e.g. foldLeft


Reification
- Convert all constructors and combinators into data. The data should be exactly the parameters to the method **including this where available**.
- Interpreters can then be a structural recursion (pattern match) on this data structure.

0. What are these language constructs?
1. What is an abstract syntax tree?
2. Why would we use an abstract syntax tree here?

```scala
// A sealed trait can only be extended within the file in which it is defined.
// If you see one, you are almost certainly looking at an algebraic data type!
sealed trait Stream[A] {
  import Stream._
  // Combinator
  def map[B](f: A => B): Stream[B] =
    // Reifying map
    Map(this, f)
    
  // Interpreter
  def foldLeft[B](z: B)(f: (B, A) => B): B =
    ??? 
}
// Companion object
//
// An object with the same name as a type (type is a trait or class) in the same file as that type.
// Companion objects, by convention, hold constructors
// They have no other relationship to the type. In particular they are *NOT* instances of the type
// Java / C++ equivalent is static methods
object Stream {
  // Represent the map method as data
  final case class Map[A,B](source: Stream[A], f: A => B) extends Stream[B]

  // Constructor
  def emit[A](values: Iterator[A]): Stream[A] =
    ???
}
```


|--------------|-------------------|--------------|
|              | Add new functions | Add new data |
|--------------|-------------------|--------------|
| Sealed Trait | :)                | :(           |
| Open class   | :(                | :)           |
|--------------|-------------------|--------------|

Algebraic data types (sealed traits) are classic FP
Open class (trait or class) is classic OO [Fragile base class?]


```scala
// Math nerds: this is basically proof by induction
def numberOrZero(option: Option[Int]): Int =
  option match {
    case Some(v) => v
    case None => 0
  }
  
def handleExn(exn: Exception): String =
  exn match {
    case e: ArrayIndexOutOfBounds => ???
    ??? // There are an unbounded number of exception subclasses
  }
```
