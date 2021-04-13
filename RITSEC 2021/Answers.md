```
Lookup this

answers.ritsec.club:53/udp

~knif3
```

Querying the DNS server, we observe it returns 3 CNAME records with randomly generated names. Querying one of them, gives us 3 more similar CNAME records. Therefore we *dig* recursively and see what happens:

```nim
import osproc, strformat, strutils

var targets = @["answers.ritsec.club"]

while true:
  var newTargets: seq[string]
  for t in targets:
    let reply = execCmdEx(&"dig +short @answers.ritsec.club {t}").output
    for line in reply.splitLines[0..^2]: # last line is empty
      echo line
      newTargets.add line
  targets = newTargets
```

Gives us:
```sh
(...)
qowowynlflfnwaby.answers.ritsec.club.
pfbpysbbfvttrekf.answers.ritsec.club.
lnnxinxbimaritnr.answers.ritsec.club.
random_txt_record_mirueq.answers.ritsec.club.
calxwqwwimldhuyi.answers.ritsec.club.
fustxqbbkgtwsscn.answers.ritsec.club.
mfyxclljenoqueti.answers.ritsec.club.
(...)
```

We can get the flag by querying for `TXT` record
```sh
dig @answers.ritsec.club random_txt_record_mirueq.answers.ritsec.club TXT
```

`mirueq` seems to be generated on the fly and be *valid* for a short amount after retrieving it (~1 minute or something).