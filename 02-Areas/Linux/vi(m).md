Creation date: Wednesday, February 11th 2026, 1:52:11 am

#### **Why Vi?**

Vi is installed on:

- Every Linux distribution
    
- Every Unix system
    
- macOS
    
- Minimal Docker containers
    
- Embedded systems and routers

When you SSH into a server at 3 AM to fix a production issue, vi will be there. VS Code won’t be.

---
#### **Vi vs Vim**

On Ubuntu, `vi` is actually `vim` in disguise:

```
ls -la /usr/bin/vi
```

Output:

```
lrwxrwxrwx 1 root root 20 ... /usr/bin/vi -> /etc/alternatives/vi
```

Follow it further:

```
ls -la /etc/alternatives/vi
```

Output:

```
lrwxrwxrwx 1 root root 18 ... /etc/alternatives/vi -> /usr/bin/vim.basic
```

So on Ubuntu, you’re getting vim’s features. But in production containers, you won’t.

##### BusyBox

**BusyBox** is a single executable that provides stripped-down versions of many Unix utilities. It’s used in:

- Docker containers (especially Alpine-based)
    
- Embedded systems (routers, IoT devices)
    
- Rescue disks

Install it to experience true minimal vi:

```
sudo apt install busybox
```

Try BusyBox vi:

```
busybox vi test.txt
```

Notice the differences:

- No `-- INSERT --` indicator
    
- No syntax highlighting
    
- Single undo only

This is what you’ll encounter in production containers.

---
#### **First Edit**

```
busybox vi test.txt
```

1. Press `i` to enter Insert mode
    
2. Type: `Hello, this is my first vi edit.`
    
3. Press `Esc` to return to Normal mode
    
4. Type `:wq` and press Enter

---
#### **The Minimum You Need**

1. `i` - Start typing
    
2. `Esc` - Stop typing
    
3. `:wq` - Save and quit
    
4. `:q!` - Quit without saving
    
5. `dd` - Delete a line
    
6. `u` - Undo

With just these six commands, you can edit any file anywhere.

---
***Previous***: [[users, groups & permissions]]                             ***Next***: [[i-o & pipes]] 











