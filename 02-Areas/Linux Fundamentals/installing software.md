Creation date: Wednesday, February 4th 2026, 2:24:38 am

**Goal**: Install software and keep your system updated.

In Linux, you install software from the command line using a package manager.

### What is a package?

A package is a bundle containing:
- The software itself (compiled programs)
- Configuration files
- Documentation
- Information about dependencies (other packages it needs)

### What is a package manager?

A **package manager** is the tool that:
- Downloads packages from repositories
    
- Installs them correctly
    
- Tracks what’s installed
    
- Handles updates
    
- Manages dependencies automatically

Different Linux distributions use different package managers:

- Ubuntu, DebianAPT`apt`
- Fedora, RHELDNF`dnf`
- ArchPacman`pacman`
- AlpineAPK`apk`

### What are Repositories?

Repositories are servers that store packages. When you run `apt install`, it downloads from these servers. Ubuntu's repositories contain thousands of packages, all tested to work together.

### Meet Sudo

**sudo** = "super user do". It runs commands with administrator (root) privileges. Regular users can't modify system files or install software - you need elevated permissions.

### APT Commands

`sudo apt update`: this downloads the latest list of available packages from the repositories. It doesn't install anything - it just updates the list of what's available.

`sudo apt upgrade`: this upgrades all installed packages to their latest versions.

**Tip**: Combine them: `sudo apt update && sudo apt upgrade`

`apt search [package-name]`: to search for packages

`apt show [package-name]`: to show package information

`sudo apt install [package-name]`: install a package

`sudo apt remove [package-name]`: remove a package | `sudo apt purge [package-name]`: to remove everything including config files

`apt list --installed`: this shows all installed packages. You can filter it using 
`apt list --installed | grep [package-name]`

### Package Installed on this day

`htop`: better process viewer (like task manager)

`tree`: shows directory structure visually

`curl`: downloads files and makes HTTP requests

`vim`: text editor

***Previous***: [[shell]]                                                                    ***Next***: [[file system]]






