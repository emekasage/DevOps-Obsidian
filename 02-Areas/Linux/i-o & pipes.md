Creation date: Wednesday, February 11th 2026, 3:29:43 am

```
ls /nonexistent
```

Output:

```
ls: cannot access '/nonexistent': No such file or directory
```

This error message went to stderr, not stdout. They look the same on screen, but they’re different streams.

---
#### **Redirection**

##### Redirect Output to File (Overwrite)

```
ls /etc > listing.txt
```

The `>` sends stdout to a file instead of the screen. Creates the file or overwrites if it exists.

```
cat listing.txt
```

##### Redirect Output to File (Append)

```
echo "Line 1" > file.txt
echo "Line 2" >> file.txt
cat file.txt
```

Output:

```
Line 1
Line 2
```

The `>>` appends to the file instead of overwriting.

##### Redirect Input from File

```
wc -l < /etc/passwd
```

The `<` reads from a file instead of keyboard. This counts lines in `/etc/passwd`.
##### Redirect Errors

```
ls /nonexistent 2> errors.txt
```

The `2>` redirects stderr (file descriptor 2) to a file.

```
cat errors.txt
```

Output:

```
ls: cannot access '/nonexistent': No such file or directory
```

##### Redirect Both stdout and stderr

```
ls /etc /nonexistent &> all-output.txt
```

The `&>` redirects both streams to the same file.

Or separately:

```
ls /etc /nonexistent > stdout.txt 2> stderr.txt
```
##### Discard Output

```
ls /etc > /dev/null
```

`/dev/null` is a black hole - output goes nowhere.

```
command 2>/dev/null
```

Hide error messages.

```
command &>/dev/null
```

Hide all output.

---
#### **Pipes**

##### The Pipe Operator

The pipe `|` sends stdout from one command to stdin of another.

```
ls /etc | head -5
```

Output:

```
adduser.conf
alternatives
apt
bash.bashrc
bindresvport.blacklist
```

`ls` output goes directly into `head` without creating a file.
##### Chaining Commands

```
cat /etc/passwd | grep bash | wc -l
```

This:

1. Reads `/etc/passwd`
    
2. Filters lines containing “bash”
    
3. Counts those lines

Result: number of users with bash as their shell.

##### Practical Examples

**Find a command in your history**

```
history | grep ssh
```

**See who’s logged in, sorted:**

```
who | sort
```

**Count files in a directory:**

```
ls | wc -l
```

**Find large log files:**

```
du -h /var/log/* | sort -h | tail -10
```

**Search running processes:**

```
ps aux | grep nginx
```

---

#### **Useful Filter Commands**

These commands are designed to work in pipelines.
##### Create Sample Files

```
echo "Charlie" > names.txt
echo "Alice" >> names.txt
echo "Bob" >> names.txt
echo "Alice" >> names.txt
echo "Bob" >> names.txt
echo "Alice" >> names.txt
```

##### sort - Sort Lines

```
sort names.txt
```
Alphabetically sorts the names

```
sort -r names.txt
```
Reverse sort (Z to A).

```
du -h /var/log/* 2>/dev/null | sort -h
```

Sort by human-readable sizes (K, M, G). The `-n` flag sorts numerically if you have plain numbers.

##### uniq - Remove Duplicates

```
sort names.txt | uniq
```

Remove adjacent duplicates. Usually used with `sort` first.

```
sort names.txt | uniq -c
```

Count occurrences:

```
      3 Alice
      2 Bob
      1 Charlie
```
##### cut - Extract Columns

```
cut -d: -f1 /etc/passwd
```

Extract field 1, using `:` as delimiter. Shows all usernames.

```
cut -d: -f1,3 /etc/passwd
```

Extract fields 1 and 3 (username and UID).

```
echo "hello world" | cut -c1-5
```

Extract characters 1-5: `hello`

##### tr - Translate Characters

```
echo "HELLO" | tr 'A-Z' 'a-z'
```

Output: `hello` (uppercase to lowercase)

```
echo "hello world" | tr ' ' '\n'
```

Replace spaces with newlines:

```
hello
world
```

```
echo "abc123" | tr -d '0-9'
```

Delete digits: `abc`

##### tee - Write to File and Screen

```
ls /etc | tee listing.txt
```

Output goes to both the screen and the file.

```
ls /etc | tee listing.txt | head -5
```

Useful for debugging pipelines - save intermediate output while continuing the pipe.

##### xargs - Build Commands from Input

Some commands don’t read from stdin - they need arguments. `xargs` converts stdin into arguments.

```
cat names.txt | xargs echo "Hello:"
```

Output:

```
Hello: Charlie Alice Bob Alice Bob Alice
```

All names become arguments to `echo`. To process one at a time:

```
cat names.txt | xargs -n 1 echo "Hello:"
```

Output:

```
Hello: Charlie
Hello: Alice
Hello: Bob
Hello: Alice
Hello: Bob
Hello: Alice
```

The `-n 1` means “one argument per command.”

**Practical example** - create a file for each name:

```
sort names.txt | uniq | xargs touch
ls
```

This creates files named `Alice`, `Bob`, and `Charlie`.

---
#### **Summary**

- Programs have stdin (input), stdout (output), stderr (errors)
    
- `>` redirects stdout to file, `>>` appends
    
- `2>` redirects stderr, `&>` redirects everything
    
- `|` pipes output from one command to another
    
- Filter commands: `sort`, `uniq`, `cut`, `tr`, `tee`
    
- Interactive shells have a terminal; scripts don’t
    
- `/dev/null` discards output

---
***Previous***: [[vi(m)]]                             ***Next***: