Creation date: Saturday, February 7th 2026, 7:58:36 am

***Everything is represented as a file in Linux***

#### The Root Directory

This is the top of the file system and it's called "root" (not to be confused with the root user)
```
ls /
```

#### Absolute vs Relative Paths

**Absolute path**: starts from (`/`), always works regardless of where you are
```
/home/username/documents/file.txt
```

**Relative path**: starts from your current location
```
documents/file.txt (if you're in /home/username)
```

Special shortcuts:

- `.` = current directory
    
- `..` = parent directory (one level up)
    
- `~` = your home directory

#### Show Working Directory
```
pwd - print current/working directory
```

#### Change Directory
```
cd /var/log
```

#### Go Home
```
cd OR cd ~
```

#### Go Up One Level
```
cd ..
```

#### Go to Previous Directory
```
cd -
```

#### List Directory Contents
```
ls
```

Basic listing.

```
ls -l
```

Long format with details:

```
-rw-r--r-- 1 user user 1234 Dec 10 14:30 myfile.txt
drwxr-xr-x 2 user user 4096 Dec 10 14:30 mydir
```

The first character tells you the type:

- `-` = regular file
    
- `d` = directory
    
- `l` = symbolic link
    

```
ls -a
```

Show **a**ll files, including hidden ones (files starting with `.`).

```
ls -la
```

Combine them: long format + hidden files. This is the most useful form.

```
ls -lah
```

Add **h**uman-readable file sizes (1K, 234M, 2G instead of bytes).

#### Visual Directory Structure

```
tree
```

Shows the directory as a tree. Very visual.

```
tree -L 2
```

Limit to 2 levels deep.

```
tree /etc/apt
```

Tree a specific directory.

------
#### Create a Directory

```
mkdir projects
```
Creates a directory called `projects`

```
mkdir -p projects/work/2025
```

Creates nested directories. The `-p` flag creates parent directories as needed.

#### Create an Empty File

```
touch notes.txt
```

Creates an empty file. If the file exists, it updates its timestamp.

#### Copy Files

```
cp notes.txt notes-backup.txt
```

Copies `notes.txt` to `notes-backup.txt`.

```
cp notes.txt projects/
```

Copies `notes.txt` into the `projects` directory.

```
cp -r projects projects-backup
```

Copies a directory. The `-r` flag means **r**ecursive - copy the directory and everything in it.

#### Move and Rename Files

```
mv notes.txt documents/
```

Moves `notes.txt` into `documents/`.

```
mv notes.txt memo.txt
```

Renames `notes.txt` to `memo.txt`. Moving and renaming are the same operation.

```
mv -i important.txt somewhere/
```

The `-i` flag prompts before overwriting existing files.

#### Delete Files

```
rm notes.txt
```

Deletes `notes.txt`. **No confirmation. No trash can. Gone.**

```
rm -i notes.txt
```

Prompts for confirmation before deleting.

```
rm -r projects/
```

Deletes a directory and everything in it recursively.

### **The Dangerous Command**

```
rm -rf /
```

**NEVER RUN THIS.** It deletes everything on the system. The `-f` flag means force (no prompts). This is mentioned only so you understand the danger.

>**Warning**: Be extremely careful with `rm -rf`. Double-check your path. There’s no undo.

#### Remove Empty Directories

```
rmdir empty-folder
```

Only works if the directory is empty. Safer than `rm -r`.

-----------------------------------
#### Find Files by Name

```
find /home -name "*.txt"
```

Find all `.txt` files under `/home`.

```
find . -name "config"
```

Find files named “config” starting from current directory (`.`).

```
find /etc -name "*.conf"
```

Find all `.conf` files in `/etc`.

#### Find by Type

```
find /var -type d -name "log"
```

Find directories (`-type d`) named “log”.

```
find ~ -type f -name "*.sh"
```

Find files (`-type f`) ending in `.sh`.

#### Find by Size

```
find /var/log -size +10M
```

Find files larger than 10 megabytes.

#### Locate (Faster but Less Current)*

```
locate filename
```

Much faster than `find` because it uses a database. But the database might be outdated.

Update the database:

```
sudo updatedb
```

> **Note**: `locate` may not be installed by default. Install with `sudo apt install mlocate`.

#### Which: Find Command Location

```
which ls
```

Shows where a command’s executable is located.

#### Command -v: The Portable Way

```
command -v mkdir
```

Similar to `which`, but more reliable. This is the POSIX standard way to find commands, and it works in scripts where `which` might not.

**Why use** `command -v` over `which`?

- `which` is an external program that behaves differently on different systems
    
- `command -v` is built into the shell and works the same everywhere
    
- Scripts should use `command -v` for portability
#### Type: What Kind of Command

```
type mkdir
```

Output:

```
ls is aliased to `ls --color=auto'
```

```
type cd
```

Output:

```
cd is a shell builtin
```
`type` tells you if something is an alias, builtin, or external command. More informative than `which`.

---
#### The man Command

```
man ls
```
Opens the **man**ual page for `ls`. This is the official documentation.

**Navigation inside man:**

- `Space` or `f` = next page
    
- `b` = previous page
    
- `/pattern` = search for text
    
- `n` = next search result
    
- `q` = quit
#### Man Page Sections

Man pages are organized into numbered sections:

SectionContents1User commands (what you run in the terminal)2System calls (kernel functions)3Library functions (C programming)4Device files5File formats and config files6Games7Miscellaneous (conventions, overviews)8System administration commands

Sometimes a name exists in multiple sections. For example, `passwd` is both a command (1) and a file format (5):

```
man passwd       # Shows section 1 (the command)
man 5 passwd     # Shows section 5 (the file format)
```
#### Searching Man Pages

Don’t know the exact command name?

```
man -k copy
```
Searches man page descriptions for “copy”. Returns a list of relevant commands.

```
man -k "file system"
```
Use quotes for multi-word searches.

#### The help Command

Some commands are **shell builtins** - they’re part of bash itself, not separate programs. These don’t have man pages. Use `help` instead:

```
help cd
```
Shows help for the `cd` builtin.

```
help type
```
Shows help for the `type` builtin.

#### The --help Flag

Most commands support a `--help` flag for quick reference:

```
ls --help
```

Shorter than the man page. Good for a quick reminder of options.

```
mkdir --help
```

This works for almost every command.

---
#### **When to Use Which**

| Situation                            | Use              |
| ------------------------------------ | ---------------- |
| Full documentation for a command     | `man command`    |
| Shell builtin (cd, type, help, etc.) | `help command`   |
| Quick reminder of options            | `command --help` |
| Don’t know the command name          | `man -k keyword` |

---
#### **Practice**

```
man find          # The find command has many options
help read         # read is a bash builtin
grep --help       # Quick overview of grep options
man -k permission # Find commands related to permissions
```

---
#### **Commands Used**

| Commands                    | Description                  |
| --------------------------- | ---------------------------- |
| `pwd`                       | Print working directory      |
| `cd directory`              | Change to directory          |
| `cd` or `cd ~`              | Go to home directory         |
| `cd ..`                     | Go up one level              |
| `cd -`                      | Go to previous directory     |
| `ls`                        | List directory contents      |
| `tree`                      | Visual directory tree        |
| `tree -L n`                 | Limit tree depth to n levels |
| `mkdir name`                | Create a directory           |
| `touch file`                | Create empty file            |
| `cp src dest`               | Copy file                    |
| `cp -r src dest`            | Copy directory recursively   |
| `mv src dest`               | Move or rename file          |
| `rm file`                   | Delete file                  |
| `rm -r dir`                 | Delete directory recursively |
| `rm -i file`                | Delete with confirmation     |
| `rmdir dir`                 | Remove empty directory       |
| `find path -name "pattern"` | Find files by name           |
| `locate name`               | Fast search using database   |
| `man command`               | View manual page             |
| `man -k keyword`            | Search man pages             |
| `man N command`             | View specific section        |
| `command --help`            | Quick command help           |

---
#### **Summary**

- Linux has a single directory tree starting at `/`
    
- Key directories: `/home` (your files), `/etc` (config), `/var` (logs)
    
- Use `pwd` to see where you are, `cd` to move, `ls` to list
    
- `mkdir`, `touch`, `cp`, `mv`, `rm` for file operations
    
- `find` searches the file system, `locate` uses a database
    
- `type` is better than `which` for understanding commands
    
- Use `man` for documentation, `help` for shell builtins, `--help` for quick reference
---
#### **Definitions**

**Root Directory**: The top of the file system hierarchy, represented by `/`.

**Absolute Path**: A path starting from root (`/home/user/file.txt`).

**Relative Path**: A path relative to current location (`./file.txt` or `../other/`).

**Working Directory**: The directory you’re currently in.

**Hidden File**: A file whose name starts with `.` - not shown by default in `ls`.

**Recursive**: Operating on a directory and all its contents, including subdirectories.

**Home Directory**: Your personal directory, usually `/home/username`, abbreviated as `~`.

**Man Page**: The manual page for a command, accessed via `man`. The primary documentation source on Unix/Linux systems.

**Shell Builtin**: A command built directly into the shell (like `cd`, `type`, `help`) rather than an external program.

---
***Previous***: [[installing software]]                            ***Next***: [[viewing & manipulating files]]






