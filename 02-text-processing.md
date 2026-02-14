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

This took some reading to understand. `sort` groups identical lines together.   
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

---

## Level 10 - Base64 Decoding

This level required some reading about Base64.  
Base64 is an encoding scheme that converts binary data into text, useful for systems that can only handle text input.  
It works by taking a block of 3 bytes (24 bits), splitting it into 4 blocks of 6 bits, and assigning each block a character from a set of 64 characters (A-Z, a-z, 0-9, +, /, and = for padding).

An interesting side note - this encoding increases file size by about 33%. We start with 3 bytes (24 bits), and end up with 4 characters of 8 bits each (32 bits). So we go from 3 bytes to 4 bytes, an increase of a third.

The `data.txt` file contained text that had already been encoded in Base64. Decoding it was straightforward with the `-d` flag:
```bash
base64 -d data.txt
```

Password revealed.

**Key concepts:** Base64 encoding/decoding, understanding how encoding increases file size.

---

## Level 11 - ROT13 Cipher

This one was cool honestly. The `data.txt` file had been encoded with ROT13 - a simple substitution cipher that shifts each letter 13 places in the alphabet.  
For example, `a` becomes `n`, and `z` becomes `m`.

The `tr` (translate) command turned out to be extremely powerful for this. It works by mapping one set of characters to another:  
`tr [SET1] [SET2]`. I realized that in ROT13, `a-m` maps to `n-z` and `n-z` maps back to `a-m`:
```bash
cat data.txt | tr 'a-m,n-z' 'n-z,a-m' | tr 'A-M,N-Z' 'N-Z,A-M'
```

The sets have to be in alphabetical order - writing `n-m` would throw an error, so they need to be split into `n-z` and `a-m`. The second `tr` handles uppercase letters.  
Technically the second pipe wasn't necessary, but I did it out of careful noobiness. Password revealed.

**Key concepts:** ROT13 cipher, character translation with `tr`, chaining multiple translations.

---

## Level 12 - Compression Chains

This was the hardest one yet by far.  
The `data.txt` file was a hex dump of a file that had been compressed multiple times with different tools.

The key insight was learning that hex dumps contain compression signatures at the start of the file - `1F 8B 08` for gzip, `42 5A 68` for bzip2.  
These stamps tell you which decompression command is needed to peel back each layer.
To decompress with gzip:
```bash
gzip -d filename.gz
```
To decompress with bzip2:
```bash
bzip2 -d filename.bz2
```

The `-d` flag is for decompression. To compress, simply run the commands without it (`gzip filename`, `bzip2 filename`).

After several rounds of decompressing, a new problem appeared - they started hiding a decompressible file inside an undecompressible one. I had to manually extract the relevant portion using `cat > newfilename.txt`, edit it with `nano`, and then decompress again.

This happened multiple times. Layer after layer of compression, each one requiring identification and the right tool to unwrap.  
Eventually, after what felt like forever - password revealed.

**Key concepts:** Hex dumps, compression signatures (gzip, bzip2), multi-layered decompression, manual file extraction with `cat >` and `nano`.
