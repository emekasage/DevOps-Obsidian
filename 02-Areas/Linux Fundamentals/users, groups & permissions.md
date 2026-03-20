Creation date: Monday, February 9th 2026, 8:35:19 am

#### **Users & Groups**

##### Who Am I?
```
whoami
```
shows who the current user is

##### More User Information

```
id
```

Output:

```
uid=1000(yourusername) gid=1000(yourusername) groups=1000(yourusername),4(adm),27(sudo)
```

This shows:

- `uid` - Your user ID number
    
- `gid` - Your primary group ID
    
- `groups` - All groups you belong to

##### What Groups Am I?

```
groups
```
Being in the `sudo` group means you can use `sudo` to run commands as root.

##### Understanding Users

Every user has:

- A username (like `john`)
    
- A user ID number (like `1000`)
    
- A home directory (like `/home/john`)
    
- A default shell (like `/bin/bash`)
    

User information is stored in `/etc/passwd`:

```
cat /etc/passwd
```
prints out the user information

Each line represents a user:

```
yourusername:x:1000:1000:Your Name:/home/yourusername:/bin/bash
```

Format: `username:password:uid:gid:comment:home:shell`

The `x` means the password is stored elsewhere (in `/etc/shadow`).

##### Understanding Groups

Groups allow multiple users to share access to files.

```
cat /etc/group
```

Each line:

```
sudo:x:27:yourusername
```
Format: `groupname:password:gid:members`

Common groups:

- `sudo` - Can use sudo command
    
- `adm` - Can read log files
    
- `www-data` - Web server group
    
- `docker` - Can use Docker

##### Creating a User

```
sudo adduser testuser
```

You’ll be prompted to set a password and optional info (you can press Enter to skip the optional fields).

This creates:

- A new user account
    
- A home directory `/home/testuser`
    
- A group with the same name

Verify:

```
cat /etc/passwd | grep testuser
```

Output:

```
testuser:x:1001:1001::/home/testuser:/bin/bash
```

##### Deleting a User

```
sudo deluser testuser
```

To also remove their home directory:

```
sudo deluser --remove-home testuser
```

---
#### **File Permissions**

##### Reading ls -l Output

```
ls -l /etc/passwd
```

Output:

```
-rw-r--r-- 1 root root 1842 Dec 10 14:30 /etc/passwd
```

Let’s break this down:

```
-rw-r--r--  1  root  root  1842  Dec 10 14:30  /etc/passwd
│├──┤├──┤├──┤ │  │     │     │         │           │
│ │   │   │   │  │     │     │         │           └── Filename
│ │   │   │   │  │     │     │         └── Modification date
│ │   │   │   │  │     │     └── Size in bytes
│ │   │   │   │  │     └── Group owner
│ │   │   │   │  └── User owner
│ │   │   │   └── Number of hard links
│ │   │   └── Others permissions
│ │   └── Group permissions
│ └── Owner permissions
└── File type (- = file, d = directory, l = link)
```

##### The Permission Groups
Every file has three permission sets:

Position.                   Who                       Description
- Characters  2-4: Owner - The user who owns the file
- Characters 5-7: Group Users in the file’s group
- Characters 8-10: Others Everyone else

##### The Permission Types

LetterPermissionOn FilesOn Directories
`r` - Read: Can view contents, can list contents
`w` - Write: Can modify contents, can add/delete files
`x` - Execute: Can run as program, can enter (cd into)
`-` None Permission: Permission denied

##### Examples

```
-rw-r--r--   Regular file, owner can read/write, others can only read
-rwxr-xr-x   Executable, owner full access, others can read/execute
drwxr-xr-x   Directory, owner full access, others can read/enter
-rw-------   File only owner can read/write (private)
```

##### Changing Permissions: chmod

**Symbolic mode:**

```
chmod u+x script.sh    # Add execute for user (owner)
chmod g+w file.txt     # Add write for group
chmod o-r file.txt     # Remove read for others
chmod a+r file.txt     # Add read for all (a = all)
chmod u+rwx file.txt   # Add read, write, execute for user
```
Symbols:

- `u` = user (owner)
    
- `g` = group
    
- `o` = others
    
- `a` = all
    
- `+` = add permission
    
- `-` = remove permission
    
- `=` = set exactly
    

**Numeric mode:**

Each permission has a value:

- `r` = 4
    
- `w` = 2
    
- `x` = 1
    

Add them up for each group:

NumberPermissionsMeaning
- 7: rwx- = Read + Write + Execute
- 6: rw- = Read + Write
- 5: r-x = Read + Execute
- 4r– = Read only
- 3: -wx = Write + Execute
- 2: -w- = Write only
- 1: –x = Execute only
- 0—No permissions

Common combinations:

```
chmod 755 script.sh    # rwxr-xr-x (executable by all)
chmod 644 file.txt     # rw-r--r-- (readable by all)
chmod 600 secret.txt   # rw------- (private file)
chmod 700 private_dir  # rwx------ (private directory)
```

##### Changing Ownership: chown

```
sudo chown newowner file.txt
```

Change owner and group:

```
sudo chown newowner:newgroup file.txt
```

Change just the group:

```
sudo chgrp newgroup file.txt
```

Recursive (for directories):

```
sudo chown -R user:group directory/
```

##### Permissions in Action

Let’s prove permissions actually work. First, create a test user:

```
sudo adduser testuser
```

We’ll use `/tmp` for this test. Your home directory might block other users from even entering it, which would confuse our test. `/tmp` is world-accessible, so we’re testing only the file permissions.

Create a private file in `/tmp`:

```
echo "This is my secret" > /tmp/secret.txt
chmod 600 /tmp/secret.txt
ls -l /tmp/secret.txt
```

Output:

```
-rw------- 1 yourusername yourusername 18 Dec 12 10:00 /tmp/secret.txt
```

The `600` means only the owner can read/write. No group access, no others access.

Now try to read it as the test user:

```
sudo -u testuser cat /tmp/secret.txt
```

Output:

```
cat: /tmp/secret.txt: Permission denied
```

It works. The test user cannot read your private file.

Now make it readable by others:

```
chmod 604 /tmp/secret.txt
sudo -u testuser cat /tmp/secret.txt
```

Output:

```
This is my secret
```

Now they can read it. That’s permissions in action.

Clean up:

```
rm /tmp/secret.txt
sudo deluser --remove-home testuser
```

---
#### **Sudo & Root**

##### The Root User

Root (also called superuser) has UID 0 and can do anything:

- Read any file
    
- Modify any file
    
- Kill any process
    
- Change any setting
    

Root ignores all permission checks. That’s why it’s dangerous.

##### Why Not Just Be Root?

- One typo can destroy your system
    
- Malware would have full access
    
- No protection against mistakes
    
- It’s against best practices

Instead, we use `sudo` for temporary elevation.

##### Using Sudo

```
sudo command
```

This runs one command as root. You’ll be asked for YOUR password (not root’s password).

```
sudo apt update        # Update packages (needs root)
sudo vi /etc/hosts     # Edit system file (needs root)
sudo cat /etc/shadow   # View passwords file (needs root)
```
##### Sudo Timeout

After entering your password, sudo remembers for a few minutes. You won’t be asked again immediately.

To force sudo to forget:

```
sudo -k
```
##### Running a Root Shell

```
sudo -i
```

This gives you a root shell. Your prompt changes from `$` to `#`.

Type `exit` to return to your normal user.

> **Warning**: Be extremely careful in a root shell. Don’t leave it open.

##### Who Can Use Sudo?

Users in the `sudo` group can use sudo. Check:

```
groups
```

If you see `sudo` in the list, you can use it.

The rules are defined in `/etc/sudoers` (don’t edit directly - use `visudo`).

##### Common Permission Errors

**“Permission denied”** - You need to:

1. Use `sudo` for the command, or
    
2. Change the file permissions, or
    
3. You’re trying to access something you shouldn’t
    

**“Operation not permitted”** - Even sudo can’t do it (immutable file, mounted read-only, etc.)

---
#### **Summary**

- Every file has an owner (user), group, and permissions
    
- `adduser` creates users, `deluser` removes them
    
- Permissions: read ®, write (w), execute (x)
    
- Three sets: owner, group, others
    
- `chmod 600` = private, `chmod 644` = readable by all
    
- `sudo -u user` runs commands as another user
    
- Root can do anything - use `sudo` instead of being root
---
#### **Definitions**

**User**: An account that can log in and own files.

**Group**: A collection of users that can share file access.

**UID**: User ID - a number identifying a user (root is 0).

**GID**: Group ID - a number identifying a group.

**Permissions**: Rules controlling who can read, write, or execute a file.

**Owner**: The user who owns a file (can change its permissions).

**Root**: The superuser with UID 0, having full system access.

**Sudo**: “Superuser do” - temporarily elevate privileges for one command.

**chmod**: Change mode - modify file permissions.

**chown**: Change owner - modify file ownership

---
***Previous***: [[viewing & manipulating files]]                                                                  ***Next***: [[vi(m)]]