Creation date: Tuesday, February 24th 2026, 2:42:21 am

**Goal**: Transform your terminal from functional to professional with custom configuration files.

Your terminal is your workspace. A good setup makes you more productive and comfortable. Let’s configure a professional environment.

---
#### **Understanding Dotfiles**

##### What is a Dotfile?

Files starting with `.` are hidden by default. Many are configuration files:

```
ls -la ~
```

Output includes:

```
.bashrc
.profile
.vimrc
.ssh/
.config/
```
These “dotfiles” configure your shell, editor, and tools.

##### .profile vs .bashrc

FileWhen It RunsPurpose`.profile`Login shellsEnvironment variables, PATH`.bashrc`Interactive non-login shellsAliases, prompt, shell behavior

- **Login shell**: When you SSH in or log in at console. 
- **Non-login shell**: When you open a new terminal window.

In practice, most people put everything in `.bashrc` and source it from `.profile`.

##### View Your Current .bashrc

```
cat ~/.bashrc
```
You’ll see default Ubuntu configuration - aliases, prompt settings, etc.

---
#### **Starship Prompt**

##### Why Starship?

The default prompt shows username, hostname, and directory:

```
sagecode@caspian:~$
```

Starship shows much more:

- Current directory (shortened intelligently)
    
- Git branch and status
    
- Programming language versions
    
- Command execution time
    
- Error status of last command
    

And it’s fast, written in Rust.

##### Install Starship

```
curl -sS https://starship.rs/install.sh | sh
```
Press `y` when prompted.

##### Enable Starship

Add to the end of your `.bashrc`:

```
vi ~/.bashrc
```

Go to end of file (`G`), open new line (`o`), and add:

```
eval "$(starship init bash)"
```

Save and quit (`:wq`).

##### Apply Changes**

```
source ~/.bashrc
```

Your prompt changes immediately.

##### Configure Starship

Starship reads `~/.config/starship.toml`:

```
mkdir -p ~/.config
vi ~/.config/starship.toml
```

A minimal configuration:

```
# Timeout for commands (milliseconds)
command_timeout = 1000

# Get editor completions based on the config schema
"$schema" = 'https://starship.rs/config-schema.json'

# Inserts a blank line between shell prompts
add_newline = true

# Replace the '❯' symbol in the prompt with '➜'
[character] # The name of the module we are configuring is 'character'
success_symbol = '[➜](bold green)' # The 'success_symbol' segment is being set to '➜' with the color 'bold green'

# Disable the package module, hiding it from the prompt completely
[package]
disabled = true


[memory_usage]
disabled = false
threshold = -1
symbol = ' '
style = 'bold dimmed green'

```

After saving, the prompt updates automatically.

---
#### **Vim Configuration**

##### The .vimrc File

Vim reads `~/.vimrc` on startup. This file customizes vim’s behavior.

##### Copy the default

```
cp /etc/vim/vimrc ~/.vimrc
```

Uncomment what you like.

Save and quit (`:wq`).

##### Test Your Configuration

```
vi ~/.bashrc
```

---
#### **Tmux Configuration**

##### The .tmux.conf File

Tmux reads `~/.tmux.conf` on startup.

```
vi ~/.tmux.conf
```

##### Useful Configuration

```

# Start window numbering at 1 (not 0)
set -g base-index 1
setw -g pane-base-index 1

# Enable mouse support
set -g mouse on

# Increase history limit
set -g history-limit 10000

# 256 color support
set -g default-terminal "tmux-256color"
set-option -sa terminal-overrides ',xterm-256color:RGB'

```

Save and quit.

---
#### **Summary**

- Dotfiles (files starting with `.`) configure your environment
    
- `.bashrc` runs for interactive shells, `.profile` for login shells
    
- Starship gives you a modern, informative prompt with git status
    
- `.vimrc` makes vim much more usable with syntax highlighting and line numbers
    
- `.tmux.conf` customizes tmux (consider changing prefix to `Ctrl+a`)
    
- Back up your dotfiles in a git repository for portability

---
#### **Definitions**

**Dotfile**: A configuration file whose name starts with `.` (hidden by default).

**Shell Configuration**: Files that customize shell behavior (`.bashrc`, `.profile`).

**Starship**: A fast, customizable prompt written in Rust.

**TOML**: Tom’s Obvious Minimal Language - a configuration file format.

**.vimrc**: Vim’s configuration file, read at startup.

**Source**: Running a script in the current shell (`source file` or `. file`).

---
***Previous***: [[Tmux]]           

