This note covers the core Markdown syntax used in Obsidian and GitHub-style documentation.

---

## Headings

Used to structure content.

# H1 - Main Title
## H2 - Section
### H3 - Subsection
#### H4 - Smaller section

Rules:
- Use one space after the #
- Use H1 once per note when possible

---

## Paragraphs

Just write text normally.
Leave a blank line between paragraphs.

---

## Bold and Italic

**Bold text**
*Italic text*
***Bold and italic***

---

## Bullet Lists

- Item one
- Item two
- Item three

Nested bullets:
- Parent item
  - Child item
  - Child item

---

## Numbered Lists

1. First item
2. Second item
3. Third item

---

## Checklists (Tasks)

- [ ] Not started
- [x] Completed
- [ ] In progress

Useful for projects and daily notes.

---

## Internal Links (Obsidian)

Link to another note:
[[Docker]]
[[Linux Permissions]]
[[Kubernetes Volumes]]

If the note does not exist yet, Obsidian will create it when clicked.

---

## External Links

[Link text](https://example.com)

---

## Code (Inline)

Use single backticks for commands or short code.

`docker ps`
`kubectl get pods`

---

## Code Blocks (Multi-line)

Use triple backticks.

```bash
docker ps
docker images
```

```
jhwbjshbjksa
```

Specify language for syntax highlighting:

- bash
    
- yaml
    
- json
    
- python
    
- terraform
    

---

## Blockquotes

> This is a blockquote.  
> Useful for notes or quoted text.

---

## Horizontal Rule

Creates a divider.

---

---

---

## Tables

| Column 1 | Column 2 |
| -------- | -------- |
| Value A  | Value B  |
| Value C  | Value D  |

---

## Escaping Characters

Use a backslash to escape formatting characters.

*This will not be italic*

## Best Practices

- Keep notes simple and readable
    
- Prefer headings over long paragraphs
    
- Use code blocks for commands and configs
    
- Link related notes using double brackets
    
- Don’t over-format
    

Markdown is meant to be lightweight.
