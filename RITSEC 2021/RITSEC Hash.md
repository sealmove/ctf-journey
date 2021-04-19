```
Hmmm.. we found this hash along with a white paper explaining this custom hashing algorithm.

Can you break it for us?

hash : 435818055906

Flag should be submitted as RS{<cracked hash>}

Author: 1nv8rZim
```
[RITSEC_HASH](<https://www.dropbox.com/s/giqh9yhlbfeegse/RITSEC_HASH.pdf?dl=0>)

**Hint:**
```
The red crosses in the hash diagram are addition
```
**Hint:**
```
Goal here is not to test ur hash cracking rigs, please use rockyou.txt

Solution should be first match and hopefully will be pretty recognizable
```

Studying the pdf, we implement the hash function and bruteforce the password with the infamous *rockyou* wordlist.
```nim
import strformat, strutils, sequtils

const
  seed = "RITSEC".mapIt(it.byte)
  target = "435818055906".parseHexStr.mapIt(it.byte)
  wordlist = "rockyou.txt"

proc H(h: seq[byte], x, r: byte): seq[byte] =
  var (A, B, C, D, E, F) = (h[0], h[1], h[2], h[3], h[4], h[5])
  @[((C xor E) and F) + B + D shl 2 + x + r, A, D shl 2, B shr 5, A + F, D]

proc hash(h: seq[byte], x: byte): seq[byte] =
  result = h
  for r in 0 .. 12:
    result = H(result, x, r.byte)

echo &"[+] Loading wordlist: {wordlist}"
let passwords = readFile(wordlist).splitLines
echo &"[+] Bruteforcing with: {wordlist}"
for i, password in passwords:
  var h = seed
  for c in password:
    h = hash(h, c.byte)
  if h == target:
    echo &"[+] Cracked password: {password}"
    quit(QuitSuccess)
```

**Remark**:
Even without reading the second hint, we know we cannot reverse the function because it uses `and`, and therefore it is impossible to reverse the values `C`, `E`, `F`.
