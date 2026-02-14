# Text Processing - Grep, Piping, Encoding & Compression

## Level 7 - Searching Within a File

The home directory contained a file called `data.txt` with a massive amount of text. The password was stored on the line next to the word "millionth".
```bash
grep millionth data.txt
```

The `grep` command searches for a specific string inside a file and outputs the matching line. Password revealed.

**Key concepts:** Using `grep` to search for text within files.

---

## Level 8 - Finding Unique Lines

The home directory contained a file called `data.txt`, full of duplicate lines. The password was the only line that appeared exactly once. After a lot of research, I landed on this:
```bash
sort data.txt | uniq -u
```

This took some reading to understand.  
`sort` groups identical lines together.   
The `|` operator pipes that sorted output into `uniq -u`, which only prints lines that appear once.  
The `uniq` command has other useful options too - `-d` prints only the duplicate lines, and `-c` precedes each line with a count of how many times it occurred. What helped me most here was reading about the difference between piping (passing output between programs) and redirection (passing output to files). 

Password revealed.

**Key concepts:** Piping with `|`, sorting with `sort`, filtering unique lines with `uniq -u`.

---

## Level 9 - Extracting Strings from Binary

This was not easy. The `data.txt` file was some kind of binary mess - trying to `cat` it scrambled my entire terminal.  
I had to read about the `strings` command, which extracts readable text from files that are otherwise unreadable.

The hint was that the password was near several `=` characters. So I piped the output of `strings` into `grep`:
```bash
strings data.txt | grep '='
```

This narrowed the output down to only lines containing `=` signs, making the password easy to spot. Password revealed.

**Key concepts:** Using `strings` to extract readable text from binary files, chaining commands with pipes.
