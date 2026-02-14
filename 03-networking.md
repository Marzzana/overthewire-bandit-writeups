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

