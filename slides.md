---
title: Understanding Git
theme: black
revealOptions:
  transition: 'none'

---

# Understanding Git

> Achilleas Koutsou

2018-11-14

---

# Part 0
## Preface

---

## Shell commands

Shell commands will be prefixed by `$` and the output will follow without any prefix:
```bash
$ echo "Like this"
Like this
```

---

## Variables instead of examples

Shell variable notation will be used in place of remote names, branch names, commit messages, etc.

For example:
```bash
$ git commit -m $commitmsg
$ git push $remote $branch
```

Instead of:
```bash
$ git commit -m "Initial commit: Add README and templates"
$ git push origin master
```

---

# Part 1
## Porcelain and Plumbing

Note:
The Git manual and documentation pages separate commands into *porcelain* and *plumbing* commands. These are high level and low level commands, respectively. The metaphor being that *plumbing* is what happens behind the scenes, out of sight. In contrast, *porcelain* refers to the parts of the system that the user interacts with directly.



---

# Code

```bash
echo "This is bash version $(bash --version)"
```

```python
def prmsg():
  print("This is Python code")
```

---

# Part 1.5
## Hash functions

What is a hash function?

---

# Part 2
## What is Git and what does it do?

---

Which of these statements are true?
- Git is a version-control system for source code
- Git keeps track of file changes across versions
- Git is a backup tool

---

# Part N
## Workflows

---

# Add Commit Push

```bash
$ git add $filename
$ git commit -m $commitmsg
$ git push $remote $branch
```

---

# Pull
## Fetch & Merge

```bash
$ git fetch $remote
$ git merge $remote/$branch
```
