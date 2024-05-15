---
title: "Primes And Integer Factorisation" 
date: 2024-04-27T14:39:00.000Z 
categories: "articles" 
tags: 
  - "cryptography" 
  - "number theory"
params:
  math: true
draft: true 
---

<!-- toc -->

- [Introduction](#introduction)
  - [Sieve of Eratosthenes](#sieve-of-eratosthenes)
- [Primality Testing](#primality-testing)
  - [AKS Primality Test](#aks-primality-test)
<!-- tocstop -->

# Introduction

Integer Factorisation is an interesting problem, each integer $i\in\mathbb{Z}$
has a unique prime factorisation such that $p_1\cdot...\cdot p_n=i$. At first
this doesn't sound like something interesting at all, but this property of
integers has underpin cryptography for years until recently and is still very
much being actively researched. Numbers with no prime factors where $i=i\cdot1$
are naturally prime numbers - which are critical to many areas of cryptography,
computing and number theory in general.

People have developed methods for both finding prime numbers and factorising
integers into their prime roots for centuries. The simplest method being the
brute force approach, where one simply tries to divide the number by each number
before it and if none yield an answer its prime and can be added to the list.
The ancient Sieve of Eratosthenes is a technique of generating prime numbers,
that is much more efficient than the brute force approach and is fairly simple,
lets dive into it quickly.

## Sieve of Eratosthenes

Essentially the sieve of Eratosthenes iterates over the integers $\le n$ for
those that are prime (starting with 2) we mark all of its multiples as non-prime.
Then onto the next number and repeat. For all numbers left unmarked we know these
are prime as there are no factors before them in the interval $[2, i]$, i.e.
their only roots are $1$ and $i$.

```go
func SieveOfEratosthenes(n int) []int {
    if n < 2 {
        return nil // 1 is not prime
    }
    // Initialise a slice to store the primes we generate
    primes := make([]int, 0, n)
    // Initialise a boolean slice with all false values
    b := make([]bool, n+1)
    b[0], b[1] = true, true
    // Set all multiples of numbers up to sqrt(n) to false
    for i := 2; i*i <= n; i++ {
        if b[i] {
            continue
        }
        for j := i * i; j <= n; j += i {
            b[j] = true
        }
    }
    // Primes are remaining true values
    for i, v := range b {
        if !v {
            primes = append(primes, i)
        }
    }
    return primes
}
```

Since this ancient technique was developed numerous improvements have been made
in the generation of prime numbers such as the Sieve of Atkin which is faster but
a lot more complex and various Wheel Sieves. However, there is a problem with
these algorithms, in one way or another they iterate up to $n$ producing a list
of primes $\le n$. What is we wanted to find a really large prime number? Then
we would need to take a different approach. We would need 2 tools - an
efficient and accurate primality test and a way to generate large prime numbers
without going through every number before them.

# Primality Testing

Primality testing is simply the act of discovering whether an integer is prime
or not. There are a number of algorithms to do this, one of the most famous being
the Miller-Rabin primality test, and the new AKS primality test.

The Miller-Rabin primality test is 100% accurate for integers $\le2^{64}-1$ or
simply all `uint64` integers and up to 3.3 septillion with some variations. But
can be made almost 100% accurate for arbitrarily large numbers with additional
rounds - we will dive into what this means shortly. This means it is a
probabilistic test - as it is an approximation it is extremely fast (hence its
wide usage), but has the possibility of false

The Miller-Rabin primality test for an integer $n$, effectively works as follows:

 - For numbers $\ge2$ choose a number of rounds $k$
 - Find numbers $s$ and $d$ such that $n-1=2^s\cdot d,~s,d>0~\text{and }d\mod2=1$
 - For each round $r=1,~r\le k$ repeat the following
   - Choose a random base $a$ such that $2 < a < n-2$
   - Compute $x=a^d\mod n$
   - Then repeat the following $s$ times:
     - Compute $y=x^2\mod n$
     - If $y=1\text{ and }x\ne1\text{ and }x\ne n-1$ then the number isn't prime
     - Set $x=y$
  - If $y\ne1$ then the number isn't prime
  - Otherwise the number is most likely prime

For an increased accuracy in the Miller-Rabin primality test the higher the
chosen value $k$, the number of rounds, the more accurate the final result on
the primality of $n$. For smaller $n$ sieves or other methods can be used to
improve the speed of the test. An implementation using both the Sieve of
Eratosthenes and the Miller-Rabin Primality test in conjunction follows:

```go
var zero = big.NewInt(0)
var one = big.NewInt(1)
var two = big.NewInt(2)

// Return new slice containing only the elements in a that are not in b.
//     d = {x : x ∈ a and x ∉ b}
func difference(a, b []int) []int {
    diff := make([]int, 0, len(a))
    for _, v := range a {
        if found := slices.Contains(b, v); !found {
            diff = append(diff, v)
        }
    }
    return diff
}

// Generate a bitmask for primes from lo->hi
func primemask(lo, hi int) *big.Int {
    low := make([]int, lo)
    high := make([]int, hi)
    // Get primes 2->hi
    high = SieveOfEratosthenes(hi)
    // Only get lower limit primes if lo > 2
    if lo > 2 {
        low = SieveOfEratosthenes(lo)
    }
    // Use only difference between high and low slices
    diff := difference(high, low)
    // Create the mask by performing mask |= 1<<prime
    mask := new(big.Int)
    one := big.NewInt(1)
    for i := 0; i < len(diff); i++ {
        prime := new(big.Int)
        // Left shift 
        prime = prime.Lsh(one, uint(diff[i]))
        mask = mask.Or(mask, prime)
    }
    return mask
}

func bigPowMod(b, e, m *big.Int) *big.Int {
    // Special case
    if m.Cmp(one) == 0 {
        return zero
    }
    // Copy arguments
    base := new(big.Int)
    base = base.Set(b)
    exp := new(big.Int)
    exp = exp.Set(e)
    mod := new(big.Int)
    mod = mod.Set(m)
    // Initialise remainder
    r := big.NewInt(int64(1))
    // Use the binary exponentation of e to successively calculate (b*b) (mod m)
    base = base.Mod(base, mod)
    for exp.Cmp(zero) > 0 {
        em := new(big.Int)
        em = em.Mod(exp, two)
        if em.Cmp(one) == 0 {
            r = r.Mul(r, base).Mod(r, mod)
        }
        // Divide exponent by 2
        exp = exp.Rsh(exp, uint(1))
        // Repeatedly square b to achieve b=b^(2^i) (mod m)
        base = base.Mul(base, base).Mod(base, mod)
    }
    return r
}

func MillerRabin(n *big.Int, rounds int, force2 bool) bool {
    // n must and odd integer > 2
    one := big.NewInt(1)
    even := new(big.Int)
    even = even.And(n, one)
    if n.Cmp(two) <= 0 || even.Cmp(zero) == 0 {
        return false
    }
    // Get prime bitmask for primes between 2-2^16
    if n.Cmp(big.NewInt(2 << 16)) <= 0 {
        primeBitmask := primemask(2, 2 << 16)
        // Check that primeBitmask&(1<<n) != 0 -> n == prime
        shift := big.NewInt(1)
        shift = shift.Lsh(shift, uint(n.Uint64()))
        prime := new(big.Int)
        prime = prime.And(primeBitmask, shift)
        return prime.Cmp(zero) != 0
    }
    nm1 := big.NewInt(-1)
    nm1 = nm1.Add(n, nm1)
    nm4 := big.NewInt(-4)
    nm4 = nm4.Add(n, nm4)
    // s > 0 and d > 0 such that n-1 = 2^s(d)
    s := nm1.TrailingZeroBits() // get highest power of 2 from n-1
    d := new(big.Int)
    d = d.Rsh(nm1, s) // get odd multiplier such that n-1/2^s=d
    // Get random source
    src := rand.NewSource(time.Now().UTC().UnixNano())
    rand := rand.New(src)
    y := new(big.Int)
    for i := 0; i < rounds; i++ {
        // n is always a probable prime to base 1 and n-1
        a := new(big.Int)
        if i == reps-1 && force2 {
            a = two // Force using base 2 once
        } else {
            a = a.Rand(rand, nm4).Add(a, big.NewInt(int64(2))) // random(2, n-2)
        }
        x := bigPowMod(a, d, n)
        for j := uint(0); j < s; j++ {
            y = bigPowMod(x, two, n)
            if y.Cmp(one) == 0 && x.Cmp(one) != 0 && x.Cmp(nm1) != 0 {
                return false // nontrivial square root of 1 (mod n)
            }
            x = x.Set(y)
        }
        if y.Cmp(one) != 0 {
            return false
        }
    }
    return true
}
```

By using $25$ rounds we can achieve almost 100% accuracy and can probabilistically
guarantee the primality of $n$ for arbitrarily large numbers.

## AKS Primality Test

Developed in 2002 a new primality test was discovered as an alternative to the
Miller-Rabin primality test which is deterministic - not probabilistic.

It works as follows:

  - First check $n\ge2$
  - If $n$ is a perfect power: $n=a^b,~a,b>1$ then $n$ is not prime
  - Find $r$ such that $\text{ord}_r(n)>(\log_2 n)^2$
    - If $\text{gcd}(r,~n)\ne1$ (they are not coprime) then $n$ is not prime
  - For all $2\le n\le\text{min}(r,~n-1)$
    - If $a~|~n$ then for some $2\le a\le\text{min}(r,~n-1)$ then $n$ is not prime
  - If $n\le r$ then $n$ is prime
  - For $a=1$ to $\lfloor{\sqrt{\varphi(r)}\log_2n}\rfloor$:
    - If $(X+a)^n\ne X^n+a\pmod{X^r-1,~n}$ then $n$ is not prime
  - Otherwise $n$ is prime

<!-- TODO: Implement AKS test -->

# Integer Factorisation

Now we have a few methods to test whether a number is prime lets look at numbers
that are not prime, composite numbers. As previously stated all integers have a
unique prime factorisation and as such finding these prime roots are an
interesting field of study. If you can factorise a composite number into its
prime roots you can break RSA encryption and much more.

{{< subscribe-button >}}
