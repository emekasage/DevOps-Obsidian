Creation date: Sunday, March 1st 2026, 7:06:14 am

#### **What is Bash**

```
echo $SHELL
bash --version
```

- **Terminal**: The window you type in
    
- **Shell**: A program that interprets your commands
    
- **Bash**: Bourne Again Shell - the standard Linux shell since 1995-96

> _“Bash isn’t just for running commands - it’s a full programming language.”_

Bash has variables, conditionals, loops, and functions. It’s not just a command runner - it’s a scripting language that’s on every Linux server.

**Why bash over zsh/fish?** Bash is the standard. When you SSH into a server, it’s running bash. Write portable scripts that work everywhere.

**Why write scripts?** Instead of typing commands one by one, write them once and run whenever needed. Automation, repeatability, documentation.

---
#### **Creating First Script**

```
vim hello
```

```
#!/bin/bash

echo "Hello, DevOps!"
```

##### The Shebang - Security Critical

The first line `#!/bin/bash` tells the system which interpreter to use.

**Use** `#!/bin/bash`, NOT `#!/usr/bin/env bash`

Why? `#!/usr/bin/env bash` searches your PATH for bash. An attacker can put a malicious “bash” earlier in your PATH. With `#!/bin/bash` you know exactly what runs.

##### No File Extensions

Name scripts without extensions (`hello`, not `hello.sh`). The shebang tells the system what interpreter to use.

The `.sh` extension is a convention for humans, not a requirement. Real Unix tools don’t have extensions - `ls`, `grep`, `vim`. Your scripts shouldn’t either.

---
#### **Running Scripts**

**Method 1: Explicit interpreter**

```
bash hello
```

**Method 2: Make executable**

```
chmod +x hello
./hello
```

**Why** `./`? The current directory isn’t in PATH (security feature). Never add `.` to your PATH.

> _“Adding . to your PATH is how you get hacked. Someone drops a malicious ‘ls’ in a directory you visit, game over.”_

##### Debug Mode

When scripts misbehave, run with `-x` to see each command as it executes:

```
bash -x hello
```

This shows variable expansion and command execution - invaluable for debugging.

Before asking for help, run your script with `bash -x` first. You’ll often spot the problem immediately.

---
#### **Shellcheck - Mandatory

Shellcheck finds bugs and security issues in shell scripts.

```
# Install
sudo apt install shellcheck    # Debian/Ubuntu
sudo dnf install ShellCheck    # Fedora

# Use
shellcheck hello
```

Run shellcheck on EVERY script before committing or deploying. Not optional. Not “nice to have.” Mandatory.

---
#### **Practical Example**

```
#!/bin/bash

# Description: Display system information

echo "=== System Information ==="
echo "Hostname: $(hostname)"
echo "Date: $(date)"
echo "Kernel: $(uname -r)"
```

```
shellcheck sysinfo
chmod +x sysinfo
./sysinfo
```

Always add a description comment. Six months from now, you won’t remember what the script does.

---
#### **Commands Used**

- `chmod +x hello` Make file executable
- `./script` Run script in current directory
- `bash script`Run script with bash explicitly
- `bash -x script`Run with debug output
- `shellcheck script`Validate script

---
#### **Your Dotfiles Directory**

Build a dotfiles repository - a collection of your personal configuration files that you can take anywhere.

##### Step 1: Create Your Dotfiles Directory

```
mkdir -p ~/dotfiles
```

##### Step 2: Move Your Existing Configs (if they exist)

```
# Move existing files into dotfiles (if they exist)
mv ~/.bashrc ~/dotfiles/bashrc
mv ~/.vimrc ~/dotfiles/vimrc
mv ~/.config/starship.toml ~/dotfiles/starship.toml
```

##### Step 3: Write the Setup Script

Create `~/dotfiles/setup`:

```
#!/bin/bash

# Description: Link dotfiles to home directory

mkdir -p ~/.config

ln -sf ~/dotfiles/bashrc ~/.bashrc
ln -sf ~/dotfiles/vimrc ~/.vimrc
ln -sf ~/dotfiles/starship.toml ~/.config/starship.toml

echo "Dotfiles linked!"
```

Verify with shellcheck!

Let’s run it:

`chmod +x ~/dotfiles/setup && ./setup`

The `-sf` flags mean: `-s` creates a symbolic link, `-f` forces (overwrites existing).

Now your configs live in `~/dotfiles/` and symlinks point to them. You can version control this directory with Git (covered in the Git course) and use the same configs on any machine.





