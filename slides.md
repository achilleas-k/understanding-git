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

# Part 2
## What is Git and what does it do?

---

Which of these statements are true?
- Git is a version-control system for source code
- Git keeps track of file changes across versions
- Git is a backup tool

---

- Git is a version-control system for source code
    - Kinda
- Git keeps track of file changes across versions
    - Kinda
- Git is a backup tool
    - Not really

---

# Part 2.5
## Hash functions

---

What is a hash function?

- Deterministic
- Quick to compute
- One-way: easy to compute on any input; infeasible to invert given output
- Small changes in input map to large changes in output
- Collision free: infeasible to find two different inputs that map to the same output

---

Why do we care?

---

> Git is fundamentally a content-addressable filesystem with a VCS user interface written on top of it.

Source: https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain

---

Git is a key-value store: A database of objects where each object is addressable by its SHA-1 hash

Every other feature is implemented by tools and functions that manipulate this database.

---
# Part 3
## Get to the point

---

Let's start from scratch

```bash
$ git init
Initialized empty Git repository in $directory
```

Our object store is empty
```bash
$ tree -F .git/objects
.git/objects
├── info/
└── pack/

2 directories, 0 files
```

---

Add a readme
```bash
$ echo "# Understanding Git" > README.md
```
```bash
$ git add README.md
```
```bash
$ git commit -m "Initial commit: Add README"
[master (root-commit) 10f21b8] Initial commit: Add README
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

---

Check the object store again
```bash
$ tree -F .git/objects
.git/objects
├── 10/
│   └── f21b8ddd8fa6ea61fc1e3bb72ae448c5129515
├── 32/
│   └── c65e86d72d50be78c536f79d8036604eb713b1
├── dc/
│   └── d0c23a90b9a55df7caa0c5c182254ace2e0be4
├── info/
└── pack/

5 directories, 3 files
```

---

`rev-parse`
  - A plumbing command used internally for parsing flags and revision arguments

`HEAD`
  - The head (tip) of the current branch
  - Points to the revision in the working directory before any uncommitted changes

---

```bash
$ git rev-parse HEAD
10f21b8ddd8fa6ea61fc1e3bb72ae448c5129515
```

Translates symbolic name `HEAD` to the `SHA-1` key

---

`cat-file`
  - A plumbing command for showing the information or content of a Git object

---

What *type* (`-t`) of object is `HEAD`?

```bash
$ git cat-file -t HEAD
commit
```

---

Show me (`-p` print) the *contents* of `HEAD`

```bash
$ git cat-file -p HEAD
tree dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
author Achilleas Koutsou <achilleas.k@gmail.com> 1542042136 +0100
committer Achilleas Koutsou <achilleas.k@gmail.com> 1542042136 +0100

Initial commit: Add README
```

---

A revision (commit) consists of:
  - A reference to a tree (the top-level directory)
  - A reference to a parent commit (the *previous* commit in history; except when it's the first commit)
  - An author, committer, and a message (commit metadata)

---

Our `HEAD` doesn't have a parent. Let's make a new commit and check again.

---

```bash
echo "# Understanding Git

> Achilleas Koutsou

2018-11-14" > slides.md
git add slides.md
git commit -m "Add first presentation slide"
```

---

```
$ git cat-file -t dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
tree
```
```
$ git cat-file -p dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
100644 blob 32c65e86d72d50be78c536f79d8036604eb713b1	README.md
```
```
$ git cat-file -t 32c65e86d72d50be78c536f79d8036604eb713b1
blob
```
```
$ git cat-file -p 32c65e86d72d50be78c536f79d8036604eb713b1
# Understanding Git
```

Note:
Lets create a Git repository and see how this all works in action.

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
