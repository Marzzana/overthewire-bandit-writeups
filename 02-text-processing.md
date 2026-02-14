# Text Processing - Grep, Piping, Encoding & Compression

## Level 7 - Searching Within a File

The home directory contained a file called `data.txt` with a massive amount of text. The password was stored on the line next to the word "millionth".
```bash
grep millionth data.txt
```

The `grep` command searches for a specific string inside a file and outputs the matching line. Password revealed.

**Key concepts:** Using `grep` to search for text within files.

---

## Level 8 — Finding Unique Lines

The home directory contained a file called `data.txt`, full of duplicate lines. The password was the only line that appeared exactly once. After a lot of research, I landed on this:
```bash
sort data.txt | uniq -u
```

This took some reading to understand. `sort` groups identical lines together. The `|` operator pipes that sorted output into `uniq -u`, which only prints lines that appear once. The `uniq` command has other useful options too — `-d` prints only the duplicate lines, and `-c` precedes each line with a count of how many times it occurred. What helped me most here was reading about the difference between piping (passing output between programs) and redirection (passing output to files). 

Password revealed.

**Key concepts:** Piping with `|`, sorting with `sort`, filtering unique lines with `uniq -u`.

