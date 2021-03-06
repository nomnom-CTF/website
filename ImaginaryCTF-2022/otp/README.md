# ImaginaryCTF 2022 - OTP Challenge Writeup

## Description

Encrypt your messages with our new OTP service. Your messages will never again be readable to anyone.

## Source Code

```python
#!/usr/bin/env python3

from Crypto.Util.number import long_to_bytes, bytes_to_long
import random
import math

def secureRand(bits, seed):
  jumbler = []
  jumbler.extend([2**n for n in range(300)])
  jumbler.extend([3**n for n in range(300)])
  jumbler.extend([4**n for n in range(300)])
  jumbler.extend([5**n for n in range(300)])
  jumbler.extend([6**n for n in range(300)])
  jumbler.extend([7**n for n in range(300)])
  jumbler.extend([8**n for n in range(300)])
  jumbler.extend([9**n for n in range(300)])
  out = ""
  state = seed % len(jumbler)
  for _ in range(bits):
    if int(str(jumbler[state])[0]) < 5:
      out += "1"
    else:
      out += "0"
    state = int("".join([str(jumbler[random.randint(0, len(jumbler)-1)])[0] for n in range(len(str(len(jumbler)))-1)]))
  return long_to_bytes(int(out, 2)).rjust(bits//8, b'\0')

def xor(var, key):
  return bytes(a ^ b for a, b in zip(var, key))

def main():
  print("Welcome to my one time pad as a service!")
  flag = open("flag.txt", "rb").read()
  seed = random.randint(0, 100000000)
  while True:
    inp = input("Enter plaintext: ").encode()
    if inp == b"FLAG":
      print("Encrypted flag:", xor(flag, secureRand(len(flag)*8, seed)).hex())
    else:
      print("Encrypted message:", xor(inp, secureRand(len(inp)*8, seed)).hex())

if __name__ == "__main__":
  main()
  ```
  
## Solution
  
In this challenge, the flag gets xor'd with the output from the `secureRand()` method. By printing out the output of the method a few times, we can see a pattern where the number of 1 bits largely dominates the number of 0 bits. This is because the statement `if int(str(jumbler[state])[0]) < 5` is true approximately 75% of the time since the first digit of all the numbers in jumbler is less than 5 approximately 75% of the time. Knowing that information, we can generate a large sample size of encrpyted flags and xor the most frequent bit at each index with 1. Finally, we get the original bits of the flag (`(flag[i]^1)^1 = flag[i]`). Then we can use an online converter to convert those recovered bits from binary to ascii, and we get the flag.

Solver Script:
```python
from Crypto.Util.number import long_to_bytes, bytes_to_long
from pwn import *
import time

r = remote('otp.chal.imaginaryctf.org', 1337)

a = []

while True:
    line = r.recvline().decode()
    r.sendline('FLAG')
    if "Enter" in line:
        r.sendline('FLAG')
    if "Encrypted" in line:
        a.append(bin(int(line.split("flag: ")[1],16))[2:])
    if len(a) > 300:
        break
        
ans = ''
for index in range(376):
    ones = 0
    zeros = 0
    for binary in a:
        if binary[index] == '1':
            ones += 1
        else:
            zeros += 1
            
    if ones > zeros:
        ans += '0'
    else:
        ans += '1'
print(ans)     
```
`flag: ictf{benfords_law_catching_tax_fraud_since_1938}`
