Creation date: Thursday, March 5th 2026, 3:02:03 am

**Goal**: Learn to make decisions in your scripts safely. Master exit codes, logical operators, and why double brackets are mandatory for security.

#### **Exit Codes**

> _“Everything in Unix returns an exit code. 0 = success. Non-zero = failure. This is the foundation of all conditionals.”_

Every command returns an exit code when it finishes.

```
ls /etc/passwd
echo "Exit code: $?"
# 0 (success)

ls /nonexistent
echo "Exit code: $?"
# 2 (failure)
```

##### The Rules

- **0** = Success
    
- **Non-zero** = Failure (1-255)

This is how `&&` and `||` work. They check exit codes.

Every command you run returns one - this isn’t optional, it’s fundamental to how Unix works.

##### Setting Exit Codes

```
#!/bin/bash

if [[ ! -f "$1" ]]; then
    echo "File not found: $1"
    exit 1
fi

echo "Processing $1..."
exit 0
```

Always exit with meaningful codes. `exit 0` for success, `exit 1` for general errors. Other scripts will check your exit code.

---
#### **Logical Operators**

##### AND &&

Run the second command only if the first succeeds:

```
mkdir mydir && cd mydir
git pull && echo "Pull successful"
```
##### OR ||

Run the second command only if the first fails:

```
cd mydir || mkdir mydir
command -v docker &>/dev/null || sudo apt install docker
```

##### Chaining

```
cd project || mkdir project && cd project
./run-tests && ./deploy || echo "Tests failed!"
```

`&&` and `||` are your bread and butter. They check exit codes.

This is why exit codes matter - they drive conditional execution.

---
#### **The [[]] Test Command - Security Critical 

> _“Single brackets are a security vulnerability. Double brackets. Always. No exceptions.”_

**Always use** `[[ ]]`, NEVER single brackets `[ ]`:

```
# WRONG - single brackets have many gotchas
if [ $name == "DevOps" ]; then

# CORRECT - double brackets are safer
if [[ "$name" == "DevOps" ]]; then
```

##### Why Single Brackets Are Dangerous

Single brackets `[ ]` are an external binary (`/usr/bin/[`). They don’t prevent word splitting.

```
name="hello world"

# Single brackets - BREAKS!
[ $name == "hello world" ]
# ERROR: too many arguments

# Double brackets - works
[[ $name == "hello world" ]]
# No error
```

> _“I’ve seen production outages caused by single brackets and unquoted variables. This isn’t academic - it’s job security.”_

##### Syntax Rules

```
# Spaces are required around brackets and operators
[[ "$name" == "DevOps" ]]
# CORRECT
[[  "$name"=="DevOps"  ]]
# WRONG - no spaces around ==
```

---
#### **String Comparisons**

```
str1="hello"
str2="world"

[[ "$str1" == "$str2" ]] && echo "Equal"
[[ "$str1" != "$str2" ]] && echo "Not equal"

# Empty/not empty
[[ -z "$name" ]] && echo "Empty"

[[ -n "$name" ]] && echo "Not empty"
```

OperatorMeaning
`==`Equal
`!=`Not equal
`-z`Zero length (empty)
`-n`Non-zero length (not empty)

---
#### **Numeric Comparisons**

For numbers, use different operators:

```
count=5

[[ $count -eq 5 ]] && echo "Equal to 5"
[[ $count -gt 3 ]] && echo "Greater than 3"
[[ $count -lt 10 ]] && echo "Less than 10"
```

OperatorMeaning
`-eq`Equal
`-ne`Not equal
`-gt`Greater than
`-lt`Less than
`-ge`Greater than or equal
`-le`Less than or equal

> _“Use -eq, -lt, -gt for numbers. Use ==, != for strings. Mix them up and you get weird bugs.”_

##### Arithmetic (( )) for Numbers

```
count=5
if (( count > 3 )); then
    echo "Count is greater than 3"
fi
```

Inside `(( ))`: use `<`, `>`, `==`, no `$` needed for variables.

---
#### **File Tests**

```
[[ -f /etc/passwd ]] && echo "File exists"
[[ -d /home ]] && echo "Directory exists"
[[ -x /bin/bash ]] && echo "Executable"
```

TestTrue if…
`-f FILE`File exists and is regular file
`-d FILE`Directory exists
`-e FILE`Exists (file or directory)
`-r FILE`Readable
`-w FILE`Writable
`-x FILE`Executable
`-s FILE`Exists and not empty

File tests are used constantly. `-f` for files, `-d` for directories, `-x` for executables. Know these by heart.

---
#### **If/Else Statements**

```
#!/bin/bash

if [[ -f "$1" ]]; then
    echo "It's a file"
elif [[ -d "$1" ]]; then
    echo "It's a directory"
else
    echo "Not found: $1"
fi
```

##### Combining Conditions

```
# AND - both must be true
if [[ -f "$1" && -r "$1" ]]; then
    echo "File exists and is readable"
fi

# OR - either can be true
if [[ -z "$1" || "$1" == "--help" ]]; then
    echo "Usage: $0 <filename>"
fi

# NOT - invert the condition
if [[ ! -f "$1" ]]; then
    echo "Not a file: $1"
fi
```

---
#### **Checking Command Existence**

**Use** `command -v` NOT `which`:

```
if command -v docker &>/dev/null; then
    echo "Docker is installed"
else
    echo "Docker is not installed"
fi
```

Why `command -v` instead of `which`?

- `command -v` is a bash builtin (faster)
    
- `which` is an external command (may not be installed)
    
- `command -v` is POSIX compliant

##### Check Dependencies Pattern

```
#!/bin/bash

for cmd in git docker kubectl; do
    if ! command -v "$cmd" &>/dev/null; then
        echo "Error: $cmd is not installed"
        exit 1
    fi
done

echo "All dependencies satisfied"
```

This pattern shows up in every production script. Check your dependencies upfront before the script does real work.

---
#### **Case Statements**

When you have more than 2-3 conditions, case is cleaner than nested if/elif.

```
case $1 in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit 1
        ;;
esac
```

- Each pattern ends with `)`
    
- Each block ends with `;;`
    
- `*` is the default (matches anything)
    
- `esac` ends the case
    

Case statements are cleaner for multiple choices. You’ll see this pattern in every init script and CLI tool.

---
***Previous***: [[Parameter Expansion]]





