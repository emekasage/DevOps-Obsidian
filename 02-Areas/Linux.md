#### What is Linux?
When your VM is running, there's a program called the *kernel* that's in charge of everything. The kernel
- Controls the hardware (CPU, memory, disk, network)
- Manages files
- Runs programs
- Keep tracks of users and permission

The Linus kernel *is* Linux. Everything else is just programs running on top of it.

***You can't talk to the kernel directly***

The kernel doesn't understand basic human instructions e.g "run this program". It speaks a different language - low level instructions called *system calls* that only programs know how to use.

So there's a gap between the kernel and YOU the user.

To bridge the gap, we use a ***Shell***. It is a program that bridges that gap

### Common Shells
- Bash - Bourne Again Shell. the default on most Linux systems
- Zsh - Z shell. the default on macOS
- sh - The original Bourne Shell
- fish - Friendly Interactive Shell

To check which shell your're using
```
echo $SHELL

output: 
/bin/bash
```

The *Terminal* is the window - the application that displays text and accepts keyboard input e.g. WezTerm, Windows Terminal, macOS Terminal.app, iTerm2, The VMware console window.

##### Shell vs Terminal 
The ***Shell*** is the program running inside the terminal that interprets your commands. The terminal is the window. The Shell is what's running in that window.

**CLI** stands for Command Line Interface.

#### The Prompt
When you see this: 
```
username@hostname: ~$
```
This is your prompt. It tells you:
- username - who you're logged in as
- hostname - what computer you're on
- ~ - where you are (your home directory)
- $ - You're a normal user

#### Why Linux for DevOps?
- **Servers**: The vast majority of servers run Linux
- **Containers**: Docker containers are Linux
- **Cloud**: AWS, GCP, Azure
- **Automation**: Linux is built for scripting and automation
- **Free & Stable**: No licensing costs + Linux servers run for years without issues

#### What's a Distribution?
Linux is just the kernel. A ***distribution*** packages the kernel with additional software to make a complete OS.

Popular distributions:
- **Ubuntu**: Beginner-friendly (what I'm currently using)
- **Debian**: Ubuntu is based on this, very stable
- **Red Hat / Rocky / AlmaLinux**: Enterprise-focused
- **Fedora**: Red Hat based
- **Arch**: For advanced users who want full control
- **Alpine**: minimal, popular in containers
