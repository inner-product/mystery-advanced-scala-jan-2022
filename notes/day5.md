## ??? Advanced Scala: Day 5

## Cats

- Utilities that make day to day programming easier
- Concepts for system architecture 
- Reusable abstractions to transfer knowledge between domains

Main tools:
- type classes => implicits
- semigroup, monoid
- applicative (semigroupal), monad, traversable
- also, some useful data types (e.g. `NonEmptyList`)


## Cats Effect

A composable abstractions for handling concurrency, asynchronicity, and parallelism.


## Implicits

Three main concepts:
- implicit parameters
- implicit values
- implicit classes (extension method; optional)
- implicit conversion (evil; don't use)

Implicit composition

Debugging implicits

@implicitNotFound annotation

### Implicit Parameters and Values

Implicit parameter:
- final parameter list of a method
- starts with the keyword implicit
- applies to all the parameters in the parameter list
- makes those parameters *implicit parameters*

```scala
def combineAll[A](list: List[A])(implicit monoid: Monoid[A]): A =
  ???
```

`monoid` is an implicit parameter

Can use them like normal parameters: pass values / arguments to them. Nothing special.

If we don't specify an implicit parameter explicitly the compiler will try to provide a value for us.

```scala
def combine[A](x: A, y: A)(implicit sum: (A, A) => A): A = 
  sum(x, y)
```

What values will the compiler attempt to provide? *Implicit values* in the *implicit scope* that match the type of the parameter we are looking for.

Implicit value:
1. `implicit val`
2. `implicit object`
3. `implicit def` where the parameters are also `implicit`. Note the condition on the parameters.

Implicit scope:
1. The normal (lexical) scope. Enclosing {}, the current package, and imports
2. The companion object of any type involved in the method we're calling that has implicit parameters.

Example, in the code below

```scala
trait Combiner {
  def combine[A](x: A, y: A)(implicit sum: (A, A) => A): A = 
    sum(x, y)
}
```

we would look in the companion object for Combiner, the concrete type that A is instantiated to, and Function2.

=> The best place to put implicit values is in companion objects. Then you never need to import them. This requires they are unique (not ambiguous) for a given type.

```scala
final case class Apples(count: Int)
object Apples {
  implicit val addApples: (Apples, Apples) => Apples = 
    (a1, a2) => Apples(a1.count + a2.count)
}

combine(Apples(1), Apples(2))
```


## Packaging Implicits

0. Having multiple implicits with the same type is usually a bad idea. Can easily lead to ambiguity (developer or compiler).
1. Use a companion object if possible
2. Have a well known place where implicits are stored. Usually a separate package, called, e.g. `implicits`. Import `my.system.implicits._` to bring them all into scope.
3. Can turn a normal value into an implicit easily. May be useful for a limited scope

```scala
val theImplicit = theNormalValue

// This isn't used very much. Play (webframework) uses it.
val f: Int => Int = (implicit x: Int) => x * 2 // x is an implicit value in the scope of the function body, but not an implicit parameter
```

Cats:

`import cats.implicits._` is the idiomatic way to get all implicits into scope.

*Need discipline* in packaging implicits. Otherwise things get hard to reason about.


### Implicits UX

For the library designer:

- @implicitNotFound annotation allows you to give more information to user about what might have gone wrong.

For the library user:

- This is annoying. Work out why the implicit value is not found. Start with the overall value you're looking for. If it's composed of smaller values, check those smaller values have implicits of the correct type available. Repeat until you find the missing value.


## Type Classes

Type classes use implicits -> they are pattern build from implicit values and parameters.

Type class : Trait
Type class instance : implicit value implementing the trait
Type class usage : implicit parameter requiring an instance of the trait

Type class is an interface or description of some functionality that we're going to provide for a type.

E.g. 

```scala
// There is a type class called Semigroup
// If we implement it for some type A we must provide the method combine
trait Semigroup[A] {
  def combine(x: A, y: A): A
}
```

Type class instances are implementations of a type class (i.e. implementations of an interface) for a specific type. They are implicit values; they are separate from the type for which they are defined.

```scala
final case class Apples(count: Int) extends Semigroup[Apples] // WRONG!

final case class Apples(count: Int)
object Apples {
  implicit val applesSemigroup: Semigroup[Apples] =  // CORRECT!
    new Semigroup[Apples] { ... }
}
```

```scala
def combine[A](x: A, y: A)(implicit s: Semigroup[A]): A =
  s.combine(x, y)
  
combine(Apples(42), Apples(666)) // Looks in the companion object and finds the type class instance
```

This allows us to define functionality (type class instances; implementations of the interface) separately from the type for which that functionality is defined.
