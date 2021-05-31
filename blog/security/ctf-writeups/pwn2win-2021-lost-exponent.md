# Pwn2Win 2021 - Lost Exponent

## Challenge Description

> #### Lost Exponent - ppc, misc - 253 pts
>
> While Laura was looking for her brother, she found a program that seems to scramble a password and save the result. Could you help her find the original password so that she can find and save her brother before it's too late?
>
> **Author**: [andre\_smaira](https://github.com/afsmaira)
>
> **Attachments:** [Files](https://static.pwn2win.party/lost_exponent_2184dc532e4aec92252b9ae6a53058930e750abcbe6b33b70a9ac2fafae6449d.tar.gz) \| [Mirror](https://drive.google.com/file/d/1JUmZxfB5InabwfWTkRpBUyhxsFUH1JCi/view?usp=drivesdk)

## Writeup

### First Look

The attachment contain 2 files: a python script `encode.py` to encode \(or encrypt, if I may\) the flag, and the 17MB encrypted flag `enc`. In this script, it imports 2 variables: `e` and `flag` which we didn't have access to.

```python
from math import sqrt
from random import seed, shuffle
from lost import e, flag
from itertools import product
from numpy import sign, diff


assert flag.startswith('CTF-BR{')
seed(6174)
n = int(sqrt(len(flag))) + 2
order = list(product(range(n), repeat=2))
shuffle(order)
order.sort(key=(lambda x: sign(diff(x))))


class Matrix:
    def __init__(self):
        self.n = n
        self.m = [[0]*n for _ in range(n)]

    def __iter__(self):
        for i in range(self.n):
            for j in range(self.n):
                yield self.m[i][j]

    def I(self):
        r = Matrix()
        for i in range(n):
            r[i, i] = 1
        return r

    def __setitem__(self, key, value):
        self.m[key[0]][key[1]] = value

    def __getitem__(self, key):
        return self.m[key[0]][key[1]]

    def __mul__(self, other):
        r = Matrix()
        for i in range(n):
            for j in range(n):
                r[i, j] = sum(self[i, k]*other[k, j] for k in range(n))
        return r

    def __pow__(self, power):
        r = self.I()
        for _ in range(power):
            r = r * self
        return r

    def __str__(self):
        return str(self.m)


if __name__ == '__main__':
    m = Matrix()
    for i, f in zip(order, flag):
        m[i] = ord(f)
    cflag = list(map(str, m ** e))
    mn = max(map(len, cflag))
    mn += mn % 2
    cflag = ''.join(b.zfill(mn) for b in cflag)
    cflag = bytes([int(cflag[i:i+2]) for i in range(0, len(cflag), 2)])

    with open('enc', 'wb') as out:
        out.write(cflag)
```

The code is fairly simple. It randomly scramble the flag and represent it into a matrix, then it will do matrix exponentiation of this matrix with `e`, flatten the resulting matrix and write it into a file.

However, since the seed \(6174\) is given, the scramble is not purely random and we can determine the order by following the script. Unfortunately, the order is derived from the flag's length \(which we didn't know at this point\). So we need to find the length before we can continue.

```python
seed(6174)
n = int(sqrt(len(flag))) + 2
order = list(product(range(n), repeat=2))
shuffle(order)
order.sort(key=(lambda x: sign(diff(x))))
```

### Finding the Flag's Length

There are several ways to find the length, one way is to guess from the resulting file. From the script: after the matrix undergoes exponentiations, it will stringify all the numbers inside the matrix and `zfill` \(prepend zeroes\) them so all numbers have same length.

```python
m = Matrix()
for i, f in zip(order, flag):
    m[i] = ord(f)
cflag = list(map(str, m ** e))
mn = max(map(len, cflag))
mn += mn % 2
cflag = ''.join(b.zfill(mn) for b in cflag)
cflag = bytes([int(cflag[i:i+2]) for i in range(0, len(cflag), 2)])
```

From this characteristic, we can guess that most of the resulting numbers should have zeroes prefix \(only few will have non-zeroes prefix\). The resulting line of numbers should be like this:

```text
000000...1234
000000...1337 
000023...3456
000000...0000    // some line may contain only zeros
000000...1321 
123131...4131    // the max length 
002512...1231
000000...0000
...              // and so on
```

By analyzing the location of long zeroes inside `enc`,  we may be able to guess `mn` \(the length of every number in the matrix\).

First, let's change `enc` representation from bytes into string:

```python
def change_into_string():
    bytes = []
    with open('enc', 'rb') as enc:
        bytes = enc.read()
    digits = [str(b).zfill(2) for b in bytes]
    digits = ''.join(digits)
    with open('enc.string', 'w') as f:
        f.write(digits)
```

Running the above function will result to a file with double the size of `enc`. Which is expected, since we change each byte representation into 2 chars \(1 char is 1 byte\).

```text
❯ ls -l
-rw-r--r--   1 adam  staff  35091252 May 29 22:14 enc.string
-rw-r--r--@  1 adam  staff  17545626 May 29 22:13 enc
-rw-r--r--@  1 adam  staff      2548 May 29 20:55 encode.py
```

We can also analyze possibilities of `mn` by viewing the matrix characteristic:

* Remember that all numbers in the matrix have length `mn`. That means the size of `enc` \(`size`\) should be divisible by `mn`. Therefore, `mn` should be a factor of `size`.
* The matrix is square, so the number of its element should be a square number. That means `size / mn` should be a square number.
* Assume flag's length should be within `6 <= len(flag) <= 100`. 

From these characteristics, we can have potential candidates of `mn`:

```python
def get_mn(size):
    factors = sorted(list(get_factors(size)))
    candidates_mn = []
    for f in factors:
        line = size / f
        if math.sqrt(line).is_integer():
            candidates_mn.append(f)

    for mn in candidates_mn:
        print('---')
        print('mn:', mn)
        print('lines:', f / mn)
        print('n:', math.sqrt(f / mn))
        print('flag:', (math.sqrt(f / mn) - 2) ** 2)
        
get_mn(35091252)

'''
output:
---
mn: 716148
lines: 49.0
n: 7.0
flag: 25.0
---
mn: 974757
lines: 36.0
n: 6.0
flag: 16.0
---
'''
```

At this point, there are 2 possibilities, whether the flag length is 25 or 16, both sound equally probable. So next, we can analyze the long-zeroes location inside `enc`. In order to do this, we can use this script:

```python
digits = open('enc.string', 'r').read()

start_i = None
i = 0
while i < len(digits):
    if digits[i:i+16] == '0' * 16:
        if start_i is None:
            start_i = i
    if digits[i] != '0':
        if start_i is not None:
            print(start_i, i)  # print long-zeroes location 
        start_i = None
    i += 1

'''
output:
0 141963
716148 5154999
5729183 10168035
10742217 15163599
15755255 15879749
16471404 16595897
17187552 17312045
17903700 20052146
23632884 25065181
28645913 28686627
29362068 30078217
33658956 33699663
34375104 34514009
'''
```

Basically, by manual analysis, we decide that it is more probable that the flag length is 25, since the long-zeroes location if the flag length is 16 make the challenge unsolvable.

To help us analyze further, let's split the `enc.string` to multiple files.

```python
mn = 716148

digits = open('enc.string', 'r').read()

for i in range(49):
    bytes = digits[mn*mn*(i+1)]
    open(f'output/{i}.txt','w').write(bytes)
```

### Constructing the matrix order

Now that we know the flag length, we can determine the `order`. Next, let's visualize the matrix. We know that the first 7 chars of flag is `CTF-BR{` and the 25th char is `}`

```python
from math import sqrt
from random import seed, shuffle
from itertools import product
from numpy import sign, diff

flag = 'CTF-BR{??????????????????}'

seed(6174)
n = int(sqrt(len(flag))) + 2
order = list(product(range(n), repeat=2))
shuffle(order)
order.sort(key=(lambda x: sign(diff(x))))

m = [['.']*n for _ in range(n)]
for i, f in zip(order, flag):
    m[i[0]][i[1]] = f

for row in m:
    print(' '.join(row))

'''
output:

? . . . . . .
? . . . . . .
- B . . . . .
? { ? ? . . .
T ? ? ? } . .
R ? ? C ? ? .
F ? ? ? ? ? ?

'''
```

### Finding the exponent "e"

Looking at the matrix, there is a notable finding there: the location of `}`  which is \(5, 5\). If we do exponentiation to the matrix by `e`. The values of index \(5, 5\) is `ord('}') ** e`. With this, we can quickly brute force the value of `e`. Note that the last 32 digits of \(5, 5\) in `enc` is _92880428233183920383453369140625_.

```python
e = 1
while e := e + 1:
    if pow(ord('}'), e, 10 ** 32) == 92880428233183920383453369140625:
        print('FOUND:', e)
        break

'''
❯ python3 find_e.py
FOUND: 341524
'''
```

### Finding the Flag

Now that we find the `e`. We can slowly craft the flag from our matrix. The idea is to find the `?` value from matrix diagonal. After that, for each line, slowly move left to get find `?` value. There may be an automated way to get this flag, but I do the calculation manually using the program below:

```python
import string

e = 341524
# flag = 'CTF-BR{abcdefghijklmnopqr}'
__flag = 'CTF-BR{s0M3_0F_m47r1X_106}'
MOD = 10 ** 32

A = ord('s')
B = ord('0')
C = ord('M')
D = ord('3')
E = ord('_')
F = ord('0')
G = ord('F')
H = ord('_')
I = ord('m')


L = ord('r')
M = ord('1')
N = ord('X')
O = ord('_')
P = ord('1')
Q = ord('0')
R = ord('6')


def q():
    target = int(open('numbers/0.txt', 'r').read()[-32:])
    for c in string.printable:
        v = pow(ord(c), e, 10 ** 32)
        if int(str(v)[:32]) == target:
            print('FOUND:', c)
# q()

def a():
    target = int(open('numbers/7.txt', 'r').read()[-32:])
    for c in string.printable:
        v = (pow(Q, e-1, 10 ** 32) * ord(c)) % MOD
        if int(str(v)[:32]) == target:
            print('FOUND:', c)
# a()

def r():
    target = int(open('numbers/24.txt', 'r').read()[-32:])
    for c in string.printable:
        v = pow(ord(c), e, 10 ** 32)
        if int(str(v)[:32]) == target:
            print('FOUND:', c)
# r()

def o():
    target = int(open('numbers/40.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v = pow(ord(c), e, 10 ** 32)
        if int(str(v)[:32]) == target:
            print('FOUND:', c)

# o()

def p():
    target = int(open('numbers/48.txt', 'r').read()[-32:])
    print(target)
    for i in range(255):
        v = pow(i, e, 10 ** 32)
        if int(str(v)[:32]) == target:
            print('FOUND:', chr(i))
# p()

def m():
    target = int(open('numbers/23.txt', 'r').read()[-32:])
    print(target)
    for i in range(255):
        v = (pow(R, e-1, 10 ** 32) * i) % MOD
        if int(str(v)[:32]) == target:
            print('FOUND:', chr(i))
# m()

def d():
    target = int(open('numbers/31.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        d = ord(c)
        x = ord('}')
        for _ in range(1, e):
            d = ((d * R) + (x * ord(c))) % MOD
            x = (x * ord('}')) % MOD
        print('TRY:', c, d)
        if d == target:
            print('FOUND:', c)
# d()

def f():
    target = int(open('numbers/39.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v = ord(c)
        x = O
        for _ in range(1, e):
            v = ((v * ord('}')) + (x * ord(c))) % MOD
            x = (x * O) % MOD
        print('TRY:', c, v)
        if v == target:
            print('FOUND:', c)
# f()

def i():
    target = int(open('numbers/21.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = ord(c)
        v2 = ord('{')
        v3 = M
        v4 = R
        for _ in range(1, e):
            v1 = sum([
                v1 * Q,
                v2 * A,
                v3 * ord('-'),
                v4 * ord(c)
            ]) % MOD
            v2 = sum([
                v3 * ord('B'),
                v4 * ord('{')
            ]) % MOD
            v3 = sum([
                v4 * M,
            ]) % MOD
            v4 = sum([
                v4 * R
            ]) % MOD
        print('TRY:', c, v1)
        if v1 == target:
            print('FOUND:', c)
# i()

def g():
    target = int(open('numbers/30.txt', 'r').read()[-32:])
    for c in string.printable:
        v1 = ord(c)
        v2 = D
        v3 = ord('}')
        for _ in range(1, e):
            v1 = sum([
                v2 * M,
                v3 * ord(c),
            ]) % MOD
            v2 = sum([
                v2 * R,
                v3 * D,
            ]) % MOD
            v3 = (v3 * ord('}')) % MOD

        print('TRY:', c, v1)
        if v1 == target:
            print('FOUND:', c)
# g()

def c():
    target = int(open('numbers/29.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = ord(c)
        v2 = G
        v3 = D
        v4 = ord('}')
        for _ in range(1, e):
            v1 = sum([
                v2 * ord('B'),
                v3 * ord('{'),
                v4 * ord(c)
            ]) % MOD
            v2 = sum([
                v3 * M,
                v4 * G,
            ]) % MOD
            v3 = sum([
                v3 * R,
                v4 * D,
            ]) % MOD
            v4 = (v4 * ord('}')) % MOD

        print('TRY:', c, v1)
        if v1 == target:
            print('FOUND:', c)
# c()

def h():
    target = int(open('numbers/37.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = O
        v2 = F
        v3 = ord('C')
        v4 = ord(c)
        for _ in range(1, e):
            v4 = sum([
                v3 * M,
                v2 * G,
                v1 * ord(c)
            ]) % MOD
            v3 = sum([
                v3 * R,
                v2 * D,
                v1 * ord('C')
            ]) % MOD
            v2 = sum([
                v2 * ord('}'),
                v1 * F
            ]) % MOD
            v1 = (v1 * O) % MOD

        print('TRY:', c, v4)
        if v4 == target:
            print('FOUND:', c)
# h()

def find_l():
    target = int(open('numbers/47.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = P
        v2 = ord(c)
        for _ in range(e-1):
            v2 = sum([
                    v2 * O,
                    v1 * ord(c)
                ]) % MOD
            v1 = (v1 * P) % MOD
        print('TRY:', c, v2)
        if v2 == target:
            print('FOUND:', c)
# find_l()

def find_e():
    target = int(open('numbers/46.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = P
        v2 = L
        v3 = ord(c)
        for _ in range(e-1):
            v3 = sum([
                v3 * ord('}'),
                v2 * F,
                v1 * ord(c)
            ]) % MOD
            v2 = sum([
                    v2 * O,
                    v1 * L
                ]) % MOD
            v1 = (v1 * P) % MOD
        print('TRY:', c, v3)
        if v3 == target:
            print('FOUND:', c)

# find_e()



def find_b():
    target = int(open('numbers/45.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = P
        v2 = L
        v3 = E
        v4 = ord(c)
        for _ in range(e-1):
            v4 = sum([
                v4 * R,
                v3 * D,
                v2 * ord('C'),
                v1 * ord(c)
            ]) % MOD
            v3 = sum([
                v3 * ord('}'),
                v2 * F,
                v1 * E
            ]) % MOD
            v2 = sum([
                    v2 * O,
                    v1 * L
                ]) % MOD
            v1 = (v1 * P) % MOD
        print('TRY:', c, v4)
        if v4 == target:
            print('FOUND:', c)

# find_b()


# index 44: b*m e*g l*h p*n
def find_n():
    target = int(open('numbers/44.txt', 'r').read()[-32:])
    print(target)
    for c in string.printable:
        v1 = P
        v2 = L
        v3 = E
        v4 = B
        v5 = ord(c)
        for _ in range(e-1):
            v5 = sum([
                v4 * M,
                v3 * G,
                v2 * H,
                v1 * ord(c)
            ]) % MOD
            v4 = sum([
                v4 * R,
                v3 * D,
                v2 * ord('C'),
                v1 * B
            ]) % MOD
            v3 = sum([
                v3 * ord('}'),
                v2 * F,
                v1 * E
            ]) % MOD
            v2 = sum([
                    v2 * O,
                    v1 * L
                ]) % MOD
            v1 = (v1 * P) % MOD
        print('TRY:', c, v5)
        if v5 == target:
            print('FOUND:', c)

# find_n()


# index 43: n*B b*{ e*c l*k p*j
def find_c_k():
    target = int(open('numbers/43.txt', 'r').read()[-32:])
    print(target)
    for k in 'tT7':
        for c in 'aA4':
            v1 = P
            v2 = L
            v3 = E
            v4 = B
            v5 = N
            v6 = ord(c)
            for _ in range(e-1):
                v6 = sum([
                    v5 * ord('B'),
                    v4 * ord('{'),
                    v3 * C,
                    v2 * ord(k),
                    v1 * ord(c)
                ]) % MOD
                v5 = sum([
                    v4 * M,
                    v3 * G,
                    v2 * H,
                    v1 * N
                ]) % MOD
                v4 = sum([
                    v4 * R,
                    v3 * D,
                    v2 * ord('C'),
                    v1 * B
                ]) % MOD
                v3 = sum([
                    v3 * ord('}'),
                    v2 * F,
                    v1 * E
                ]) % MOD
                v2 = sum([
                        v2 * O,
                        v1 * L
                    ]) % MOD
                v1 = (v1 * P) % MOD
            print('TRY:', c, v6)
            if v6 == target:
                print('FOUND:', c, k)

# find_c_k()
```

## Flag

`CTF-BR{s0M3_0F_m47r1X_106}`

