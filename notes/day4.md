# ??? Advanced Scala: Day 4

- Solving filter
- Interleave
- Merge: generalized algebraic data types, and Scala 2 workarounds
- Zip: state
- Performance

## Co- and Contravariance

Subtyping
- If A <: B (A is a subtype of B)
  Then, whenever we expect a B we accept an A
  
A extends B => A <: B

What is the subtyping relationship between F[A] and F[B], given some subtyping relationship between A and B?

E.g.
trait Fruit
case class Orange extends Fruit
case class Apple extends Fruit

Is List[Orange] <: List[Fruit]?


Choices:
- If A <: B, then F[A] <: F[B] *Covariance*
- If A >: B, then F[A] <: F[B] *Contravariance*
- We don't care about the relationship between A and B, F[A] is never a subtype of F[B] *Invariance*

If List is covariant then List[Orange] <: List[Fruit]
If List is contravariant then List[Fruit] <: List[Orange]
If List is invariant then there is no relationship between List[Fruit] and List[Orange]

In Scala, when we declare a type parameter (generic type) we can declare variance:
- [+A] = covariant
- [-A] = contravariant
- [A] = invariant

Immutable containers are covariant (outputs)
Mutable containers are invariant
Things that accept data but don't return it are contravariant (inputs)

```scala
val oranges: Array[Orange] = Array(Orange, Orange, Orange)

def buyFruit(basket: Array[Fruit]): Unit =
  basket(0) = Apple

buyFruit(oranges)

oranges(0) // This is an Apple ?!?!?!?
// Java used to allow this!
```

// Kid => Clean
// Animal => Clean
// KidTheYounger => Clean

Animal => Clean <: Kid => Clean
Animal >: Kid

Functions and methods are contravariant in their inputs (and covariant in their output)

// Kid <: Animal
// KidTheYounger <: Kid
// KidTheElder <: Kid

Vanilla <: Icecream <: Snack

Icecream => PreparedFood
Snack => PreparedFood
Vanilla => PreparedFood

CreditCardCheck <: PurchaseEvent <: UserEvent

object SerializePurchases {
  def serialize(event: PurchaseEvent, serializer: PurchaseEvent => Array[Byte]): Unit =
    cassandra.write(serializer(event))
}
PurchaseEvent => Array[Byte] <-- This is what we want
CreditCardCheck => Array[Byte] <-- Too specific
UserEvent => Array[Byte] <-- This will do. This is contravariant *in the input*

Function[In, Out]


## General Principles

- outputs are covariant (containers)
- inputs are contravariant (function parameters)
- mutable state is invariant (array)


## Variance and Algebraic Data Types

See Essential Scala for details

Nothing is a subtype of all types.
Any is a supertype of all types.

Invariant

```scala
// Option is a Some or None
sealed trait Option[A]
final case class Some[A](value: A) extends Option[A]
final case class None[A]() extends Option[A]

```

All instances of None are the same, but we allocate a new one every time we need one.

Can we do better?

```scala
sealed trait Option[A]
final case class Some[A](value: A) extends Option[A]
case object None extends Option[Nothing]
```

Only one instance of None. No excess allocation.

Need covariance!

```scala
sealed trait Option[+A]
final case class Some[A](value: A) extends Option[A]
case object None extends Option[Nothing]
```

Option[Nothing] <: Option[A] for all A, because Nothing <: A

Rules for ADTs that are containers with generic types
1. Make the generic type covariant
2. Cases that have no data can be implemented as case objects and supply `Nothing` as the type parameter
```scala
case object Await extends Response[Nothing]
```
3. Methods that have an input of the type parameter need a new type parameter that is a super type. 
```scala
def getOrElse[AA >: A](otherwise: AA): AA
```


## Generalized Algebraic Data Types

GADT = Combine concrete type and type variables in `extend`

Scala 2 has type inference bugs relating to pattern matching and GADTs.

To fix them:

1. Implement structural recursion using dynamic dispatch (OO style) instead of pattern matching

2. Pattern matching 

Pattern for GADTs need to be of the form

```scala
case m: Merge[a, b] => ???
```

for a GADT `Merge` with two type variables

The overall type we match on needs to have any type varables introduced on a method, not on a class or trait.

```scala
def helper[B](stream: Stream[B]): Response[B] =
  ???
```


```scala
sealed trait Fruit
final case class Apple(count: Int) extends Fruit
final case class Orange(count: Int) extends Fruit

fruit match {
  case Apple(c) => ???
  case Orange(c) => ???
}

fruit match {
  case a: Apple => a.count // the name a with type Apple is available in the RHS
  case o: Orange => o.count // the name o with type Orange is available in the RHS
}
```
