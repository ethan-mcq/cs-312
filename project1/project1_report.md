# [20 points] Explain the time and space complexity of your RSA algorithm by showing and summing up the complexity of each subsection of your code. This must appear as a subsection of your report (that is, it is great to have this info in your code comments, but you should also pull it out and include it in the report proper). For this project all time and space complexities should be in terms of bits. In particular you need to explain your time and space complexity for:

### Modular Exponentiation
```
def mod_exp(x: int, y: int, N: int) -> int:
    if y == 0:
        return 1
    z = mod_exp(x, y // 2, N)
    if y % 2 == 0:
        return (z * z) % N
    else:
        return (x * z * z) % N
```
This algorithm will stop after, at most, n recursive calls. Additionally, it will multiply n-bit numbers. Each call does an O(n^2) operation, and there are O(n) calls due to halving the exponent. O(n^3) is the total time complexity for this algorithm. 

### Fermat for testing if a number is prime
```
def fermat(N: int, k: int) -> str:
    for i in range(k):
        a = random.randint(2, N - 2)
        if mod_exp(a, N - 1, N) != 1:
            return "composite"
    return "prime"
```
My Fermat function runs the mod_exp function k times to check the primality of N. Considering the most computationally expensive part which is the repeated calls to mod_exp, is O(k*n^3) where k is the number of calls (note for i in range(k))

### Miller-Rabin for testing if a number is prime
```
def miller_rabin(N: int, k: int) -> str:
    r, d = 0, N - 1
    while d % 2 == 0: # d is my constant operator
        d //= 2
        r += 1
    for i in range(k):
        a = random.randint(2, N - 2)
        x = mod_exp(a, d, N)
        if x == 1 or x == N - 1:
            continue
        for i in range(r - 1):
            x = mod_exp(x, 2, N)
            if x == N - 1:
                break
        else:
            return "composite"
    return "prime"
```
My Miller-Rabin algorithm hasnt really been optimized, but the main point of this algorithm is to ensure less false-positive calls than the Fermat function. The time complexity of this algorithm is O(k * n^4), again where k is the number of calls on the algorithm. The addition of O(n) comes from the time complexity of the inital calculation of r and d. 

### Extended Euclid for finding a multiplicative inverse
```
def ext_euclid(a: int, b: int) -> tuple[int, int, int]:
    if b == 0:
        return (1, 0, a)
    x1, y1, gcd = ext_euclid(b, a % b)
    return (y1, x1 - (a // b) * y1, gcd)
```
This one can become computationally intensive as you increase in n. The are n-calls in this algorithm and each has an O(n) operation complexity, thus this algorithm has an O(n^2) complexity.

### Generating a random prime number
```
def generate_large_prime(bits=512) -> int:
    while True:
        p = random.getrandbits(bits)
        p |= (1 << (bits - 1)) | 1
        if miller_rabin(p, 10) == "prime":
            return p
```
The largest cost to this is testing primality of the number. getrandbits() is an O(n) complexity, plus the miller-rabin, we would expect this time complexity to be O(k * n^5).

### The full RSA algorithm to generate key pairs (the used combination of all of the above)
```
def generate_key_pairs(bits: int) -> tuple[int, int, int]:
    p = generate_large_prime(bits // 2)
    q = generate_large_prime(bits // 2)
    N = p * q
    phi = (p - 1) * (q - 1)
    for e in primes:
        x, y, gcd = ext_euclid(e, phi)
        if gcd == 1:  # e is relatively prime to phi
            d = x % phi
            if d < 0:
                d += phi
            return (N, e, d)
```
This should not pass in complexity the computation of the generate_large_prime function: O(k * n^5) as all other computations in this algorithm are less complex. p and 1 are calculated independently, each O(k * n^5), calculating N is O(n) complexity, and eht ext_euclid() algorithm is O(n^2) complexity as dicussed before. If we were to sum the total time complexity for each operation in this algorithm it would be; O(k * n^5) + O(k * n^5) + O(n) + O(n^2).

# [10 points] Discuss the equations you used to compute the probability of correctness for both your Fermat and Miller-Rabin implementations.

In our calculations of correctness, we need not calculate the chances of getting a composite, as if a number gets a composite return statement, we know with 100% certainty in these algorithms that they were indeed composite. However, if they are unable to fail the composte testing, they then are assumed prime, which is where we lack in accuracy. The problem that arrises with Fermats algorithm lies in Carmichael numbers, that are pseudoprimes to every base k that is relatively prime to N. However, for a random base k, the likelihood that a composite N is a pseudoprime to base k is about 50%​. Thus we calculate the liklihood of false positives for Fermat as follows:
```
def fprobability(k: int) -> float:
    return 1 - 1 / (2 ** k)
```
For Miller-Rabin, it assumes that if k bases are used, the maximum probability that it is a false-positive is 25% because of the additional steps implemented in catching Carmichaels numbers. 
```
return 1 - 1 / (4 ** k)
```
