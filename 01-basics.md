# Basics — File Navigation & Permissions

## Level 0 — SSH Login
The entry point to Bandit. The goal is to connect to the remote machine using SSH.
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```
The `-p` flag specifies the port to connect through — in this case, port 2220. After running the command, I entered the password `bandit0` to log in.

Once inside, I explored the environment:
```bash
ls
cat readme
```
The `readme` file contained the password for the next level.
**Key concepts:** SSH remote login, specifying a port with `-p`.
