Creation date: Sunday, March 8th 2026, 10:43:42 am

**Goal**: Learn to write small, composable tools that work with your editor. This is how I actually use bash day-to-day.

#### **The Unix Philosophy**

A Unix filter is simple:

- Read from stdin
    
- Write to stdout
    
- Do one thing

That’s it. No arguments, no fancy options. Just transform input to output.

```
# These are all filters
grep error file.txt | sort | uniq
```

Each command reads stdin, does its thing, writes stdout. They compose.

---
#### **Setting Up Your Filter Directory**

Before we write filters, let’s set up a place to put them.

The standard location for user scripts is `~/.local/bin` - this follows the XDG Base Directory convention and many systems already include it in PATH.

```
mkdir -p ~/.local/bin
```

Check if it’s already in your PATH:

```
echo $PATH | tr ':' '\n' | grep local
```

If not, add it:

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Now any executable script you put in `~/.local/bin` is available as a command - including inside vim.

---
#### **Creating Filters**

Create `upper`:

```
#!/bin/bash
read -r line
echo "${line^^}"
```

No `tr`, no external commands - just parameter expansion from Module 3.

```
echo "hello world" | upper
# HELLO WORLD
```

Create `lower`:

```
#!/bin/bash
read -r line
echo "${line,,}"
```

These are tiny, but that’s the point. Small tools that do one thing.

---
#### **Practical Filters**

##### gendate - Insert Today's Date

```
#!/bin/bash
date +%Y-%m-%d
```

This one ignores stdin entirely - it just outputs today’s date. Still a filter.

##### slugify - Convert Text to URL Slug

```
#!/bin/bash
read -r line
line="${line,,}"
line="${line//[^a-z0-9]/-}"

# NOTE: ^ means negated because it's in brackets

line="${line//--/-}"
echo "${line#-}"
```

```
echo "Hello World! This is a Test" | slugify
```

Output: `hello-world-this-is-a-test`

All parameter expansion - no `tr`, no `sed`.

---
#### **Vim Integration**

This is where it gets good. Your scripts become part of your editor.

In vim, `!` runs text through an external command.

##### The `!! or !` Command

`!! or !` filters the current line through a command:

```
Hello World
```

Type `!!upper<Enter>`:

```
HELLO WORLD
```

##### Insert Today’s Date

On an empty line, type `!!gendate<Enter>`:

```
2024-01-15
```

You just inserted today’s date without leaving vim.

##### Filter a Selection

In visual mode, select lines, then type `:!sort<Enter>` to sort them.

##### Common Patterns

`!!command` - Filter current line
`!}command` - Filter to end of paragraph
`!Gcommand` - Filter to end of file
`:'<,'>!command` - Filter visual selection

---
#### **Why This Matters**

I write these tiny scripts constantly. They take 30 seconds to write and save hours over time.

1. **Speed** - `!!gendate` is faster than typing the date
    
2. **Accuracy** - No typos in date formats
    
3. **Composability** - Chain them together
    
4. **Editor integration** - Your scripts work inside vim

This is the Unix way. Small tools. Text in, text out. Everything composes.

---
***Previous***: [[Arrays & Functions]]









