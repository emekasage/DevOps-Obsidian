## What it is

Commitizen is a tool that helps teams write consistent and meaningful commit messages and automate versioning and changelog generation in projects. It enforces a commit message style by default based on the **Conventional Commits** specification.

## Why it matters

- Makes commit history more readable.
- Helps automate version bumps and changelog updates. 
- Works well with **semantic versioning**

## Key features

- Interactive CLI for creating standardized commits. 
- Version bumping and changelog generation.
- Commit message validation (can integrate with pre-commit hooks). 
- Custom rules and templates via plugins.

## Basic commands

```
# Initialize Commitizen config
cz init

# Create a commit interactively
cz commit
# or
cz c

# Bump version and update changelog
cz bump
```

## Install

Commitizen can be installed globally or in a project using pipx: 
```
pipx install commitizen
pipx upgrade commitizen
```

Commitizen can also be installed on macOS with Homebrew:
```
brew install commitizen
```


## Useful links
- #### Commitizen docs: https://commitizen-tools.github.io/commitizen/
- #### Commitizen commit spec: https://www.conventionalcommits.org/
