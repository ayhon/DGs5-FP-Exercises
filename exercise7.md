# Exercise Session 7

## Question 1

Consider the following series:

```
1
1 1
2 1
1 2 1 1
1 1 1 2 2 1
3 1 2 2 1 1
...........
```


1. Find the next element in the sequence above.

Now, let us encode an element of the sequence above as a `List[Int]`.


2. Write a function to compute the next element.

```scala
def nextLine(currentLine: List[Int]): List[Int] =
    currentLine.foldLeft(Nil)((acc: List[Int], elem: Int) => acc match
        case  last :: count :: rest => if last == elem 
            then last :: (count+1) :: rest 
            else elem :: 1 :: acc
        case _ => elem :: 1 :: acc
    ).reverse
```
```scala
def nextLine(currentLine: List[Int]): List[Int] =
    currentLine.foldRight(Nil)((elem: Int, acc: List[Int]) => acc match
        case  count :: last :: rest => if last == elem 
            then (count+1) :: last :: rest 
            else 1 :: elem :: acc
        case _ => 1 :: elem :: acc
    )
```

3. Implement a lazy list `funSeq` which constructs this sequence. Recall: to construct a lazy list, you can use `LazyList.cons[A](a: A, b: => LazyList[A]): LazyList[A]`

```scala
lazy val funSeq: LazyList[List[Int]] = List(1) #:: nextLine(funSeq)
```

## Question 2

```scala
def from(n: Int): LazyList[Int] = n #:: from(n+1)
```

1. Write a lazy list of squares of integers ≥ 1. You may use `LazyList.from(i: Int)`

```scala
val squares: LazyList[Int] = LazyList.from(1).map(x => x*x)
```

2. Write a lazy list of all non-empty strings using the characters "0" and "1" and the concatenation operation +. In other words, every non-empty string composed of "0" and "1" should be reached at some point.

```scala
val codes: LazyList[String] = "" #:: codes.flatMap(s => LazyList(s ++ "0", s ++ "1"))
```

3) Using `codes`, write a lazy list of all possible non-empty palindromes of "0" and "1". You may use the `.reverse` function defined on strings.

```scala
val palCodes: LazyList[String] = codes.filter(s => s == s.reverse)
```
4. Can you do the same without filtering? The palindromes need not to be in the same order.
```scala
val palCodes: LazyList[String] = "" #:: "0" #:: "1" #:: palCodes.flatMap(s => LazyList(s"0${s}0",s"1${s}1"))
```

5. Given another lazy list `otherCodes`, possibly finite or infinite, you don't know at first:

```scala
val otherCodes: LazyList[String] = [some external source]
```

Build a lazy list `allCodes` that interleaves `palCodes` and `otherCodes`.

```scala
lazy val allCodes: LazyList[String] = interleave(palCodes, otherCodes)

def interleave(as: LazyList[String], bs: LazyList[String]): LazyList[String] =
    (as, bs) match
        case (Nil, Nil) => Nil
        case (x #:: xs, Nil) => as
        case (Nil, y #:: ys) => bs
        case (x #:: xs, y #:: ys) => x #:: y #:: interleave(xs, ys)
```
