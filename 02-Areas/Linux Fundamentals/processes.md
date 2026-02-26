Creation date: Monday, February 16th 2026, 7:24:30 am

#### **Process Management**

##### What is a process?

A process is a running instance of a program. When you run `ls`, a process starts, does its work, and exits. Long-running programs like web servers stay running as processes.

Every process has:

- A **PID** (Process ID) - unique number
    
- A **parent process** (PPID) - the process that started it
    
- An **owner** - the user running it
    
- A **state** - running, sleeping, stopped, etc.

##### List Your Processes

```
ps
```
Shows processes attached to your terminal

##### List ALL Processes

```
ps aux
```

The flags:
- `a` - All users
- `u` - User-oriented format
- `x` - include processes without a terminal

##### Search for Processes

```
ps aux | grep cron
```
Find processes with "cron" in the name.

```
pgrep cron
```
Returns just the PIDs.
##### Interactive Process Viewer: top
```
top
```

Real-time view of processes. Updates continuously.

KeyAction
`q` - Quit
`k` - Kill a process (enter PID)
`M` - Sort by memory
`P` - Sort by CPU
`h` - Help
##### Better Process Viewer: htop

```
htop
```

- Arrow keys to navigate
    
- F9 to kill a process
    
- F6 to sort
    
- F10 or `q` to quit
##### Stop a Stuck Process

Press `Ctrl+C` in the terminal running the process. This sends an interrupt signal.

##### Kill a Process by PID

```
kill 1234
```
Sends SIGTERM (polite request to stop). The process can clean up.

```
kill -9 1234
```
Sends SIGKILL (force kill). Use when the process ignores SIGTERM.

##### Kill by Name

```
killall firefox
```
Kills all processes named “firefox”.

```
pkill -f "python script.py"
```
Kills processes matching the pattern.

##### Background Processes

Run a command in the background:

```
sleep 60 &

```

The `&` starts it in the background. You get your prompt back immediately.

##### List Background Jobs

```
jobs
```

##### Bring to Foreground

```
fg %1
```
Brings job 1 to the foreground. Now `Ctrl+C` will stop it.

##### Send to Background

If a process is running in the foreground:

1. Press `Ctrl+Z` to suspend it
    
2. Type `bg` to continue it in the background

---
#### **System Information**

##### Basic System Info
```
uname -a 
```
Shows: kernel name, hostname, kernel version, architecture

##### System Uptime
```
uptime
```
Shows how long the system has been running and load averages.

##### Memory Usage
```
free -h
```

Output:

```
            total        used        free      shared  buff/cache   available
Mem:        1.9Gi       512Mi       256Mi       8.0Mi       1.2Gi       1.3Gi
Swap:       2.0Gi          0B       2.0Gi
```

The `-h` flag gives human-readable sizes.

Key numbers:

- **available** - Memory ready for use (more relevant than “free”)
    
- **buff/cache** - Memory used for caching (can be freed if needed)

##### CPU Information

```
lscpu
```
Shows CPU architecture, cores, threads, speed.

```
lscpu | grep -E "^(CPU\(s\) |Model name)"
```
Displays just the essentials: no. of CPUs, Model name

##### Disk Information

```
lsblk
```
Shows block devices (disks) and partitions

##### Disk Space

```
df -h
```
Shows mounted filesystems and space usage (covered in [[viewing & manipulating files]])

---
#### **Service Management (systemd)**

##### What is systemd?

Systemd is the init system on modern Linux. It:

- Starts services at boot
    
- Manages running services
    
- Handles dependencies between services
    
- Collects logs
##### Check Service Status

```
systemctl status ssh
```
Shows if running, when it started, resource usage.

##### Start, Stop, Restart Services

```
sudo systemctl stop cron        # Stop a service (the cron service in this case)
sudo systemctl start cron       # Start a service
sudo systemctl restart cron     # Restart a service
```

##### Enable/Disable at Boot

```
sudo systemctl enable cron      # Start automatically at boot
sudo systemctl disable cron     # Don't start at boot
```

Most services are enabled by default. Use `systemctl is-enabled cron` to check.

##### List All Services

```
systemctl list-units --type=service
```

Add `--state=running` to see only running services.

##### View Logs with journalctl

```
journalctl
```

Shows all system logs. Press `q` to quit.

```
journalctl -u ssh
```

Logs for a specific service.

```
journalctl -f
```

Follow logs in real-time (like `tail -f`).

```
journalctl --since "1 hour ago"
```

Logs from the last hour.

```
journalctl -b
```

Logs since last boot.

---
***Previous***: [[i-o & pipes]]                                               ***Next***: [[networking and ssh]]
