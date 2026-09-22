# OverTheWire Bandit — Levels 0 to 11

**Platform:** OverTheWire | **Category:** Linux / CLI | **Difficulty:** Beginner | **Date:** 20/09/2026 

## Objective
Work through the first twelve Bandit levels to build fluency with the
Linux command line — navigation, file inspection, searching, and basic
encoding.

## Level summary

| Level | Concept | Key command |
|-------|---------|-------------|
| 0 → 1 | SSH on a non-standard port | `ssh -p 2220` |
| 1 → 2 | Filenames the shell interprets as flags | cat ./- |
| 2 → 3 | Filenames containing spaces | cat "spaces in this filename" |
| 3 → 4 | Hidden files | ls -la |
| 4 → 5 | Spotting the one ASCII text file among binary data | file ./* |
| 5 → 6 | Searching by size and permission | find . -type f -size 1033c ! -executable |
| 6 → 7 | Searching by owner and group, discarding errors | find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null |
| 7 → 8 | Finding a line inside a large file | grep "millionth" data.txt |
| 8 → 9 | Finding the one non-repeating line | sort data.txt \| uniq -u |
| 9 → 10 | Readable strings inside a binary | strings data.txt \| grep "=" |
| 10 → 11 | Base64 decoding | base64 -d data.txt |

## What I actually learned
A filename starting with - gets read by cat as an option flag, not a file. Prefixing it with ./ forces the shell to treat it as a path.
I practiced multiple commands and can now identify different file types, search for files based on specific conditions,
find unique lines and extract readable text from binary files.
These levels helped me become more comfortable with the terminal

## Where I got stuck
Level 6 took longest. My first find searched only the current directory
I hadn't realised I needed to start from /. Then the output flooded with 'Permission denied' until I learned to redirect stderr
