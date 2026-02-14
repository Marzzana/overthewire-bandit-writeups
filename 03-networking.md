# Networking - SSH Keys, Netcat, SSL & Port Scanning

## Level 13 - SSH Key-Based Authentication

This was weird. Instead of a password, the level gave me a private key that I had to use to log into the next user.

This required studying how SSH key-based authentication works.
I created a `.ssh` directory in the home directory, saved the private key inside it using `nano`, and then used the `-i` flag to authenticate with it:
```bash
ssh -i ~/.ssh/private.key bandit14@bandit.labs.overthewire.org -p 2220
```

The `-i` option tells SSH to use a specific private key file instead of a password. Once logged in as bandit14, I could access the password file directly since the permissions were set for that user only.  
Password revealed.

**Key concepts:** SSH key-based authentication, the `-i` flag, storing private keys in `~/.ssh`.

---

## Level 14 - Netcat & Localhost Communication

A lot of reading went into this one. I set up a Linux VM, and from this point forward I was working on Ubuntu. I studied networking concepts - virtual machines, hostnames, localhost, ports, and how connections work.

The task was to submit the current level's password to localhost on port 30000. The key to understanding this: `localhost` refers to the machine you're currently on, and port 30000 is the specific channel waiting to receive the password.

The tool for establishing network connections is `nc` (netcat):
```bash
nc localhost 30000
```

After connecting, I typed in the password for the current level.  
The server on the other end accepted it and returned the password for the next level. Password revealed.

**Key concepts:** Networking fundamentals (localhost, ports), establishing connections with `nc`, sending data over a network connection.

---

## Level 15 - Encrypted Connections with Ncat

Again, read a ton. This level was about encrypted communication. I studied several network tools - `nc`, `ncat`, and `socat`, each serving different purposes, with `socat` being the heaviest for complex setups.  
I also read about network information tools like `nmap`, `netstat`, `ip`, and `ss`.

The task was the same as the previous level - submit the password to localhost on port 30001, but this time through an encrypted channel.
```bash
ncat --ssl localhost 30001
```

`ncat` is the upgraded version of `nc` that supports TLS/SSL encrypted communication.
The `--ssl` flag opens the connection on an encrypted channel. After connecting and typing the password - password revealed.

**Key concepts:** TLS/SSL encrypted connections, `ncat` vs `nc`, network information tools (`nmap`, `netstat`, `ip`, `ss`).

---

---

## Level 16 - Port Scanning & Key Discovery

A bit easier once I had read all those damn commands from the previous levels.

The goal was to find a service running on a port between 31000 and 32000. I started with `ss` to look at socket statistics - but running `ss` alone dumps every socket connection regardless of status, which prints a LOT of lines.  
So I added the `-l` flag to only show ports in listening mode. Still too many results, so I kept narrowing it down, and eventually got to this command:
```bash
ss -l | sort -u | grep "LISTEN" | grep "31"
```

Each step peeled away more noise. `sort -u` removed duplicates, the first `grep` kept only listening sockets, and the second `grep` narrowed to lines containing "31" to stay within the 31000–32000 range.  
The sorting might have been redundant, but it did the job - I was left with about 10 lines, and from there it was easy to spot the relevant ports with the human eye.

I then tried connecting to each one with `ncat --ssl` and typing the password. Eventually, one of them responded - not with a password this time, but with a private key.  
I copied it, saved it using `nano` into `~/.ssh`, set the permissions to 600 (very important!), and logged into the next level:
```bash
ncat --ssl localhost 31xxx
ssh -i ~/.ssh/keyfile bandit17@bandit.labs.overthewire.org -p 2220
```

Password revealed.

**Key concepts:** Socket inspection with `ss`, iteratively filtering output with chained commands, setting file permissions with `chmod 600` for private keys.

---

## Level 20 - Client-Server Communication

Two whole days spent on this one, and 99% of the thinking was in the completely wrong direction.

The home directory contained a binary called `suconnect`. Given a port number, it connects to that port, reads a line, and if the line matches the previous level's password, it transmits the next password.

So I started thinking - okay, I need to find which port has the thing that gives me the password. I did a HUGE amount of overthinking.  
I scanned for open ports using `nmap` and `nc`:
```bash
nmap -sV localhost
nc -zv localhost 2>&1 | grep succeeded
```

I found port 12345, tried it with `suconnect`, and it told me the password matched and that it was sending the next one. Great, right? NO. Where is it sending the password to?  
This was a decoy, and it completely set me off in the wrong direction.  
I spent two days trying to figure out where the password was going - listeners on other ports, intercepting packets, controlling processes - nothing worked.

A great lesson came out of this: **if it's taking too long, release the idea. The assumptions you built on are probably wrong. Rethink the whole problem.**

The real answer was painfully simple. `suconnect` didn't care which port it got - it just acted as a client. All it needed was a server. MY server.  
I created my own listener on a port of my choosing, then ran `suconnect` pointing at that port:
```bash
nc -l localhost 25000
./suconnect 25000
```

I used `tmux` to split into separate terminals - one running the listener, the other running `suconnect` - so I could interact with both sides simultaneously.

And then both of them just sat there, waiting. The listener waited for input, and `suconnect` waited to read. I typed the previous level's password into the listener - immediately `suconnect` confirmed the match and transmitted the next password right back to the listener. Password revealed.

A side note on the port scanning command - `nc -zv` outputs to stderr (stream 2), not stdout (stream 1). Since `grep` only reads from stdout, I had to redirect stderr to stdout with `2>&1`. The `&` tells the shell that `1` is the stdout stream, not a filename.

**Key concepts:** Port scanning with `nmap` and `nc`, creating listeners with `nc -l`, `tmux` for managing multiple terminals, understanding stdin/stdout/stderr streams (0, 1, 2), stream redirection with `2>&1`.

