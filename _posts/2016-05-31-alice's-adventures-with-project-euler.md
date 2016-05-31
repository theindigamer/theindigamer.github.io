---
layout: post
title: "Alice's Adventures with Project Euler: Haskell, Python and Rust"
categories: [programming]
tags:
- haskell
- python
- rust
- project euler
---

[projecteuler]:  https://projecteuler.net
[wiki-lazyeval]: https://en.wikipedia.org/wiki/Lazy_evaluation
[mathse-pe106]:  https://math.stackexchange.com/questions/1723048/project-euler-106-necessary-and-sufficient-conditions
[reddit-rust]:   https://www.reddit.com/r/rust
[irc-rust]:      https://chat.mibbit.com/?server=irc.mozilla.org%3A%2B6697&channel=%23rust
[crates.io]:     http://doc.crates.io/
[wiki-purity]:   https://en.wikipedia.org/wiki/Purely_functional
[hoogle]:        https://www.haskell.org/hoogle/?hoogle=
[matplotlib]:    http://matplotlib.org/
[sympy]:         http://www.sympy.org/en/index.html
[numpy]:         http://www.numpy.org/
[rouge-issue]:   https://github.com/jneen/rouge/issues/287
[rouge-PR]:      https://github.com/jneen/rouge/pull/452

### Prologue

After her adventures in the strange and esoteric Wonderland, Alice was in a bad mood. She refused to venture outside, lest she fall into a rabbit hole again. Instead, she would sit in front of the computer all day, reading all there was to read on the internet. One of her favourite pastimes was to read about mathematics. Her father, ever so keen on turning Alice into a proper lady, appointed a governess to teach her literature, history and philosophy. Alice would often fight with the governess; the governess would make her read Hegel and Kierkegaard, whereas Alice was much more interested in the works of Bertrand Russell and Wittgenstein.

For every little task the governess assigned to Alice, Alice would do the exact opposite if it were possible to do so. Finally, the governess was fed up.
She told Alice, "I will give you permission to study the life and work of any one person you may desire. It may even be a mathematician (hmph!). However, you must prepare a thorough report describing everything you've read about that person, no shorter than thirty pages." Alice was not amused. However, she grudgingly accepted. After all, 1 > 0.
Of course, that doesn't mean that she intended to write the report by herself. As all good students do, she first checked whether such a report was already available on the internet. Since Euler was her favourite mathematician (he was the first one to solve the Basel problem!), she typed in "Euler project report" and hit the first [link][projecteuler] she encountered without thinking too much. It said:

> What is Project Euler?
> 
> Project Euler is a series of challenging mathematical/computer programming problems that will require more than just mathematical insights to solve. Although mathematics will help you arrive at elegant and efficient methods, the use of a computer and programming skills will be required to solve most problems.
> 
> The motivation for starting Project Euler, and its continuation, is to provide a platform for the inquiring mind to delve into unfamiliar areas and learn new concepts in a fun and recreational context.

Alice's interest was slightly piqued. She loved challenging mathematical problems but she didn't know programming. Her friend Bob did. So she wrote the following email to Bob:

> Bob,
> 
> I've stumbled across this site called Project Euler. Do you know about it?
> It has a large number of nice mathematics/programming problems.
> Can you recommend a fun programming language to learn for solving them?
> I'll pick up the details from tutorials online (duh!), but I just need a suggestion to get started.
> 
> Bye,
>
> A

Bob was surprised. Alice generally expressed distaste for using external aid in solving problems. He wrote back promptly:

> Hi Alice,
> 
> Oh, so you've finally written to me about programming.
> Despite all the arithmetic you can do in your head, I'm happy that you've realized that this is too much for you.
> Unfortunately, given that there are so many fun programming languages out there, it is hard to recommend just one.
> So I've selected three (amongst the many I know :P) that you'll probably enjoy. In no particular order,
> 
> * Haskell: Perfect for budding mathematicians. I know you'll love it.
> * Python: You might want to start with this for a gentle introduction to programming :P.
> * Rust: This one is hard to describe to a non-programmer like you :-/. Just read the official description.
> 
> Let me know which one you like the most!
> 
> Best,
> 
> Bob

Finally, Alice set out to randomly solve a few problems from the Project Euler database. She tried all the languages that Bob had suggested. She also tried solving some problems using multiple languages. While several of the problems were trivially easy, she was diligent enough to write down short notes for all the problems describing the syntax and her programming experience.

### Haskell

#### Problem 4

> A palindromic number reads the same both ways. The largest palindrome made from the product of two 2-digit numbers is 9009 = 91 × 99.
> 
> Find the largest palindrome made from the product of two 3-digit numbers.

Solution to problem 4 (0.005s):

```haskell
palindrome x = (show x == reverse (show x))
main = print 
  (maximum 
    (filter palindrome [y1*y2 | let x = [999, 998 .. 100], y1 <- x, y2 <- x]))
```

Notes: `show` converts stuff to a `string`. `filter` filters the elements in the list which give `palindrome x == True`. `<-` represents the set membership ∈. The solution is really clean and easy to understand. I tried rewriting it in Python and Rust later but I like the Haskell solution the best.

#### Problem 14

> The following iterative sequence is defined for the set of positive integers:
> 
> n → n/2 (n is even)
> n → 3n + 1 (n is odd)
> 
> Using the rule above and starting with 13, we generate the following sequence:
> 13 → 40 → 20 → 10 → 5 → 16 → 8 → 4 → 2 → 1
> 
> It can be seen that this sequence (starting at 13 and finishing at 1) contains 10 terms. Although it has not been proved yet (Collatz Problem), it is thought that all starting numbers finish at 1.
> 
> Which starting number, under one million, produces the longest chain?
> 
> NOTE: Once the chain starts the terms are allowed to go above one million.
 
Solution to problem 14 (4.7s):

```haskell
collatz :: Int -> Int
collatz n
    | n `rem` 2 == 0 = n `div` 2
    | otherwise = 3*n+1

collatz_len :: Int -> Int
collatz_len 1 = 1
collatz_len n = 1 + collatz_len (collatz n)

main = print (maximum (map collatz_len [1, 2 .. 999999]))
```

Notes: I've specified the types of functions for convenience. The pipes `|` act as conditional statements `if`, `else` etc. `map` applies `collatz_len` to all elements of the list. Again, writing the code is very straight-forward from the problem statment.

#### Problem 25

> The Fibonacci sequence is defined by the recurrence relation:
>
>    F<sub>n</sub> = F<sub>n−1</sub> + F<sub>n−2</sub> , where F<sub>1</sub> = 1 and F<sub>2</sub> = 1.
>
> Hence the first 12 terms will be:
> 
>   F<sub>1</sub> = 1,
>   F<sub>2</sub> = 1,
>   F<sub>3</sub> = 2,
>   F<sub>4</sub> = 3,
>   F<sub>5</sub> = 5,
>   F<sub>6</sub> = 8,
>   F<sub>7</sub> = 13,
>   F<sub>8</sub> = 21,
>   F<sub>9</sub> = 34,
>   F<sub>10</sub> = 55,
>   F<sub>11</sub> = 89,
>   F<sub>12</sub> = 144.
>
> The 12th term, F<sub>12</sub> , is the first term to contain three digits.
> 
> What is the index of the first term in the Fibonacci sequence to contain 1000 digits?

Solution to problem 25 (0.26s):

```haskell
fibs :: [Integer]
fibs = 1:1:zipWith (+) fibs (tail fibs)

fib :: Int -> Integer
fib n = fibs !! (n-1)

fibdigs n = (n, length (show (fib n)))

main = print (head (filter ((==1000).snd) (map fibdigs [1, 2 .. ])))
```

Notes: The Fibonacci numbers are arbtrary precision `Integer`s. `zipWith` takes a function `f` and two lists and creates a new list by applying `f` to pairs of elements. `!!` selects an element from the list. `==1000` creates a partial function. `.` denotes the function composition ∘. `snd` picks out the second element of a tuple. Here, we are using an infinite list `[1, 2 .. ]` which is [lazily evaluated][wiki-lazyeval]. As with the earlier two problems, writing the code is very easy from the problem statement with the only clever part being the construction of the Fibonacci numbers.

#### Problem 104

> The Fibonacci sequence is defined by the recurrence relation:
> 
> F<sub>n</sub> = F<sub>n−1</sub> + F<sub>n−2</sub> , where F<sub>1</sub> = 1 and F<sub>2</sub> = 1.
> 
> It turns out that F<sub>541</sub> , which contains 113 digits, is the first Fibonacci number for which the last nine digits are 1-9 pandigital (contain all the digits 1 to 9, but not necessarily in order). And F<sub>2749</sub> , which contains 575 digits, is the first Fibonacci number for which the first nine digits are 1-9 pandigital.
> 
> Given that F<sub>k</sub> is the first Fibonacci number for which the first nine digits AND the last nine digits are 1-9 pandigital, find k.

Solution to problem 104 (~41 min):

```haskell
import Data.Char (digitToInt)
import Data.List (sort)

fibs :: [Integer]
fibs = 1:1:zipWith (+) fibs (tail fibs)
fib :: Int -> Integer
fib n = fibs !! (n-1)

first_9_digits :: Integer -> [Int]
first_9_digits x = map digitToInt (take 9 (show x))
last_9_digits x = map digitToInt (take 9 (reverse (show x)))

pandigital_ends :: Int -> (Int, Bool)
pandigital_ends n
    | [1 .. 9] == (sort f9) && [1 .. 9] == (sort l9) = (n, True)
    | otherwise = (n, False)
    where f = fib n
          f9 = first_9_digits f
          l9 = last_9_digits f

main = print (head [x | x <- map pandigital_ends [1, 2 ..], snd x])
```

Notes: The `where` clause allows short substitutions, much like `let` earlier. This solution is fairly slow probably because a lot of time is being wasted in actually evaluating all the Fibonacci numbers.

### Python

#### Problem 69

> Euler's Totient function, φ(n) [sometimes called the phi function], is used to determine the number of numbers less than n which are relatively prime to n. For example, as 1, 2, 4, 5, 7, and 8, are all less than nine and relatively prime to nine, φ(9)=6.
> 
> | n 	| Relatively Prime | φ(n) | n/φ(n) |
> |:---:|:-----------------|:-----|:-------|
> | 2 	| 1 	           |   1  | 2      |
> | 3 	| 1,2 	           |   2  | 1.5    |
> | 4 	| 1,3 	           |   2  | 2      |
> | 5 	| 1,2,3,4 	       |   4  | 1.25   |
> | 6 	| 1,5 	           |   2  | 3      |
> | 7 	| 1,2,3,4,5,6 	   |   6  | 1.1666... |
> | 8 	| 1,3,5,7 	       |   4  | 2      |
> | 9 	| 1,2,4,5,7,8 	   |   6  | 1.5    |
> | 10 	| 1,3,7,9 	       |   4  | 2.5    |
> 
> It can be seen that n=6 produces a maximum n/φ(n) for n ≤ 10.
> 
> Find the value of n ≤ 1,000,000 for which n/φ(n) is a maximum.

Solution to problem 69 (PyPy: 75s)

```python
maxnum = 1000000

totient_table = ['X' for x in range(0,maxnum+1)]
totient_table[1] = 1
totient_table[2] = 1

def gcd(a,b):
    while b!= 0:
        t = b
        b = a % b
        a = t
    return a

primes = [2]

def totient(a):
    if a == 2:
        return 1
    for b in primes:
        if a % b == 0:
            c = a
            d = 1
            while (c % b) == 0:
                c = c / b
                d = d * b
            assert(gcd(c,d) == 1)
            if(d == a):
                totient_table[a] = a - a / b
                return totient_table[a]
            totient_table[a] = totient_table[c]*totient_table[d]
            return totient_table[a]
    # Otherwise `a` is a prime
    totient_table[a] = a-1
    primes.append(a)
    return totient_table[a]

a, b, n = 1, 1, 2
for x in range(2,maxnum+1):
    a = float(x)/totient(x)
    if a > b:
        b = a
        n = x

print(b,n,totient_table[maxnum])
```

Notes: Actually, this problem can be solved by hand if one sees Euler's own simplified formula for the totient function. And I used to think he was my favourite. :'( 

#### Problem 102

> Three distinct points are plotted at random on a Cartesian plane, for which -1000 ≤ x, y ≤ 1000, such that a triangle is formed.
> 
> Consider the following two triangles:
> 
> A(-340,495), B(-153,-910), C(835,-947)
> 
> X(-175,41), Y(-421,-714), Z(574,-645)
> 
> It can be verified that triangle ABC contains the origin, whereas triangle XYZ does not.
> 
> Using triangles.txt (right click and 'Save Link/Target As...'), a 27K text file containing the co-ordinates of one thousand "random" triangles, find the number of triangles for which the interior contains the origin.
> 
> NOTE: The first two examples in the file represent the triangles in the example given above.

Solution to problem 102 (Python2.7: 0.05s):

```python
def haszero(sixpoints):
    ax, ay = sixpoints[0], sixpoints[1]
    bx, by = sixpoints[2], sixpoints[3]
    cx, cy = sixpoints[4], sixpoints[5]
    s1, s2, s3 = ax*by - ay*bx, bx*cy - by*cx, cx*ay - cy*ax
    if s1 != 0 and s2 != 0 and s3 != 0:
        return ((s1/abs(s1)) == (s2/abs(s2)) and (s2/abs(s2)) == (s3/abs(s3)))
    else:
        return False

count = 0
with open('p102_triangles.txt', 'r') as f:
    for line in f:
        line = line[:-1]
        sixpoints = line.split(',')
        sixpoints = [float(x) for x in sixpoints]
        if haszero(sixpoints): 
            count = count + 1

print(count)
```

Notes: File I/O and string manipulation is very easy with Python. The usual `for a in b` syntax also works for files.

#### Problem 106

> Let S(A) represent the sum of elements in set A of size n. We shall call it a special sum set if for any two non-empty disjoint subsets, B and C, the following properties are true:
> 
> 1. S(B) ≠ S(C); that is, sums of subsets cannot be equal.
> 2. If B contains more elements than C then S(B) > S(C).
> 
> For this problem we shall assume that a given set contains n strictly increasing elements and it already satisfies the second rule.
> 
> Surprisingly, out of the 25 possible subset pairs that can be obtained from a set for which n = 4, only 1 of these pairs need to be tested for equality (first rule). Similarly, when n = 7, only 70 out of the 966 subset pairs need to be tested.
> 
> For n = 12, how many of the 261625 subset pairs that can be obtained need to be tested for equality?
> 
> NOTE: This problem is related to Problem 103 and Problem 105.

Solution to problem 106 (Python2.7: 1.2s)

```python
from scipy.misc import comb

def check_not_reqd(b,c):
    x = True
    for (bn,cn) in zip(b,c):
        x = x and (bn < cn)
    return x

def recurse(b, length, depth, start, end):
    count = 0
    if depth == 0:
        for b[length-1] in range(start, end):
            c = [x for x in range(2*length) if x not in b]
            if (not check_not_reqd(b,c)):
                count = count + 1
        return count
    else:
        for b[length-1-depth] in range(start, end):
            count = count + recurse(b, length, depth-1, b[length-1-depth]+1, end+1)
        return count

print(sum([comb(12,t)*recurse([0 for s in range(t/2)],t/2,t/2-1,0,t/2) 
    for t in [4,6,8,10,12]]))
```

Notes: We need to check the pair (B,C) if and only if we don't have \|B\|=\|C\| and we don't have the indices of C in A dominating the indices of B in A element-wise (or vice-versa). So the solution is obtained by constructing all the partitions of indices which don't satisfy the two conditions.

Here is an expanded version of the `recurse` function for `t=8`.

```python
tot = 8
count = 0
b,c = [0,0,0,0], [0,0,0,7]
start = 0
end = tot/2
for b[0] in range(start,end):
    for b[1] in range(b[0]+1,end+1):
        for b[2] in range(b[1]+1,end+2):
            for b[3] in range(b[2]+1,end+3):
                c = [x for x in range(0, tot) if x not in b]
                if(not check_not_reqd(b,c)):
                    count = count + 1
print(count)
```

Author's note: Consider the "if and only if" statement posed by Alice as P⇔Q. Although the case ¬Q⇒¬P is obvious, the other way around isn't (at least to me). So I posted the following question on Math StackExchange: [Project Euler 106: Necessary and Sufficient Conditions][mathse-pe106].

### Rust

#### Problem 23

> A perfect number is a number for which the sum of its proper divisors is exactly equal to the number. For example, the sum of the proper divisors of 28 would be 1 + 2 + 4 + 7 + 14 = 28, which means that 28 is a perfect number.
>
> A number n is called deficient if the sum of its proper divisors is less than n and it is called abundant if this sum exceeds n.
>
> As 12 is the smallest abundant number, 1 + 2 + 3 + 4 + 6 = 16, the smallest number that can be written as the sum of two abundant numbers is 24. By mathematical analysis, it can be shown that all integers greater than 28123 can be written as the sum of two abundant numbers. However, this upper limit cannot be reduced any further by analysis even though it is known that the greatest number that cannot be expressed as the sum of two abundant numbers is less than this limit.
>
> Find the sum of all the positive integers which cannot be written as the sum of two abundant numbers.

Solution to problem 23 (0.25s):

```rust
fn main() {
    const LIM: usize = 28123;
    let mut abundant = vec![true; LIM+1];
    let mut sum = vec![0; LIM+1];
    for i in 1 .. LIM+1 {
        for j in 1 .. LIM/i+1 {
            sum[i*j] += i;
        }
        if sum[i] <= 2*i {
            abundant[i] = false;
        }
    }
    //let mut x = Vec::new();
    let mut y = 0;
    let mut expressible_as_sum_of_two_abundant_nos = false;
    for i in 1 .. LIM+1 {
        for j in 1 .. i {
            if abundant[j] && abundant[i-j] {
                expressible_as_sum_of_two_abundant_nos = true;
                break;
            }
        }
        if !expressible_as_sum_of_two_abundant_nos {
            //x.push(i);
            y += i;
        }
        expressible_as_sum_of_two_abundant_nos = false;
    }
    /*println!("{:?}", abundant.iter().enumerate()
        .filter(|&(i, &a)| a).map(|(i,&a)| i)
        .collect::<Vec<_>>());
    println!("{:?}", x);*/
    println!("{:?}", y);
}
```

Notes: The `mut` marks mutable variables. `vec!` creates vectors (growable but array-like). The `..` syntax expresses ranges (lower bound inclusive, upper bound exclusive). I have left some comments while debugging for `x`.

#### Problem 40

> An irrational decimal fraction is created by concatenating the positive integers:
> 
> 0.123456789101112131415161718192021...
> 
> It can be seen that the 12th digit of the fractional part is 1.
> 
> If d<sub>n</sub> represents the nth digit of the fractional part, find the value of the following expression.
> 
> d<sub>1</sub> x d<sub>10</sub> x d<sub>100</sub> x d<sub>1000</sub> x d<sub>10000</sub> x d<sub>100000</sub> x d<sub>1000000</sub> 

Solution to problem 40 (0.26s):

```rust
fn main () {
    let max: usize = 1_000_000; // 1 million
    let mut v: Vec<i64> = vec![0; max];
    let mut ind: usize = 0;
    for i in 1 .. max {
        for j in (0 .. 7).rev() {
            if (i as i64)/((10 as i64).pow(j)) != 0 {
                v[ind] = ((i as i64) / (10 as i64).pow(j)) % 10;
                ind = ind + 1;
                if ind == max {break;}
            }
        }
        if ind == max {break;}
    }
    println!("{:?} {:?} {:?} {:?} {:?} {:?} {:?}", v[0], v[10-1], v[100-1], v[1000-1], v[10000-1], v[100000-1], v[1000000-1]);
}
```

Notes: `usize`, `i64` are types. `_` can be used as a separator for clarity. `rev` reverses an iterator. `pow` computes integer powers. All the digits are printed separately.

#### Problem 549

> The smallest number m such that 10 divides m! is m=5.
> The smallest number m such that 25 divides m! is m=10.
> 
> Let s(n) be the smallest number m such that n divides m!.
>
> So s(10)=5 and s(25)=10.
>
> Let S(n) be ∑s(i) for 2 ≤ i ≤ n.
>
> S(100)=2012.
> 
> Find S(10<sup>8</sup>).

Solution to problem 549 (39s):

```rust
#![feature(iter_arith)]

// Checks if p_j divides factorial(base)/div
fn rem0(base: usize, div: usize, p_j: usize) -> bool {
    let mut count = 1;
    let mut val = div;
    while val % p_j == 0 {
        count += 1;
        val /= p_j;
    }
    let mut count2 = 0;
    for z in 1 .. base/p_j + 1 {
        if count2 >= count {
            return true;
        }
        count2 += 1;
        val = z;
        while val % p_j == 0 {
            count2 += 1;
            val /= p_j;
        }
    }
    if count2 >= count {
        true
    }
    else {
        false
    }
}

fn main() {
    const LIM: usize = 100_000_000;
    let mut stuff: Vec<Vec<usize>> = vec![vec![LIM,0]; LIM + 1];
    // lowest prime divisor and value of s(i)
    let mut val = 0;
    for i in 2 .. LIM + 1 {
        if stuff[i][0] == LIM { // number must be prime
            stuff[i][0] = i;
            stuff[i][1] = i;
            for j in 1 .. LIM/i + 1 {
                if stuff[i*j][0] > i {
                    stuff[i*j][0] = i;
                }
            }
        }
        else {
            val = i/stuff[i][0];
            if rem0(stuff[val][1], val, stuff[i][0]) {
                stuff[i][1] = stuff[val][1];
            }
            else {
                stuff[i][1] = (stuff[val][1]/stuff[i][0] + 1) * stuff[i][0];
            }
        }
    }
    let sum: usize = stuff.iter().map(|v| v[1]).sum();
    println!("{:?}", sum);
}
```

Notes: Although fairly simple, this is probably the biggest solution I've written.

### Epilogue

Some time after solving the ten problems mentioned above, Alice wrote the following letter to Bob:

> Bob,
> 
> You were right about all the programming languages being fun in their own way.
> 
> * Haskell: This one is probably my favourite as the notation is very similar to math and it expresses intent very clearly, especially since you don't have mutation. Once I figured out the structure of the program, writing Haskell code was very easy. However, I did have some trouble writing the pseudocode itself (example: the Fibonacci list construction).
> * Python: The code reads very much like pseudocode, which is perhaps its biggest strength. Definitely the easiest to use for string manipulation and simplicity overall.
> * Rust: For my purposes, Rust seems a bit too verbose. Despite that, the code is easy to read. Also all the code that I wrote ran very fast without much hassle.
> 
> Bye,
> 
> A

Bob was pleasantly surprised. While he had hoped that Alice would try all the three, he certainly hadn't expected that she would do so. Here's what he wrote back:

> Hi Alice,
> 
> I'm happy that you enjoyed your sojourn in the land of programming languages. There are a great many more languages out there, from the weird and esoteric to the tedious and mundane.
> 
> Even for the three languages you've tried, I think you've just touched the proverbial tip of the iceberg here, especially since you were primarily concerned with solving math problems. For a working programmer, there are many more useful features that these languages provide than those you've just seen:
> 
> * Haskell: [Purity][wiki-purity] makes debugging much easier. Best of all, there is [Hoogle][hoogle] is super helpful where you can search for functions; for example, you can do a reverse look-up for function names using only the type signature.
> * Python: You have several useful libraries to do a variety of tasks. For example, you can use [`matplotlib`][matplotlib] to make plots, [`sympy`][sympy] to do symbolic calculations and [`numpy`][numpy] for numerics. Also, there are a very large number of python questions on StackOverflow asked by beginners.
> * Rust: You've got support for automatic documentation with `///` and runnable examples in comments as well. The default installation also comes with a package manager [Cargo][crates.io] which makes installing dependencies a breeze. Safe Rust also prevents data races. The best part about the language is the community which is very friendly to beginners. Check them out on [/r/rust][reddit-rust] and [#rust][irc-rust].
> 
> Best,
> 
> Bob

As Alice stepped off her computer after reading Bob's email, she realised that she hadn't even started working on the project.

---

P.S. At the time of writing, the Rust syntax highlighting is not perfect. This is an issue with the syntax highlighter that Github Pages uses: Rouge. See the [issue][rouge-issue] and a partial fix [here][rouge-PR].
