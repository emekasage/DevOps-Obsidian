Creation date: Friday, February 20th 2026, 6:56:53 am

#### **Network Configuration**

##### View IP Address

```
ip addr OR ip a
```

- `lo` - Loopback interface (127.0.0.1 = localhost)
- `eth0` (or `ens33`, `enp0s3`) - Your network interface

##### View Routing Table

```
ip route
```

Output:

```
default via 192.168.248.2 dev ens33 proto dhcp src 192.168.248.128 metric 100 

192.168.248.0/24 dev ens33 proto kernel scope link src 192.168.248.128 metric 100 

192.168.248.2 dev ens33 proto dhcp scope link src 192.168.248.128 metric 100
```

The default route (`default via 192.168.248.2`) is your gateway - where traffic goes to reach the internet.

##### View and Set Hostname

```
hostname
```

```
hostnamectl
```
Shows detailed hostname and OS information

##### The /etc/hosts File

```
cat /etc/hosts
```

Output:

```
127.0.0.1       localhost
127.0.1.1       cato OR caspian OR [your_hostname]
```

This file maps hostnames to IP addresses locally. Useful for:

- Adding shortcuts to servers you access often
    
- Blocking websites (point them to 127.0.0.1)
    
- Testing before DNS changes

Add an entry:

```
sudo vi /etc/hosts
```

Add:

```
192.168.1.50    myserver
```

Now `ping myserver` works without knowing the IP.

---
#### **Network Tools**

##### Test Connectivity: `ping`

```
ping google.com
```

```
ping -c 4 google.com
```
Send only 4 pings and stop.

##### Download Files: `curl`

```
curl https://example.com
```
Displays the content (HTML).

```
curl -O https://raw.githubusercontent.com/mischavandenburg/lab/refs/heads/main/README.md
```
Downloads to a file (`-O` uses the remote filename).

```
curl -o myfile.md https://raw.githubusercontent.com/mischavandenburg/lab/refs/heads/main/README.md

```
Downloads with a custom filename.

```
curl -I https://example.com
```
Show just the headers (useful for checking if a site is up).

##### Download Files: `wget`

```
wget https://raw.githubusercontent.com/mischavandenburg/lab/refs/heads/main/README.md
```
Downloads to a file. Simple and reliable.

##### View Network Connections: `ss`

```
ss -tuln
```

Flags:

- `-t` - TCP
    
- `-u` - UDP
    
- `-l` - Listening (server) sockets
    
- `-n` - Shows port numbers (not service names)

This shows SSH (port 22) is listening.

See established connections:

```
ss -tun
```

---
#### **SSH Deep Dive**

##### What is SSH?

SSH (Secure Shell) creates an encrypted connection between your computer and a remote server.

Everything you type and all output is encrypted.

Components:

- **SSH client** - On your computer (you type `ssh`)
    
- **SSH server** - On the remote machine (sshd daemon)
    
- **Encryption** - Your traffic is unreadable to anyone in between

##### How SSH Works

1. You run `ssh user@server`
    
2. Client connects to server port 128
    
3. They negotiate encryption (key exchange)
    
4. You authenticate (password or key)
    
5. Encrypted channel established
    
6. Your commands run on the server
##### Password vs Key Authentication

**Password authentication:**

- Simple but less secure
    
- You type password every time
    
- Vulnerable to brute force attacks

**Key authentication:**

- More secure
    
- No password needed once set up
    
- Based on cryptographic key pairs
    
- Required by many cloud providers

##### Generate SSH Keys

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Output:

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
```

Press Enter to accept the default location. You can add a passphrase for extra security.

This creates:

- `~/.ssh/id_ed25519` - Private key (NEVER share this)
    
- `~/.ssh/id_ed25519.pub` - Public key (safe to share)

##### Copy Key to Server

```
ssh-add ~/.ssh/id_ed25519

ssh-copy-id sagecode@192.168.248.128
```

You’ll enter your password once. After this, you can log in without a password.

Manual method (if `ssh-copy-id` isn’t available):

```
cat ~/.ssh/id_ed25519.pub | ssh sagecode@192.168.248.128 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

##### Verify Key Authorization is Working

After copying your key, verify it’s actually being used. Use verbose mode on your local:

```
ssh -v sagecode@192.168.248.128
```

Look for these lines in the output:

```
debug1: Authentications that can continue: publickey,password
debug1: Offering public key: /home/user/.ssh/id_ed25519 ED25519
debug1: Server accepts key: /home/user/.ssh/id_ed25519 ED25519
debug1: Authentication succeeded (publickey).
```

The key lines are:

- `Offering public key` - SSH is trying your key
    
- `Server accepts key` - The server recognized your public key
    
- `Authentication succeeded (publickey)` - You’re in with your key, not password

If you see `Authentication succeeded (password)` instead, your key isn’t set up correctly.

##### Disable Password Authentication

Once key authentication works, disable password login for better security. This prevents brute force attacks.

On the **server** (the machine you SSH into):

```
sudo vi /etc/ssh/sshd_config
```

Find and change these lines:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

If the lines have `#` at the start, remove the `#` to uncomment them.

Restart the SSH service:

```
sudo systemctl restart sshd
```

**Test before disconnecting:** Open a new terminal and verify you can still connect:

```
ssh user@192.168.100.71
```

If key auth fails now, you’ll be locked out. That’s why you test with a separate connection while keeping your current session open.

##### SSH Config File

Create `~/.ssh/config` on your local:

```
vi ~/.ssh/config
```

Add:

```
Host cato
    HostName 192.168.248.128
    User yourusername
    IdentityFile ~/.ssh/id_ed25519
```

Now instead of:

```
ssh yourusername@192.168.248.128
```

You just type:

```
ssh caspian
```

Set proper permissions:

```
chmod 600 ~/.ssh/config
```

##### Copy Files: `scp`

```
scp file.txt user@server:/home/user/
```

Copy local file to remote server.

```
scp user@server:/var/log/syslog ./
```

Copy remote file to local directory.

```
scp -r directory/ user@server:/home/user/
```

Copy a directory recursively.

##### SSH Best Practices

1. **Use keys, not passwords** - Disable password auth if possible
    
2. **Use a passphrase** - Protects your key if someone gets the file
    
3. **Use the config file** - Simpler commands, fewer mistakes
    
4. **Keep private keys private** - Never share, never commit to git

---

***Previous***: [[processes]]                                             ***Next***: [[Tmux]] 