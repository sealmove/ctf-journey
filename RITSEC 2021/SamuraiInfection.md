```
Hey Samurai, some corpo went catatonic in a local braindance den. His implants were all nonfunctional and it looks like
something wiped him, unfortunately he was running some sort of archaic PortholeOS so we can't do anything to recover
him. The halo however survived, looks like it was write protected before use. We pulled this off of it, see what you
can make of it.

NOTE: This challenge uses a custom flag format of RITSEC{}.

Hint: 1234

A corpo body hit the floor

5678

the base drop didn't wait

ATGC

is that shorthand that I see

Hint 2: Hey Samurai, some of my units have worthless ch00ms leading them. What should I do?

Flatline them? Will do.

Author: DataFrogman
```
[infection](https://www.dropbox.com/s/funifq0hh0y956q/infection?dl=0)

By `cat`ing `infection` with see a sequences of nucleotides. They are separated from one another by a single space.

The lengths of all sequences are in range `1..4`, therefore if we treat them as base 4 numbers, the biggest possible number is 4 <sup>4</sup> = 256. This means we can treat each sequence as a byte.

Now all is left is to assign each nucleotide to a digit in base 4. [According to wikipedia](https://en.wikipedia.org/wiki/Quaternary_numeral_system#Genetics), the correct mapping is `A -> 0`, `C -> 1`, `G -> 2`, `T -> 3` (alphabetical).

Since the file only contains the characters `' '`, `'A'`, `'C'`, `'G'`, `'T'`, it's easy to hack a converter:
```nim
import strutils, sequtils, streams

type Nucleotide = enum
  A, C, G, T

proc parseQuaternary(s: string): int =
  let qits = s.mapIt(parseEnum[Nucleotide]($it))
  var n = newStringOfCap(qits.len * 2)
  for q in qits:
    n &= q.ord.toBin(2)
  result = parseBinInt(n)

let dna = newFileStream("infection")

var
  curr: string
  res: seq[int]

while not dna.atEnd:
  let c = dna.readChar
  if c == ' ':
    res.add parseQuaternary(curr)
    curr = ""
  else:
    curr.add c

let
  antidote = open("antidote", fmWrite)
  bytes = res.mapIt(it.char)

for b in bytes:
  antidote.write b

close(antidote)
```

```sh
$ file antidote
antidote: MS-DOS executable
```

Unfortunately the executable doesn't work. I talked with the author a few minutes before the end of the event, and he told me he discovered a bug in his encoder. His intention was for the executable to work and give the flag.