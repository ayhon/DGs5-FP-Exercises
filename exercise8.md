# Exercise Session 8

## Question 1

In this exercise, we will define instances of the `Eq` type class.
Recall the `Ordering[A]` type class introduced in the lecture.
An instance of `Ordering[A]` allows us to compare values of type `A`
to see which one (if any) is smaller.
Similarly, an instance of `Eq[A]` allows us to compare values of type `A` to see if they are equal.
We can define it as follows:

```scala
trait Eq[T]:
  extension (x: T)
    def === (y: T): Boolean
```

1. Write a `given` instance to create `Eq[List[T]]` from a `Eq[T]`.
```scala
given listEquality[T: Eq]: Eq[List[T]] with
    extension (xs: List[T]) 
        def === (ys: List[T]): Boolean = 
            xs.length == ys.length && xs.zip(ys).forall((p: (T,T)) => p._1 == p._2)
```
2. Write a `given` instance to create `Eq[(T, U, S)]` from `Eq[T]`, `Eq[U]` and `Eq[S]`.
```scala
given tripletEquality[T: Eq, U: Eq, S: Eq]: Eq[(T,U,S)] with
    extension (x: (T,U,S))
        def === (y: (T,U,S)): Boolean = x.toList === y.toList
```
3. Write `given` instance to create `Eq[Person]`. Make use of both the definitions you have written previously.

```scala
case class Person(name: String, age: Int, neighbours: List[String])
```

```scala
given personEquality(using Eq[String], Eq[Int]): Eq[Person] with
    extension (x: Person)
        def === (y: Person) = (x,y) match
            case (Person(a, b, c), Person(d, e, f)) => (a,b,c) === (d,e,f)
```

4. Explicitly write the `using` argument to `summon` (you may need to assign names to your `given` definitions):

```scala
summon[Eq[Person]]
summon[Eq[Person]](using Eq[String], Eq[Int])
```

## Question 2

In this exercise, we will use term inference to calculate types based on other types.
First, we define `Nat`:

```scala
trait Nat

/** Zero */
case object Z extends Nat
type Z = Z.type

/** The Successor of N */
case class S[N <: Nat](pred: N) extends Nat
```

`Nat` encodes natural numbers on the type level:

```scala
S(S(Z)) : S[S[Z]]
```

The type `S[S[Z]]` represents the successor of the successor of zero: two.

We can add two values of type `Nat` as follows:

```scala
def add1(n: Nat, m: Nat): Nat =
  n match
    case Z => m
    case S(n1) => S(add1(n1, m))
```

However, this definition has an issue: the result is imprecisely typed.
To fix this, we will define the `Sum[N, M, R]` type class.
An instance of Sum represents evidence that `N + M = R`.
With the appropriate given definitions, the compiler can infer an instance of `Sum[N, M, R]` such that `N + M = R`, for any pair of natural numbers `N` and `M`:

```scala
case class Sum[N <: Nat, M <: Nat, R <: Nat](result: R)

given zero: Z = Z
given succ[N <: Nat](using n: N): S[N] = S(n)

given sumZ[N <: Nat](using n: N): Sum[Z, N, N] = Sum(n)

given sumS[N <: Nat, M <: Nat, R <: Nat](
  using sum: Sum[N, M, R]
): Sum[S[N], M, S[R]] = Sum(S(sum.result))
```

Note how the last two `given` definitions reflect the definition of the `add1` method: `sumZ` corresponds to the case for `Z + M`, and `sumS` corresponds to the case for `S[N] + M`.
The second case is recursive, explicitly for `add1` and using term inference for the `given` definition.

Using the above definitions, we can write a function that adds two `Nat`
values and assigns a precise type to the result:

```scala
def add[N <: Nat, M <: Nat, R <: Nat](n: N, m: M)(
  using sum: Sum[N, M, R]
): R = sum.result
```

As an example, the result of adding `S[Z]` to `S[Z]` is `S[S[Z]]`:

```scala
sum(S(Z), S(Z)) : S[S[Z]]
```

1. Write explicitly the `using` argument to `sum` (it may help to write all type parameters explicitly):

```scala
add(S(Z), S(S(Z)))(using sum: Sum[S[Z], S[S[Z]], S[S[S[Z]]]])
```

*Note:* If you try this in the Scala 3 compiler right now, the result won't be correct,
this is a known compiler bug: https://github.com/lampepfl/dotty/issues/7586


2. Write `given` definitions that create an instance of the
`Product[N, M, R]` type class, representing the evidence that `N * M = R`.

```scala
case class Product[N <: Nat, M <: Nat, R <: Nat](result: R)
```

Hint 1: Multiplying two natural numbers is defined as follows:

```scala
def mul1(n: Nat, m: Nat): Nat =
  n match
    case Z => Z
    case S(n1) => add(m, mul1(n1, m))
```

Hint 2: The `R` type parameter is key to making the `add` definition work.
When we start inferring a term of type `Sum[N, M, R]`, we already know what
`N` and `M` are, but we don't know what `R` is. Inferring the term will also
infer the type that stands for `R`.

```scala
extension (n: Nat) def + (m: Nat) = add(n,m)

def mul1(n: Nat, m: Nat): Nat = n match
    case Z => Z
    case S(prev) => m + mul1(prev, m) // 

case class Product[N <: Nat, M <: Nat, R <: Nat](result: R)

given multZ: [M <: Nat](using m : M): Product[Z, M, Z] = Product(Z)
given multS: [M <: Nat](
  using Product[N, M, R], sum: Sum[M,R,P]
): Product[S[N],M,P] = Product(sum.result)

/** Proper add function (Returns of proper type) **/
def mul[N <: Nat, M <: Nat, R <: Nat](n: N, m: M)(
  using prod: Product[N, M, R]
): R = prod.result

```

#### Given (implicit) code
```scala
trait Nat

/** Zero */
case object Z extends Nat
type Z = Z.type

/** The Successor of N */
case class S[N <: Nat](pred: N) extends Nat

/** Naive add function (Returns of type Nat) **/
def add1(n: Nat, m: Nat): Nat = n match
  case Z => m
  case S(n1) => S(add1(n1, m))

case class Sum[N <: Nat, M <: Nat, R <: Nat](result: R)

/** Implicits of natural type **/
given zero: Z = Z
given succ[N <: Nat](using n: N): S[N] = S(n)

/** Implicits for the sum **/
given sumZ[N <: Nat](using n: N): Sum[Z, N, N] = Sum(n)

given sumS[N <: Nat, M <: Nat, R <: Nat](
  using sum: Sum[N, M, R]
): Sum[S[N], M, S[R]] = Sum(S(sum.result))

/** Proper add function (Returns of proper type) **/
def add[N <: Nat, M <: Nat, R <: Nat](n: N, m: M)(
  using sum: Sum[N, M, R]
): R = sum.result
```

## Question 3
Extra. Implement the integers in the same way
```scala
trait Ent // Integers is Enteros in Spanish
trait ZeroOrSuccessors extends Ent
trait ZeroOrPredecessors extends Ent

case object Z extends Ent with ZeroOrSuccessors with ZeroOrPredecessors
case class S[N <: ZeroOrSuccessors](prev: N) extends ZeroOrSuccessors
case class P[N <: ZeroOrPredecessors](succ: N) extends ZeroOrPredecessors

def successor[N <: ZeroOrPredecessors](n: P[N]): N = n match
  case P(n) => n
def successor[N <: ZeroOrSuccessors](n: N): S[N] = S(n)

def predecessor[N <: ZeroOrSuccessors](n: S[N]): N = n match
  case S(n) => n
def predecessor[N <: ZeroOrPredecessors](n: N): P[N] = P(n)

// TODO: From here onwards it doesn't work. Only asking for
// Ent is too little. It's perhaps better to just separate
// the cases
def add_naive[N <: Ent, M <: Ent](n: N, m: M): Ent = n match
  case Z => m
  case S(prev) => add_naive(prev, successor(m))
  case P(succ) => add_naive(succ, predecessor(m))

def negative[N <: Ent](n: N): Ent = n match
  case Z => Z
  case S(prev) => predecessor(negative(prev))
  case P(succ) => successor(negative(succ))
```
