Creation date: Sunday, February 8th 2026, 2:45:42 am

#### **Viewing File Contents**

##### Display Entire File: cat
```
cat /etc/hostname
```
`cat` prints the entire file to the screen. Short for "concatenate" - it can also combine multiple files.

##### Pass through files: less
```
less /var/log/syslog
```
`less` lets you scroll through the file

***KeyAction***
`Space` or `f`Forward one page
`b`Back one page
`↑` / `↓`Line by line
`g`Go to beginning
`G`Go to end
`/pattern`Search forward
`n`Next search result
`q`Quit

> **Tip**: `less` is more powerful than `more`. The joke is “less is more”.

##### View First Lines: head

```
head /var/log/syslog
```
Shows the first 10 lines by default.

```
head -20 /var/log/syslog
```
Shows the first 20 lines.

```
head -1 /etc/passwd
```
Shows just the first line.

##### View Last Lines: tail
```
tail /var/log/syslog
```
Shows the last 10 lines by default.

```
tail -50 /var/log/syslog
```
Shows the last 50 lines.

##### View Files in Real-Time: tail -f
```
tail -f /var/log/syslog
```
The `-f` flag means "follow". New lines appear as they're written to the file. 

> **This is essential for debugging.** When something goes wrong, you `tail -f` the log file, reproduce the problem, and watch the errors appear.

---
#### **Searching File Contents**

##### Introduction to grep
`grep` searches for patterns in files. The name comes from "global regular expression print"

##### Basic Search
```
grep "error" /var/log/syslog
```
finds all lines containing "error" in the file

##### Case-Insensitive Search
```
grep -i "error" /var/log/syslog
```
The `-i` flag ignores case. Matches "error", "ERROR", "Error", etc

##### Show Line Numbers
```
grep -n "error" /var/log/syslog
```
The `-n` flag shows line numbers

##### Invert Match
```
grep -v "comment" config.txt
```
The `-v` flag shows lines that DON'T match. Useful for filtering out noise.

##### Search Recursively
```
grep -r "password" /etc/
```

##### Count Matches
```
grep -c "error" /var/log/syslog
```
The `-c` flag counts matching lines instead of showing them

##### Show Context

```
grep -B 2 -A 2 "error" /var/log/syslog
```
- `-B 2` shows 2 lines **B**efore the match
    
- `-A 2` shows 2 lines **A**fter the match
    

Useful for seeing what happened around an error.

##### Basic Regular Expressions

`grep` supports patterns, not just literal text:

```
grep "^root" /etc/passwd
```
`^` means “starts with”. Finds lines starting with “root”.

```
grep "bash$" /etc/passwd
```
`$` means “ends with”. Finds lines ending with “bash”.

```
grep "user[0-9]" /etc/passwd
```
`[0-9]` matches any digit. Finds “user0”, “user1”, etc.

##### Combine with Pipes
```
cat /var/log/syslog | grep "error"
```
Same result as `grep "error" /var/log/syslog`, but demonstrates piping output through grep.

---
#### **File Information**

##### What type of file is this?
```
file /bin/ls
```
`file` examines the content to determine the file type. It doesn’t rely on extensions.

##### Detailed File Information
```
stat /etc/passwd
```
Shows size, permissions, owner, timestamps, and more

##### Count Words, Lines, Characters

```
wc /etc/passwd
```

Output:

```
  32   54 1842 /etc/passwd
```
That’s: 32 lines, 54 words, 1842 characters.

```
wc -l /etc/passwd
```
Just lines: 32

```
wc -w /etc/passwd
```
Just words: 54

```
wc -c /etc/passwd
```
Just characters (bytes): 1842

##### How Big is This Directory?
```
du -sh /var/log
```
`du` = disk usage. The `s` flag summarizes, `-h` makes it human-readable

##### How Much Disk Space is Free?
```
df -h
```
`df` = disk free. Shows all mounted filesystems and their space usage.

---
#### **Summary**

- `cat` for small files, `less` for large files
    
- `head` and `tail` for beginning and end
    
- `tail -f` to watch logs in real-time (essential for debugging)
    
- `grep` searches file contents - one of the most-used commands
    
- `file`, `stat`, `wc` tell you about files
    
- `du` for directory sizes, `df` for disk space
---
#### **Definitions**

**cat**: Concatenate - display file contents or combine multiple files.

**less**: A pager program for viewing file contents one screen at a time.

**grep**: Global Regular Expression Print - searches for patterns in files.

**Pipe**: The `|` character that sends output from one command to another.

**Regular Expression**: A pattern-matching syntax for searching text (also called “regex”).

---
***Previous***: [[file system]]                                          ***Next***: [[users, groups & permissions]]