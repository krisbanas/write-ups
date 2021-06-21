## Introduction
The _opisthocomus-hoazin_ was a Crypto challenge presented at the [HSCTF](https://ctf.hsctf.com/). The challenge contained a Python script and an output file. Let's start at looking at the contents of the files.

```python
# In script.py:
1  import time
2  from Crypto.Util.number import *
3  flag = open('flag.txt','r').read()
4  p = getPrime(1024)
5  q = getPrime(1024)
6  e = 2**16+1
7  n=p*q
8  ct=[]
9  for ch in flag:
10     ct.append((ord(ch)^e)%n)
11 print(n)
12 print(e)
13 print(ct)
```

```text
# In output.txt:
15888457769674642859708800597310299725338251830976423740469342107745469667544014118426981955901595652146093596535042454720088489883832573612094938281276141337632202496209218136026441342435018861975571842724577501821204305185018320446993699281538507826943542962060000957702417455609633977888711896513101590291125131953317446916178315755142103529251195112400643488422928729091341969985567240235775120515891920824933965514217511971572242643456664322913133669621953247121022723513660621629349743664178128863766441389213302642916070154272811871674136669061719947615578346412919910075334517952880722801011983182804339339643
65537
[65639, 65645, 65632, 65638, 65658, 65653, 65609, 65584, 65650, 65630, 65640, 65634, 65586, 65630, 65634, 65651, 65586, 65589, 65644, 65630, 65640, 65588, 65630, 65618, 65646, 65630, 65607, 65651, 65646, 65627, 65586, 65647, 65630, 65640, 65571, 65612, 65630, 65649, 65651, 65586, 65653, 65621, 65656, 65630, 65618, 65652, 65651, 65636, 65630, 65640, 65621, 65574, 65650, 65630, 65589, 65634, 65653, 65652, 65632, 65584, 65645, 65656, 65630, 65635, 65586, 65647, 65605, 65640, 65647, 65606, 65630, 65644, 65624, 65630, 65588, 65649, 65585, 65614, 65647, 65660]
```

## Simplifications and Assumptions
The first step is figuring out what does the code do with the flag. The flag is read from a file in `line 3`, then `line 9` and `line 10` iterate through each character in the flag and transform it. The result of the transformation is stored in the array `ct`. The transformation is a bitwise XOR with unicode code of the character and value of variable `e` as operands. A `modulo n` is applied on the result.

Let us simplify the code. The value of `e` is contant and printed on the second line of the `output.txt`. Hence `e=65537`. 

We can also assume the range of `ord(ch)`. We could assume it having 32-bits however, we expect the flag to contain only ASCII characters. That leaves us with a 7-bit values (up to 127).

Now, the result of a bitwise XOR will never yield a higher value than both of the operands. Since both `ord(ch)` and `e` are much smaller than `n`, the result of `ord(ch)^e` is also much smaller than `n`. We can safely assume that the modulo operation will never change the resulting value and ignore the `n`.

Since `n` is ignored, we can remove its derivation (lines 4,5 and 7). I will also skip the imports, prints, and simplify the `e`. The result is:

```python
# In script.py:
flag = open('flag.txt','r').read()  # get flag as plaintext (ASCII)
e = 65537
ct=[]
for ch in flag:                  # transform each element of flag
    ct.append((ord(ch)^e))       # take a character and perform a bitwise XOR
```

We also have the values stored in array `ct`.

## On Bitwise Operations

Let us take a look at the bitwise XOR. In short, the result of XOR is `1` only if the compared values are different. For the lowercase `a`, the operation will look like this (I add padding in front of the letter to match the length):

```
  10000000000000001 # value of e in binary
^ 00000000001100001 # letter a in binary
= 10000000001100000 # the result
```

Let's mark the range of ASCII:

![image](https://user-images.githubusercontent.com/26695258/122799098-9f6a5880-d2c1-11eb-8f00-0d883b203e32.png)

Therefore, in practice, the XOR with `e` does two things to the original value:
1. Adds binary 10000000000000000
2. Changes the least significant bit to its counterpart

In order to receive the original values, these two steps have to be reversed.

## Reversing

I will use a Kotlin script to perform the reversing. 
- reversing the addition is subtraction
- in order to switch the last bit we can simply apply a XOR with 1.
- after the code point is obtained, we get its corresponding `Character` and join to `String` with no separator.

```kotlin
val input = arrayOf(65639, 65645, 65632, 65638, 65658, 65653, 65609, 65584, 65650, 65630, 65640, 65634, 65586, 65630, 65634, 65651, 65586, 65589, 65644, 65630, 65640, 65588, 65630, 65618, 65646, 65630, 65607, 65651, 65646, 65627, 65586, 65647, 65630, 65640, 65571, 65612, 65630, 65649, 65651, 65586, 65653, 65621, 65656, 65630, 65618, 65652, 65651, 65636, 65630, 65640, 65621, 65574, 65650, 65630, 65589, 65634, 65653, 65652, 65632, 65584, 65645, 65656, 65630, 65635, 65586, 65647, 65605, 65640, 65647, 65606, 65630, 65644, 65624, 65630, 65588, 65649, 65585, 65614, 65647, 65660)

val result = input.map { (it - 65536) xor 1 }.joinToString("") { Character.toString(it) }

print("The flag is: $result")
```

Running the code finally yields:

`The flag is: flag{tH1s_ic3_cr34m_i5_So_FroZ3n_i"M_pr3tTy_Sure_iT's_4ctua1ly_b3nDinG_mY_5p0On}`

Which is the correct solution to the challenge.

---
_HSCTF 8 was held on 14-18.06.2021_

_Thanks to [kzawistowski](https://github.com/kzawistowski) for participating in the CTF with me!_
