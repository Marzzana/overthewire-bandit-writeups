# Basics - File Navigation & Permissions

## Level 0 - SSH Login
The entry point to Bandit. The goal is to connect to the remote machine using SSH.
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```
The `-p` flag specifies the port to connect through - in this case, port 2220. 

After running the command, I entered the password `bandit0` to log in.

Once inside, I explored the environment:

```bash
ls
cat readme
```

The `readme` file contained the password for the next level. Password revealed.

**Key concepts:** SSH remote login, specifying a port with `-p`.

---

## Level 1 - Dashed Filename
The home directory contained a file named `-`. Running `cat -` doesn't work because the shell interprets `-` as a reference to STDIN/STDOUT rather than a filename.

The solution is to specify the path explicitly:

```bash
ls
cat ./-
```

By using `./` before the filename, the shell treats it as a file path instead of a special character.
The file contained the password for the next level. Password revealed.

**Key concepts:** Handling special filenames, relative path notation with `./`.

---

## Level 2 - Spaces in Filename
The file in this level was named `spaces in this filename`, which causes issues because the shell interprets each word as a separate argument.

The solution is to escape each space with a backslash:

```bash
ls
cat ./spaces\ in\ this\ filename
```

The file contained the password for the next level. Password revealed.

**Key concepts:** Escaping spaces in filenames with `\`.

---

## Level 3 - Hidden Files
The home directory contained a folder named `inhere`. Inside it, `ls` showed nothing - the file was hidden.

In Linux, files starting with `.` are hidden by default. The `-a` flag reveals them:

```bash
cd inhere
ls -a
cat ...Hiding-From-You
```

This revealed a hidden file called `...Hiding-From-You` which contained the password for the next level.

The hidden file contained the password for the next level. Password revealed.

**Key concepts:** Hidden files in Linux, using `ls -a` to reveal them.

---

## Level 4 - Finding Human-Readable Files

The home directory contained a folder named `inhere`. 

```bash
cd inhere
ls
```

Inside it, there were 9 files, all with names starting with `-`.

Using the `./` trick from Level 1, I used `cat` on each file until I found the one containing readable text. Most files contained garbled binary output.

Only one file contained human-readable text - and that was the password for the next level. Password revealed.

**Key concepts:** Navigating multiple dashed filenames, identifying human-readable content among binary files.
