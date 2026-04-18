Creation date: Sunday, February 22nd 2026, 5:37:20 pm

##### Why Tmux (Terminal Multiplexer)?

**The Problem**

Imagine you’re running a long process over SSH. Your connection drops.

Without tmux, the process dies. You have to start over.

Or you’re working on a server and need to:

- Watch logs in one terminal
    
- Edit files in another
    
- Run commands in a third

Without tmux, you’d need three SSH connections.

**The Solution**

Tmux solves both problems:

- **Sessions persist** - Disconnect and reconnect without losing work
    
- **Multiple panes** - Split your terminal into sections
    
- **Multiple windows** - Like tabs in a browser

---

#### **Getting Started**
##### Install Tmux

```
sudo apt install tmux
```

##### Start Tmux

```
tmux
```
You’re now inside a tmux session. Notice the green bar at the bottom - that’s the status line.

##### The Prefix Key

Tmux commands start with the **prefix**: `Ctrl+b`

You press `Ctrl+b`, release both keys, then press the command key.

For example, to detach: `Ctrl+b` then `d`

---
#### **Sessions**

##### Detach from Session

```
Ctrl+b d
```
You’re back at your normal shell, but the tmux session is still running in the background.

##### List Sessions

```
tmux ls
```

##### Attach to Session

```
tmux attach
```
Attaches to the most recent session.

```
tmux attach -t 0
```
Attaches to session 0 specifically.

##### Named Sessions

Create a session with a name:

```
tmux new -s work
```

Attach by name:

```
tmux attach -t work
```
This is clearer than remembering session numbers.

##### Kill a Session

```
tmux kill-session -t work
```
Or from inside tmux, just type `exit` in all panes.

---
#### **Windows**

Windows are like tabs. Each window is a full terminal.

ActionKeysCreate 
- new window`Ctrl+b c`
- Next window`Ctrl+b n`
- Previous window`Ctrl+b p`
- Window by number`Ctrl+b 0-9`
- Rename window`Ctrl+b ,`
- Close windowType `exit` or `Ctrl+b &`

The status bar shows your windows: `0:bash 1:vim 2:logs`

---
#### **Panes**

Panes split a window into sections. Each pane is a separate terminal.

ActionKeys
- Split horizontal`Ctrl+b "`
- Split vertical`Ctrl+b %`
- Move between panes`Ctrl+b arrow`
- Close paneType `exit` or `Ctrl+b x`
- Toggle full-screen`Ctrl+b z`
- Resize pane`Ctrl+b Ctrl+arrow`
##### Split Vertically:

```
Ctrl+b %
```
Now you have two panes side by side. Move between them with `Ctrl+b left` and `Ctrl+b right`.

##### Split one pane horizontally:

```
Ctrl+b "
```
Now you have three panes. Navigate with `Ctrl+b arrow`.

---
#### **Typical Workflow**

Here’s how you might use tmux daily:

1. SSH to server
    
2. Start tmux: `tmux new -s project`
    
3. Split into panes for different tasks
    
4. Work on your task
    
5. Detach when done: `Ctrl+b d`
    
6. Disconnect SSH
    
7. Later: SSH back, `tmux attach -t project`
    
8. Everything is exactly where you left it

##### Example Setup

```
# Start named session
tmux new -s dev

# Split vertically (editor on left, terminal on right)
Ctrl+b %

# In left pane, open editor
vi myfile.py

# Move to right pane
Ctrl+b right

# Split right pane horizontally (terminal on top, logs on bottom)
Ctrl+b "

# In bottom pane, watch logs
tail -f /var/log/syslog
```

Now you have:

- Editor on the left
    
- Terminal on the top right
    
- Logs on the bottom right

All in one SSH connection, and it survives disconnection.

---
***Previous***: [[networking and ssh]]                             ***Next***: [[dotfiles & customizations]]







