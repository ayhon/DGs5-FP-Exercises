# Exercise Session 4

This week, we will work on the idea of variance, and on pattern matching.

## Question 1

Recall that
- Lists are covariant in their only type parameter.
- Functions are contravariant in the argument, and covariant in the result.

Consider the following hierarchies:

```scala
abstract class Fruit
class Banana extends Fruit
class Apple extends Fruit
abstract class Liquid
class Juice extends Liquid
```

Consider also the following typing relationships for `A`, `B`, `C`, `D`: `A <: B` and `C <: D`.

Fill in the subtyping relation between the types below. Bear in mind that it might be that neither type is a subtype of the other.

| Left hand side             | ?:  | Right hand side              |
|                       ---: | --- | :---                         |
| List[Banana]               |`<:` | List[Fruit]                  |
| List[A]                    |`<:` | List[B]                      |
| Banana => Juice            |`<:` | Fruit => Juice               |
| Banana => Juice            |`>:` | Banana => Liquid             |
| A => C                     |`  ` | B => D                       |
| List[Banana => Liquid]     |`  ` | List[Fruit => Juice]         |
| List[A => D]               |`  ` | List[B => C]                 |
| (Fruit => Juice) => Liquid |`  ` | (Banana => Liquid) => Liquid |
| (B => C) => D              |`  ` | (A => D) => D                |
| Fruit => (Juice => Liquid) |`  ` | Banana => (Liquid => Liquid) |
| B => (C => D)              |`  ` | A => (D => D)                |

## Question 2

The following data type represents simple arithmetic expressions:

```scala
enum Expr:
    class Number(x: Int)
    class Var(name: String)
    class Sum(e1: Expr, e2: Expr)
    class Prod(e1: Expr, e2: Expr)
```

Define a function `deriv(expr: Expr, v: String): Expr` returning the expression that is the partial derivative of `expr` with respect to the variable `v`.

```scala
def deriv(expr: Expr, v: String): Expr = ???
```

### Answer
```scala 
def deriv(expr: Expr, v: String): Expr = expr match
    case Number(_) => Number(0)
    case Sum(a,b)  => Sum(deriv(a),deriv(b))
    case Prod(a,b) => Sum(Prod(a,deriv(b)),Prod(deriv(a),b))
    case Var(a)  => if a == v then Number(1) else Number(0)
```

Here's an example run of the function:

```scala
> deriv(Sum(Prod(Var("x"), Var("x")), Var("y")), "x")
Sum(Sum(Prod(Var("x"), Number(1)), Prod(Number(1), Var("x"))), Number(0))
```

## Question 3

Write an expression simplifier that applies some arithmetic simplifications to an expression. For example, it would turn the above monstrous result into the following expression:

```scala
Prod(Var("x"), Number(2))
```

### Answer
```scala
def eval(expr: Expr): Expr =
    def bubbleUpSums(expr: Expr): Expr = expr match
        case Sum(a,b)  => Sum(bubbleUpSums(a),bubbleUpSums(b))
        case Prod(a,b) =>
            val lhs = bubbleUpSums(a)
            val rhs = bubbleUpSums(b)
            (lhs, rhs) match
                case (Sum(a,b),c) =>
                    Sum(bubbleUpSums(Prod(a,c)),bubbleUpSums(Prod(a,b)))
                case (c,Sum(a,b)) =>
                    Sum(bubbleUpSums(Prod(a,c)),bubbleUpSums(Prod(a,b)))
                case _ => Prod(lhs, rhs)
        case _ => expr
        
    def fishOutNumbers(expr: Expr): Expr = expr match
        // DFS while taking note of product
        // Also construct the final child:
        //   Prod(Number(a),rest) // This is actually returned
        case Var(a) => (Number(1),expr)
        case Sum(a,b)  => Sum(fishOutNumbers(a),fishOutNumbers(b))
        case Prod(Number(c),rest) => fishOutNumbers(rest) match
            case Prod(Number(d),vars) => 
                Prod(Number(c*d),vars)
            case Number(d) => Number(d*c)

        case Prod(rest,Number(c)) => fishOutNumbers(rest) match
            case Prod(Number(d),vars) => 
                Prod(Number(c*d),vars)
            case Number(d) => Number(d*c)

        case Prod(a,b) => (fishOutNumbers(a), fishOutNumbers(b)) match
            case (Prod(Number(c),varsl),Prod(Number(d),varsr)) =>
                Prod(Number(c*d),Prod(varsl,varsr))
            case (Prod(Number(c),vars),Number(d)) =>
                Prod(Number(c*d),vars)
            case (Number(d),Prod(Number(c),vars)) =>
                Prod(Number(c*d),vars)
        case Num(a) => ??? // Shouldn't happen, nums treated beforehand

        
    def count(x: Expr): Map[Expr, Int] = x match
        case Number(a) => Map()
        case Var(v) => Map(v -> 1)
        case Prod(a,b) =>
            val other_count = count(a)
            count(b).map( (v,c) => (v, c + other_count(v)) )
        case Sum(a,b) => 
            val other_count = count(a)
            count(b).map( (v,c) => (v, c + other_count(v)) )

    def addUpProductSubtrees(ls: List[Expr]) = ls.reduceRight( (x, acc) => Sum(acc,x))

    def lessForProductSubtrees(e1: Expr, e2: Expr)
        def lessForMaps[K,V](m1: Map[K,V],m2: Map[K,V]): Boolean =
            def ls(xs: List[(K,V)], ys: List[(K,V)]): Boolean = (xs,ys) match
                case ((ka,va)::as, (kb,vb)::bs) =>
                    if      ka  < kb                          then true
                    else if ka == kb && va  < vb              then true
                    else if ka == kb && va == vb && ls(as,bs) then true
                case _ => false
            ls(m1.toList,m2.toList)
        lessForMaps(count(e1),count(e2))

    def listOfProductSubtrees(expr: Expr): List[Expr] = expr match
        case Sum(a,b) => listOfProductSubtrees(a) ++ listOfProductSubtrees(b)
        case Var(a) => List(Prod(Number(1),expr))
        case _ => List(expr)

    def combine(ls: List[Expr]): List[Expr] = ls match
        case Nil            => Nil
        case x :: y :: rest => (x,y) match
            case (Product(Number(c),restl),Product(Number(d),restr)) =>
                combine(Product(Number(c+d),restl) :: rest)
            case _ => x :: combine(y :: rest)
        case _ => ls

    addUpProductSubtrees(
        combine(
            listOfProductSubtrees(fishOutNumbers(bubbleUpSums(expr)))
                .sortWith(lessForProductSubtrees)
        )
    )
    // TODO: Perhaps the idea of product sub-trees should be its own trait
```
