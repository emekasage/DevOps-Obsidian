Creation date: Wednesday, February 25th 2026, 2:55:36 am

#### **What is Git?**

Check if Git is installed

```
git --version
```

```
git version 2.43.0
```

```
which git
```

```
/usr/bin/git
```

Git is installed. It’s just another command-line tool, like `ls` or `vim`.

##### What problem does Git solve?

Have you ever had files like this?

- `report_final.docx`
    
- `report_final_v2.docx`
    
- `report_final_REALLY_FINAL.docx`
    
- `report_final_REALLY_FINAL_fixed.docx`

Or emailed files back and forth with colleagues, losing track of who changed what?

Git solves this. It tracks every change you make, who made it, and when. You can go back to any previous version. Multiple people can work on the same project without stepping on each other.

##### Who Created Git?

Linus Torvalds - the same person who created Linux.

In 2005, the Linux kernel team needed a new version control system. Linus spent two weeks writing Git. The name? Linus said it’s either “a random three-letter combination” or British slang for a stupid person (depending on his mood).

Git was built to handle the Linux kernel - one of the largest collaborative software projects in the world. If it can handle that, it can handle your scripts.

---
#### **Git vs GitHub**

**Git** is the tool. **GitHub** is where you store and share your repositories online.

Other options exist (GitLab, Bitbucket), but GitHub is the most common. We’ll use it in this course.

---
#### **Your Repository (Repo)**

A **repository** (or “repo”) is a project folder that Git is tracking.

To create one:

```
mkdir my-project
cd my-project
git init
```

```
Initialized empty Git repository in /home/cato/my-project/.git/
```

```
ls -la
```

```
drwxrwxr-x 3 cato cato 4096 Dec 16 10:00 .
drwxr-xr-x 5 cato cato 4096 Dec 16 10:00 ..
drwxrwxr-x 7 cato cato 4096 Dec 16 10:00 .git
```

See that `.git` directory? That IS your repository. It stores all the history and tracking information. Don’t touch it directly - let Git manage it.

---
#### **The Basic Workflow**

Git has three states for your files:

1. **Working directory** - Files as you edit them
    
2. **Staging area** - Changes you’re preparing to commit
    
3. **Committed** - Saved in Git history

The workflow:

```
Edit files → git add → git commit → git push
```

---
#### **Connecting to GitHub**

Your repository only exists on your computer. Let’s put it on GitHub so:

- It’s backed up
    
- You can access it from anywhere
    
- Others can collaborate

##### Create a new SSH key and add it to GitHub

[https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent "https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent")

Now add it to GitHub:

1. Go to [github.com](https://github.com/ "https://github.com/") → Sign in
    
2. Click your profile picture (top right) → **Settings**
    
3. Click **SSH and GPG keys** (left sidebar)
    
4. Click **New SSH key**
    
5. Title: “My Linux VM” (or whatever helps you remember)
    
6. Paste your public key
    
7. Click **Add SSH key**

##### Test the Connection

```
ssh -T git@github.com
```

```
Hi YOUR_USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

If you see that message, you’re authenticated.

##### Create a GitHub Repository

1. Go to [github.com](https://github.com/ "https://github.com/")
    
2. Click the **+** in the top right → **New repository**
    
3. Name it `my-project`
    
4. **Don’t** check “Add a README” (we already have one)
    
5. Click **Create repository**

GitHub shows you commands to connect. Use the **SSH** option:

```
git remote add origin git@github.com:YOUR_USERNAME/my-project.git

git remote add origin git@github.com:mischavandenburg/my-project.git

git push -u origin master
```

No password prompt - your SSH key handled authentication.

Refresh GitHub in your browser. Your README is there!
##### What Just Happened?

- `git remote add origin URL` - Told Git where GitHub is (“origin” is just a nickname)
    
- `git push -u origin master` - Uploaded commits to GitHub
    
- SSH key authenticated you automatically

From now on, just:

```
git push
```

---
#### **Making Changes**

Edit the README.md file

```
vim README.md
```

Add some content:

```
# My Project

This is my first Git repository.

## What I Learned

- Git tracks changes
- Commits are snapshots
- GitHub stores repositories online
```

Save and exit (`:wq`).

Check what changed:

```
git status
```

See the actual changes:

```
git diff
```

```
diff --git a/README.md b/README.md
index 4b5a123..8c9d456 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,9 @@
 # My Project
+
+This is my first Git repository.
+
+## What I Learned
+
+- Git tracks changes
+- Commits are snapshots
+- GitHub stores repositories online
```

Lines starting with `+` are additions. Lines with `-` are deletions.

Stage, commit, and push:

```
git add README.md
git commit -m "Add learning notes to README"
git push
```

That’s the daily workflow. Edit, add, commit, push.
##### Commit Message Tips

- Keep them short (under 50 characters for the first line)
    
- Say **what** you did, not **how**
    
- Good: “Add user authentication”
    
- Bad: “Changed myscript lines 47-89”

Real teams often follow [Conventional Commits.](https://www.conventionalcommits.org/en/v1.0.0/ "https://www.conventionalcommits.org/en/v1.0.0/")

This is beyond the scope of this course, but my advice is to adopt these standards in every project you do. Even your small private ones.

---
#### **Branches**

In real work, you shouldn't commit directly to `master` or `main` branch.
##### Why Use Branches?

- `master` represents your working, stable code
    
- New features go in separate branches
    
- If the feature breaks, master is still fine
    
- Merge when the feature is ready and tested

Think of it like making a copy of your project, working on it separately, then merging the changes back when you’re done.
##### Create a Branch

```
git switch -c add-feature
```

```
Switched to a new branch 'add-feature'
```

This created a branch AND switched to it. Check:

```
git branch
```

```
* add-feature
  master
```

The `*` shows your current branch.
##### Work on the Branch

```
echo "## New Feature" >> README.md
echo "This is a cool new feature." >> README.md
git add README.md
git commit -m "Add new feature section"
```
##### Push the Branch

```
git push
```

Or:

```
git push -u origin add-feature
```
Your branch is now on GitHub. Notice it tells you how to create a pull request.

---
#### **Pull Requests**

A **pull request** (PR) is a request to merge your branch into master. It’s done on GitHub, not via Git commands.

(This is not completely true: you can review and merge Pull Requests using the GitHub CLI, but that is outside the scope of this course).

##### Why Pull Requests?

- Code review - others can see and comment on your changes
    
- Discussion - have conversations about the implementation
    
- Quality control - don’t merge broken code
    
- History - see why changes were made

##### Create a Pull Request

1. Go to your repository on GitHub
    
2. You’ll see a banner: “add-feature had recent pushes” with a **Compare & pull request** button
    
3. Click it
    
4. Add a title: “Add new feature section”
    
5. Add a description explaining what you did
    
6. Click **Create pull request**

##### Merge the Pull Request

In a team, someone would review your code first. For now:

1. Click **Merge pull request**
    
2. Click **Confirm merge**
    
3. Click **Delete branch** (keeps things clean)

##### Update Your Local Repository

Your branch was merged on GitHub, but your local repository doesn’t know yet:

```
git switch master
```

```
Switched to branch 'master'
Your branch is behind 'origin/master' by 2 commits, and can be fast-forwarded.
```

```
git pull
```

```
Updating a1b2c3d..d4e5f6g
Fast-forward
 README.md | 3 +++
 1 file changed, 3 insertions(+)
```

Delete your local branch (it’s merged, you don’t need it):

```
git branch -d add-feature
```

```
Deleted branch add-feature (was c7d8e9f).
```

---
#### **Cloning Repositories**

What if you want to work on an existing project? Clone it:

```
cd ~
git clone git@github.com:YOUR_USERNAME/my-project.git my-project-copy
```

This downloads the entire repository with all its history.

For public repositories (like open source projects):

```
git clone git@github.com:torvalds/linux.git
```

That would download the entire Linux kernel repository (don’t actually do this unless you have time - it’s huge).

> **Note:** You can clone public repositories without adding your SSH key. But to push changes, you need authentication.

---
#### **When Things Go Wrong**

Git can be confusing. Here’s the DevOps approach: when things get complicated, reset and try again.

##### Discard Uncommitted Changes

You made changes but want to throw them away:

```
# Discard changes to one file
git restore README.md

# Discard ALL uncommitted changes
git restore .
```

##### Undo a Commit (Before Pushing)

You committed but haven’t pushed yet:

```
# Undo commit, keep changes staged
git reset --soft HEAD~1

# Undo commit, keep changes unstaged
git reset HEAD~1

# Undo commit AND discard changes
git reset --hard HEAD~1
```

`HEAD~1` means “one commit before the current one.”

##### The Nuclear Option

Everything is broken and you just want to match what’s on GitHub:

```
git fetch origin
git reset --hard origin/master
```
This throws away ALL local changes and makes your repository match GitHub exactly.

##### The DevOps Reality

In five years of DevOps work, here’s the truth:

- Most of your changes are small
    
- Complex Git operations rarely come up
    
- When they do, it’s faster to reset and redo your work
    
- Don’t spend an hour fixing Git history for 10 minutes of work

When Git gets confusing: `git reset --hard`, make your changes again, move on.

---
#### **Summary**

- **Git** tracks changes to files over time
    
- **GitHub** hosts repositories online
    
- **Repository** = project folder with `.git` directory
    
- **Commit** = saved snapshot of your files
    
- **Branch** = parallel version for working on features
    
- **Pull request** = request to merge a branch

**The daily workflow:**

```
git switch -c feature-name      # Create branch
# ... make changes ...
git add .                       # Stage changes
git commit -m "Description"     # Commit
git push                        # Push to GitHub
# Create PR on GitHub, merge, then:
git switch master
git pull
```

**When things break:**

```
git reset --hard origin/master    # Nuclear option
```