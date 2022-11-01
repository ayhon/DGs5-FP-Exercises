# Exercise Session 11

In these exercises, you are asked to write higher-order functions in the simple untyped language supported by the interpreter for recursive higher-order functions that we developed in the lectures.

## Question 1

In this exercise, you will be working with _Church numerals_.

Church numerals are a representation of natural numbers using only functions.
In this encoding, a number `n` is represented by a function that maps any function `f` to its `n`-fold composition.

For example, `0`, `1`, `2` and `3` are represented as follows:

```scala
def zero   = (f => x => x)
def one    = 
    (f => 
        x => 
            f x)
zero(f)(x) == x
one(two)(one) →
(f => x => f x)(two)(one) →
(x => two(x))(one) →
two(one) 
(f => x => f(f(x)))(one)
x => one(one(x))

\ f x . f x
\ f x . f f x

def succ(n: Fun) =
    f => x =>
        f( n(f)(x) )


two(f)(x) == f (f x)
three(f)(x) == f (f (f x))

def two    = (f => x => f (f x))

def three  = (f => x => f (f (f x)))
```

### Question 1.1

Give an implementation of the `succ` function that takes a Church numeral and returns its successor.

```scala
def succ = n => f => x => f(n(f)(x))
```

For example, `(succ zero)` evaluates to the definition of `one` and `(succ one)` evaluates to the definition of `two`.

### Question 1.2

Give an implementation of the `add` function that takes two Church numerals and returns their sum, using `succ`.

```scala
def add = n => m => f => x => n(f)( m(f)(x) )
```

## Question 2

In this exercise, you will be working with _Church-encoded lists_.

Much like Church numerals, Church-encoded lists are a representation of lists using only functions.

In this encoding, a list is represented by a function that takes two arguments and
returns the first one if the list is empty or returns the second one applied to head and tail if the list is non-empty:

```scala
def nil  = (n => c => n)
def cons = (h => t => n => c => c h t)
```

For example, `List(1,2,3)` would be represented in this encoding as follows:

```scala
(cons 1 (cons 2 (cons 3 nil)))
(h => t => n => c => c h t)(1)(cons 2 (cons 3 nil))
c 1 (cons 2 (cons 3 nil))

l1 (0) (acc => x => add(acc)(x))

// If list is empty
(n => c =>  n)(0)(acc => x => add(acc)(x))

// If list is not empty
(n => c =>  c (h) (t) )(0) (acc => x => add(acc)(x))
(x => acc => add(x)(acc)) (h1) (cons h2 t2)


object obj extends Ordering[Int]:
    def compare(a: Int)

object obj2 extends Ordering[Int]:
    def compare(a: Int)

given obj

f // With obj
f(using ojb2)

def f(using leq: Ordering[Int]) = 
    x < y  ---> leq.compare(x,y)

f(using greaterequal)

```
l1.foldRight(l2)( (x, acc) => cons(x)(acc))

With Church-encoded lists, decomposition is achieved by "applying" the list to a pair of continuations, one for the empty case and another one for the non-empty case. For example, concatenation of two Church-encoded lists could be implemented as follows:

```scala
def cat = (l1 => l2 =>
             (l1 l2 (h => t => (cons h (cat t l2))))
```

### Question 2.1

Give an implementation of the `size` function that takes a Church-encoded list and returns its size as a Church numeral. You are allowed to use the `succ` function defined earlier.

```scala
def size = l1 =>
    l1 (zero) (h => n => succ(n))

```
### Question 2.2

Give an implementation of the `map` function which takes a Church-encoded list and a function and returns the list mapped with the function (in the sense of `List.map`). You may use recursion in your definitions.

### Question 2.3

Give an implementation of the `foldRight` function which takes a Church-encoded list and a function and returns the result of `foldRight`. You may use recursion in your definitions.

