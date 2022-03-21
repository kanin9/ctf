# Anti-Fermat
| Crypto | Warmup |
|--------|--------|

*I invented Anti-Fermat Key Generation for RSA cipher since I'm scared of the Fermat's Factorization Method.*

[Challenge download file](https://github.com/kanin9/ctf/files/8313836/challenge.zip)

From the start we are greeted with this chunk of code and output in hex format

```from Crypto.Util.number import isPrime, getStrongPrime
from gmpy import next_prime
from secret import flag

# Anti-Fermat Key Generation
p = getStrongPrime(1024)
q = next_prime(p ^ ((1<<1024)-1))
n = p * q
e = 65537

# Encryption
m = int.from_bytes(flag, 'big')
assert m < n
c = pow(m, e, n)

print('n = {}'.format(hex(n)))
print('c = {}'.format(hex(c)))
```

> n = 0x1ffc7dc6b9667b0dcd00d6ae92f...
> 
> c = 0x60160bfed79384048d0d46b807...

Now, we can deduce that q is slightly above (2<sup>1024</sup>-1) xor P. On average it is 12 lower bits that differ.
With this information we can bruteforce factors of N, if we rewrite our Q as (2<sup>1024</sup> - 1 - P + error), we can factorize N only using P
and error value which we will increment through 0 to 2<sup>13</sup>

> P((2<sup>1024</sup>-1) - P + error) = N
> 
> -P<sup>2</sup> + (error+2<sup>1024</sup>-1)P - N = 0

```from Crypto.Util.number import long_to_bytes
from gmpy2 import next_prime
from values import n, c, modinv
from math import isqrt

P = 0
e = 65537

for error in range(0, 1 << 13):
    xor = (1 << 1024) - 1
    b = xor + error
    D = b * b - 4 * n
    sqr = isqrt(D)
    if sqr * sqr == D:
        x1 = (-b + sqr) // -2
        x2 = (-b - sqr) // -2
        if x1 * (n // x1) == n:
            print("Value found ", x1)
            P = x1
        if x2 * (n // x2) == n:
            print("Value found ", x2)
            P = x2

Q = n // P
phi = (P - 1) * (Q - 1)
d = modinv(e, phi)
print(long_to_bytes(pow(c, d, n)))
```
![image](https://user-images.githubusercontent.com/101967194/159232684-9a1bfb65-7283-4843-8f83-7c130c586c15.PNG)

Ta-daaa!

`zer0pts{F3rm4t,y0ur_m3th0d_n0_l0ng3r_w0rks.y0u_4r3_f1r3d}`
