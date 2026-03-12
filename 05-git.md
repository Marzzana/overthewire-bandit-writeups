# Git - Internals, Cloning & Version Control

## Level 27 - Git Clone

The first git level. The task was to clone a repository from the Bandit server. The clone command uses an SSH URL where the port (2220) is specified with `:` right after the hostname, unlike regular SSH where you use `-p`:
```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
```

It prompted for a password, the same as the current level's. Once cloned, the answer was sitting right there:
```bash
cd repo
cat README
```

Password revealed.

A small note - git needs to be installed first with `sudo apt install git-all`.

**Key concepts:** Cloning repositories via SSH, SSH URL format with port notation.

---

## Level 28 - Git Internals & Unpacking

Again I cloned the repo, but this time the README showed censored credentials - no password, no direction.

My only lead was to explore the `.git` directory. This forced me to dive deep into git internals. I studied what git object files actually are - blobs (file content), trees (directory structure), and commit objects (snapshots pointing to trees). I also read about the index and HEAD files, though I didn't fully grasp the complete flow of git at this point - just enough of the internals to work with them directly.

When I started exploring the objects directory, something unexpected came up - there were no object files, just three packed files: `.idx`, `.pack`, and `.refs`. These are compressed versions of the repo, designed to make it lighter for transport. There's a command to extract the actual objects from them:
```bash
git unpack-objects < ~/unpacked/pack/<the .pack filename>
```

But git is efficient — it won't unpack objects that already exist in the repo. So I had to move the pack directory somewhere else first:
```bash
mv pack ~/unpacked
```
The `<` redirection is needed because `git unpack-objects` reads its input from stdin.

Then back in `repo/objects`, the unpack worked. Each object got its own directory, and to inspect them I used:
```bash
git cat-file -t <object name>    # shows the type (blob, tree, commit)
git cat-file -p <object name>    # shows the content
```

One thing that took me some exploring to realize - the object's full name is the concatenation of the directory name and the filename inside it.

I went through each object's content until I found the version of the README that had the uncensored password. Password revealed.

**Key concepts:** Git internals (blobs, trees, commits), packed files (`.idx`, `.pack`, `.refs`), unpacking with `git unpack-objects`, inspecting objects with `git cat-file`.

---

## Level 29 - More Unpacking

Same deal as the previous level. Clone, unpack, investigate.
```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
```

Then setting up the environment for unpacking:
```bash
mkdir ~/unpacked                # from home
mv pack ~/unpacked              # from repo/objects
git unpack-objects < ~/unpacked/pack/<.pack filename>   # from repo/objects
```

Then inspecting each object:
```bash
ls <directory name>
git cat-file -p <complete object name>
```

The `.pack` filename is a 40-character hash - not important to remember, just important to use. Went through the objects until the password showed up. Password revealed.

**Key concepts:** Reinforcing git internals workflow - clone, unpack, inspect.

---

## Level 30 - Organized Investigation

Same process as before, but this time I was more organized about it. In the previous levels I was jumping straight to `-p` to check content, and since I understood the difference between blob, tree, and commit content, I could handle it. But this time I first ran `-t` on every object to see its type, built a clear picture in my head of the file relationships and commit history, and only then looked at the blob contents.

This was more for myself than for solving the level - I just wanted to have a full picture of the commit structure rather than brute-force my way through. The password was in one of the blobs. Password revealed.

**Key concepts:** Systematic approach to git object inspection - type first, content second.

---

## Level 31 - Git Push

This time I had to push changes to the remote repo. This required reading about the more common side of git - `add`, `commit`, and `push`. After diving into the internals, these higher-level commands clicked much better.

The task was to create a file called `key.txt` containing the text "May I come in?" and push it to the remote. But when I tried `git add key.txt`, it got rejected - the repo had a `.gitignore` file that was set to ignore all `.txt` files. So I updated `.gitignore` to allow my file through, then:
```bash
git add key.txt
git commit -m "added new file"
git push ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo master
```

What's cool is understanding what these commands do under the hood after working with the internals manually. `git add` creates a blob for the file's content and stages a tree draft in the index. `git commit` creates a tree object from the index and a commit object pointing to it, then updates `refs/heads/master`. `git push` updates the remote repo's `.git` - and here's something that surprised me: the remote repo is what's called a "bare repo," meaning it's just a `.git` directory with no project files. This makes sense - its only purpose is to let developers share versions of the code. When I push, other devs can fetch my updated version.

After pushing, it prompted for the password, and then the next level's password was revealed. Password revealed.

**Key concepts:** `git add`, `git commit`, `git push`, `.gitignore`, bare repositories, understanding git commands through their internals.
