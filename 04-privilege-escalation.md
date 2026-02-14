# Privilege Escalation - Setuid, Cron Exploitation, Restricted Shells & Scripting

## Level 18 - Remote Command Execution

This was very weird at first. Every time I tried to log in to the user via SSH, I was immediately kicked out.

This led me to read about the `.bashrc` file - a configuration file that runs every time a user opens a terminal or logs in via SSH. I realized they had edited it to instantly disconnect anyone who logs in remotely.

Luckily, there's a way to execute a command without actually starting a session. By appending a command right after the SSH login, it runs the command and then disconnects - before `.bashrc` gets a chance to kick you out:
```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat /home/bandit18/readme
```

This logged in, ran `cat`, printed the password, and disconnected - all in one shot. A very nice trick. Password revealed.

**Key concepts:** Understanding `.bashrc`, remote command execution via SSH without starting a session.

---

## Level 19 - Setuid Exploitation

So simple, yet I couldn't grasp it for a while.

This required a lot of reading about `setuid` and `setgid`. In short - `setuid` is a special permission bit that replaces the `x` in a file's permissions with an `s`.  
When set on an executable, it means anyone who runs that file temporarily gets the privileges of the file's owner.

The home directory contained a binary called `bandit20-do`. Running it gave a deceptively vague hint: "Run a command as another user. Example: `./bandit20-do id`".  
The `id` in the example is NOT related to the solution - it's just an example command! which is confusing when the whole level revolves around users and IDs.

Once it clicked, I realized the binary lets you run any single command as the bandit20 user:
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

This ran `cat` as bandit20, who had permission to read the password file. Password revealed.

**Key concepts:** `setuid`/`setgid` permissions, running binaries with `./`, privilege escalation through setuid executables.
