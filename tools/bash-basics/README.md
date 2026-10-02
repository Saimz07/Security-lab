# Bash Basics

Three small scripts written while completing *Foundations of Bash
Scripting for Cybersecurity*. Each one was written to practice a
specific pattern that comes up constantly in security tooling:
checking file state, searching a filesystem by condition, and pulling
structured information out of unstructured text.

## 1. File inspector

Accepts a filename and reports whether it exists, its type, and its size.

```bash
./file-inspector.sh somefile.txt
```

**What it does:** Checks whether the target exists, determines its file type,
and gets its size using stat.
I also used numfmt to convert the raw byte count into a human-readable size.

**What tripped me up:** The stat section was initially confusing because 
I had to understand why stat -c %s returns the size in bytes and
 then why that value needs to be passed to numfmt separately.
I also had to remember to quote variables when they contain filenames or paths.

## 2. Executable / size finder

Accepts a directory and finds executable files, and files larger than
a chosen size.

```bash
./find-exec-large.sh /some/directory 10M
```

**What it does:** Uses find to recursively search the target directory. 
It checks file permissions to find executable files and
uses -size to find files larger than the size supplied by the user..

**What tripped me up:** The find permission syntax took some figuring out,
particularly the difference between -perm -u+x and other -perm forms. 
I also had to understand how find -size interprets units such as k, M, and G. 
Another small issue was making sure the script handled the directory and size arguments correctly.

## 3. Log parser

Reads a log file, extracts something useful, and counts occurrences.

```bash
./log-parser.sh /var/log/somefile.log
```

**What it extracts:** Parses web server log data to extract HTTP status codes and count how frequently each status code appears. 
The script uses tools such as awk, grep, sort, and uniq to filter and aggregate the log data.

**What tripped me up:** The biggest challenge was understanding how the different text-processing commands work together. 
In particular, I had to figure out which awk field contained the HTTP status code, 
then understand why sort has to come before uniq -c for the counting to work correctly. 
The grep filtering and awk field extraction also took some trial and error.

## Why this matters
These are small, but the patterns are the same ones used in real security tooling: checking state, 
searching by condition, and turning raw log text into a count you can act on.

The first script helped me understand how Bash can inspect files and work with command output. 
The second introduced practical filesystem searching based on permissions and file size. 
The third brought several Linux text-processing tools together to turn raw logs into structured information.

The log parser in particular is a direct precursor to a Python log-analysis tool I plan to build next,
using the same logic in a different language.
