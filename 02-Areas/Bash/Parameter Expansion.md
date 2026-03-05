Creation date: Wednesday, March 4th 2026, 1:50:49 am

**Goal**: Master parameter expansion - the skill that separates bash professionals from beginners.

#### **Why Parameter Expansion**

First, meet the external text tools you’ll encounter:

```
# sed - search and replace
echo "hello" | sed 's/hello/hi/'

# awk - extract fields
echo "one two three" | awk '{print $2}'

# tr - translate characters
echo "hello" | tr 'a-z' 'A-Z'

# cut - extract columns
echo "a:b:c" | cut -d: -f2
```

These are powerful for processing files and pipelines. But inside bash scripts, there’s often a better way.

> _“There is nothing that says I’m a noob more than using sed, tr, awk, or cut inside of a bash script.”_

Compare these two approaches:

```
# External command (spawns new process)
filename=$(echo "document.txt" | sed 's/.txt//') 
echo $filename

# Parameter expansion (built-in, faster)
filename="document.txt" 
echo "${filename%.txt}"
```

Parameter expansion is:

- **Faster** (no subprocess)
    
- **More portable** (built into bash)
    
- **More professional** - it shows you actually know bash
    

Every `sed` or `awk` call spawns a new process. In a loop processing 1000 files, that’s 1000 subprocesses. Parameter expansion is instant - it’s built into bash.

---
#### **Default Values**

```
# Use default if variable is empty or unset
name=""
echo "Hello, ${name:-User}"
echo "$name"
# Hello, User

# Set the variable if unset or empty
name=""
echo "${name:=User}"
echo "$name"
# Now contains "User"
```

##### Practical Use: Script Arguments

```
#!/bin/bash
name="${1:-world}"
log_level="${LOG_LEVEL:-info}"
echo "Hello, $name (log: $log_level)"
```

This is how professional scripts handle missing arguments. No `if` statement needed - just `${1:-default}`.

##### The Patterns

SyntaxMeaning`${var:-default}`Use default if empty/unset (don’t change var)`${var:=default}`Set default if empty/unset (changes var)`${var:?error}`Error if empty/unset

---
#### **String Length**

```
password="secretpass"
echo "Length: ${#password}"
# 10

# Validation
if [[ ${#password} -lt 8 ]]; then
    echo "Password too short"
    exit 1
fi
```

---
#### **Removing Patterns**

##### Remove Suffix (from end)

```
filename="document.txt"
echo "${filename%.txt}"
# document

filepath="backup.2024.tar.gz"
echo "${filepath%.*}"
# backup.2024.tar (shortest match)
echo "${filepath%%.*}"
# backup (longest match)
```

##### Remove Prefix (from beginning)

```
filepath="/home/user/document.txt"
echo "${filepath#*/}"
# home/user/document.txt (shortest match)
echo "${filepath##*/}"
# document.txt (longest match - this is basename!)
```

##### Memory Aid

> _“Look at your keyboard. # is left of $, removes from left. % is right of $, removes from right.”_

- `#` is left of `$` on keyboard → removes from left (prefix)
    
- `%` is right of `$` on keyboard → removes from right (suffix)
    
- Single `#` or `%` → shortest match
    
- Double `##` or `%%` → longest match

##### Practical: Get File Parts

```
filepath="/home/user/report.tar.gz"

filename="${filepath##*/}"
# report.tar.gz (basename)
directory="${filepath%/*}"
# /home/user (dirname)
extension="${filename##*.}"
# gz
basename="${filename%%.*}"
# report
```

You just replaced `basename`, `dirname`, and some `sed` gymnastics with four parameter expansions. No subprocesses.

---
#### **Search and Replace**

```
text="hello world world"

# Replace first match
echo "${text/world/bash}"
# hello bash world

# Replace all matches
echo "${text//world/bash}"
# hello bash bash

# Delete (replace with nothing)
echo "${text// /}"
# helloworldworld
```

##### PATH Display Trick

```
# Colons to newlines - no sed needed
echo "${PATH//:/$'\n'}"
```

> _“If you’re reaching for sed to manipulate a variable, stop. There’s probably a parameter expansion for that.”

---
#### **Case Transformation**

```
name="hello world"
echo "${name^}"
# Hello world (capitalize first)
echo "${name^^}"
# HELLO WORLD (uppercase all)

name="HELLO WORLD"
echo "${name,,}"
# hello world (lowercase all)
```

---
#### **The 7 Types of Expansion**

Bash performs expansions in this order:

1. **Brace expansion**: `{1..5}`, `*.{md,txt}`
    
2. **Tilde expansion**: ~`becomes`$HOME`
    
3. **Parameter expansion**: `${var}`, `${var:-default}`
    
4. **Arithmetic expansion**: `$((count + 1))`
    
5. **Command substitution**: `$(command)`
    
6. **Word splitting**: Unquoted results split on whitespace
    
7. **Pathname expansion**: `*.txt` expands to matching files

This order matters. Brace expansion happens BEFORE variable expansion, which is why `{1..$n}` doesn’t work - `$n`hasn’t been expanded yet when brace expansion runs.

```
n=5
echo {1..$n}
# Doesn't work - prints literal {1..5}
eval echo {1..$n}
# Works but eval is dangerous
seq 1 $n
# Better approach
```

---
#### **Summary**

- **Parameter expansion** replaces the need for sed, awk, cut in most cases
    
- **Default values**: `${var:-default}` and `${var:=default}`
    
- **String length**: `${#var}`
    
- **Remove suffix**: `${var%pattern}` (shortest), `${var%%pattern}` (longest)
    
- **Remove prefix**: `${var#pattern}` (shortest), `${var##pattern}` (longest)
    
- **Search/replace**: `${var/old/new}` (first), `${var//old/new}` (all)
    
- **Case change**: `${var^^}` (upper), `${var,,}` (lower)
    
- **Order matters**: Brace expansion happens before variable expansion

---
***Previous***: [[Variables and Quoting]]                                            ***Next***: [[Conditionals]]

