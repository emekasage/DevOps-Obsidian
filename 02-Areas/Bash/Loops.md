Creation date: Friday, March 6th 2026, 3:02:55 am

**Goal**: Learn to repeat actions efficiently. Master for loops, while loops, and the safe patterns for processing files.

#### **For Loops**

```
for name in Alice Bob Charlie; do
    echo "Hello, $name"
done
```

##### Structure

```
for VARIABLE in LIST; do
    # Commands using $VARIABLE
done
```

##### Looping Over Files

```
for file in *.txt; do
    # Handle case when no files match
    [[ -f "$file" ]] || continue
    echo "Processing: $file"
done
```

Without the check, if no `.txt` files exist, `$file` will literally be `*.txt`. Always guard against this.

##### Number Ranges

```
for i in {1..5}; do
    echo "Count: $i"
done

# With step
for i in {0..10..2}; do
    echo "Even: $i"
done
```

---

#### **Looping Over Command Output**

> _“If your script breaks on filenames with spaces, you wrote a broken script. Not the user’s fault for having spaces.”_

**For files, ALWAYS use globs, NOT command output:**

```
# CORRECT - handles spaces correctly
for file in *; do
    echo "$file"
done

# WRONG - fails with spaces in filenames
for file in $(ls); do
    echo "$file"
done
```

---
#### **While Loops**

```
count=1
while [[ $count -le 5 ]]; do
    echo "Count: $count"
    ((count++))
done
```

Use `for` when you know the list. Use `while` when you don’t - reading files, waiting for conditions.

##### Arithmetic

```

((count++))
((count--))
((count += 5))
```

---
#### **Reading Files Line by Line**

> _“Without -r, backslashes get interpreted. Someone’s filename has a backslash? Script breaks. Always use -r.”_

```
while read -r line; do
    echo "Line: $line"
done < /etc/hosts
```

- `read -r line` reads one line into variable `line`
    
- `-r` prevents backslash interpretation (always use it)
    
- `< file` redirects the file as input
    

Always `read -r`. Without `-r`, backslashes get interpreted. Someone’s data has a backslash? Script breaks.

##### Process Specific Fields

```
# /etc/passwd format: user:x:uid:gid:info:home:shell
while IFS=: read -r user _ uid gid _ home shell; do
    echo "$user (UID $uid) uses $shell"
done < /etc/passwd
```

- `IFS=:` sets field separator to colon
    
- `_` is a throwaway variable for fields we don’t need
    

`IFS` controls word splitting. `IFS=:` to read `/etc/passwd` fields. Understand that `read` splits on IFS. This is powerful.

---
#### **Loop Control**

##### Break - Exit Loop Immediately

```
for i in {1..10}; do
    [[ $i -eq 5 ]] && break
    echo "$i"
done
# Outputs: 1 2 3 4
```

##### Continue - Skip to Next Iteration

```
for i in {1..5}; do
    [[ $i -eq 3 ]] && continue
    echo "$i"
done
# Outputs: 1 2 4 5
```

##### Practical Example

```
# Process files, skip hidden ones
for file in * .*; do
    [[ "$file" == .* ]] && continue
    [[ -f "$file" ]] || continue
    echo "Processing: $file"
done
```

Check file exists in loops - `[[ -f "$file" ]] || continue`. When glob matches nothing, you iterate over the literal pattern. Guard against this.

---
#### **Common Patterns**

##### Batch Rename Files

```
for file in *.txt; do
    [[ -f "$file" ]] || continue
    newname="backup_${file}"
    mv "$file" "$newname"
done
```

##### Process Files Recursively

```
while IFS= read -r -d '' file; do
    echo "Found: $file"
done < <(find . -name "*.txt" -print0)
```

The `-print0` and `-d ''` handle filenames with spaces AND newlines safely. This is the professional pattern.

---
#### **Summary**

- **For loops**: `for item in list; do ... done` - iterate over lists
    
- **While loops**: `while condition; do ... done` - run while true
    
- **Reading files**: `while read -r line; do ... done < file` - line by line
    
- **Break**: Exit loop immediately
    
- **Continue**: Skip to next iteration
    
- **File globs**: Use `*.txt` not `$(ls *.txt)` - handles spaces correctly
    
- **Check file exists**: `[[ -f "$file" ]] || continue` in loops
    
- **Always use** `read -r`: Prevents backslash interpretation

---
#### **Definition**s

**For loop**: Iterates over a list of items.

**While loop**: Repeats while a condition is true.

**Break**: Exits a loop immediately.

**Continue**: Skips to the next loop iteration.

**IFS**: Internal Field Separator - controls word splitting.

---
***Previous***: [[Conditionals]]                                                                    ***Next***: [[Arrays & Functions]]









