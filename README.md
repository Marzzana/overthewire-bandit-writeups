# OverTheWire: Bandit - Writeups & Analysis
A collection of detailed writeups documenting my journey through **27 levels** of the [OverTheWire Bandit](https://overthewire.org/wargames/bandit/) wargame - a hands-on CTF challenge focused on Linux fundamentals, networking, and security concepts.
These write-ups go beyond just solutions.
Each one covers my thought process, mistakes I made along the way, and the concepts I studied to understand *why* things work, not just *how*.


## Skills Demonstrated
**Linux & Bash:** File navigation, permissions, hidden files, piping, redirection, stdin/stdout/stderr, shell scripting, job control, `tmux`.

**Text Processing:** `grep`, `sort`, `uniq`, `tr`, `cut`, `strings`, `find`, hex dumps, file compression chains.

**Encoding & Encryption:** Base64 encoding/decoding, ROT13 cipher, `md5sum` hashing.

**Networking:** SSH (password & key-based authentication), `nc`/`ncat`/`socat`, TLS/SSL connections, port scanning with `nmap`, socket inspection with `ss`, localhost communication, automated server interaction using `coproc`.

**Privilege Escalation & Security:** `setuid`/`setgid` exploitation, cron job analysis, restricted shell escapes (`more` -> `vi` -> `bash`), script injection, brute-force automation.


## Writeups
| File | Levels | Theme |
|------|--------|-------|
| [01-basics.md](levels/01-basics.md) | 0–5 | File navigation, permissions, hidden files |
| [02-text-processing.md](levels/02-text-processing.md) | 6–12 | Grep, piping, encoding, compression |
| [03-networking.md](levels/03-networking.md) | 13–17 | SSH keys, netcat, SSL, port scanning |
| [04-privilege-escalation.md](levels/04-privilege-escalation.md) | 18–27 | Setuid, cron exploitation, restricted shells, scripting |


## About
I worked through these challenges as a self-study project alongside my B.Sc. in Computer Science at Ben-Gurion University.
My goal was to build real, practical Linux and security skills from the ground up - and to document everything I learned along the way.
