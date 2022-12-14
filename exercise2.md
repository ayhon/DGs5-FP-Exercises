# Exercise Session 2

This week we will work on playing with functions as values.

## Question 1.

Define the function `flip`. It takes a function and returns the same function, but with the arguments flipped.

```scala
def flip(f: (Int, Double) => Int): (Double, Int) => Int = (y: Double, x: Int) => f(x,y)
```

## Question 2.1

Define the identity function for integers, which, given an `Int`, returns it.

```scala
val id: Int => Int = (x: Int) => x
```

## Question 2.2

Define the compose function, that, given 2 functions `f`, `g`, returns a function that composes them, i.e., `f ∘ g`.

```scala
def compose(f: Int => Int, g: Int => Int): Int => Int = (x: Int) => f(g(x))
```

What does `compose(id, f)(k)` evaluate to, for some function `f` and integer `k`?

## Question 2.3

Define the function `repeated`, which takes a function and repeatedly applies it `n` times (`n ≥ 0`).

```scala
def repeated(f: Int => Int, n: Int): Int => Int = (x: Int) => if n == 0 then x else f(repeated(f,n-1)(x))
```
### Enhacement: Make it tail recursive
```scala
def repeated(f: Int => Int, n: Int): Int => Int =
    def loop(n: Int)(acc: Int): Int =
        if n == 0 then 
            acc
        else 
            loop(n-1, f(acc))
    loop(n)
```

_Hint_: What values should be returned by `repeated(x => x + 1, 0)` and `repeated(x => x + 1, 3)`?

## Question 3.1

Define the function `curry2`, that curries a two arguments function. That is, `curry2(f) = g` such that `f(x, y) == (g(x))(y)`

```scala
def curry2(f: (Double, Int) => Boolean): Double => (Int => Boolean) = (x: Double)(y: Int) => f(x,y)
```

_Hint_: what should `curry2((x, y) => x < y)(1.0)` return?


## Question 3.2

Define the function `uncurry2`. It takes a curried function, and creates a two-argument function.

```scala
def uncurry2(f: Double => Int => Boolean): (Double, Int) => Boolean = (x: Double, y: Int) => f(x)(y)
```

## Question 4.

Write a function `fixedPoint` with the following signature:

```scala
def fixedPoint(f: Int => Int): Int => Int
```

The function takes a function `f` and returns a function that maps an integer into the fixed point of f that is obtained by iterating `f` some finite number of times starting from the initial value.

A value `x` is a fixed point of `f` if `f(x) == x`.

```scala
val MAX_COUNTER = 1000
def fixedPoint(f: Int => Int): Int => Int =
    @tailrec
    def loop(counter: Int)(x: Int): Int = 
        val fx = f(x)
        if fx == x or counter == 0 then 
            x 
        else 
            loop(counter-1)(fx)
    loop(MAX_COUNTER)
```

For each of the following expressions, indicate whether it terminates, and if so, what is the value returned:

- `fixedPoint(x => x/2)(4)` returns `0`
- `fixedPoint(id)(123456)` returns `123456`
- `fixedPoint(x => x + 1)(0)` returns `MAX_COUNTER`
- `fixedPoint(x => if (x % 10 == 0) x else x + 1)(35)` returns `40`
- `fixedPoint((x: Int) => x / 2 + 5)(20)` returns `10`

## Question 5.1

Write the `sum` function with the following signature:

```scala
def sum(a: Int, b: Int)(f: Int => Int): Int =
    @tailrec
    def loop(n: Int, acc: Int): Int =
        if n < b then acc else loop(n+1, f(n)+acc)
    loop(a,0)
```

Which returns the sum of `f(i)` where `i` ranges from `a` to `b`.

_Bonus point_: Can your implementation be tail recursive ?

## Question 5.2

Write the `quadratic` function with the following signature:

```scala
def quadratic(c: Int): Int => Int = (x: Int) => (x-c)*(x-c)
```

Which returns a function that takes an integer `x` as argument and returns `(x - c)²`.

## Question 5.3

Using the above functions, define the function `quad3Integrate` which, given two integers `a` and `b`, computes the sum of `(i - 3)²`  where `i` ranges from `a` to `b - 1`.

```scala
def quad3Integrate(a: Int, b: Int): Int = sum(a,b)(quadratic(3))
    
val quad3Integrate: (Int, Int) => Int = (x:Int, y: Int) => quad3Integrate(x,y)
```

