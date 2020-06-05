---
title: Understanding Git — Basics
theme: black
css: style.css
revealOptions:
  transition: 'none'

---

# Understanding Git: Basics

> Achilleas Koutsou

2019-07-23

---

# Part 0
## Preface

Note:
Who here:
- has worked with Git before?
- uses Git regularly?
- uses Git to collaborate with other people?

---

# Part 1
## Git basics

Note:
We'll start by looking at the basic components of a git repository and go through the basic workflow, explaining how it relates to and affects these objects

---

## What is a git repository made of?

A series of snapshots with some metadata about the snapshots and their content

Note:
At the highest level, a git repository is a series of snapshots with some metadata about the snapshots and their content

---

## What is a git repository made of?

- Snapshots: Commits
  - Metadata: Author, message, timestamp, etc
- File contents: Blobs
- Directories: Trees


Note:
The three basics objects of a git repository

---

## File states

- Files can be *tracked* or *untracked*
  - Untracked files are unknown to git
  - Tracked files can be in one the following states:
    - Unmodified
    - Modified
    - Staged

Note:
During the workflow, files cycle through a number of states, starting from *untracked* (unknown to git).
Once *added* to a git repository, a file can exist in one of three states, *unmodified*, *modified*, and *staged*.

---

## File states: Transitions

```
   .------------.  .------------.  .------------.  .------------.
   | Untracked  |  | Unmodified |  | Modified   |  | Staged     |
   ·------------·  ·------------·  ·------------·  ·------------·
         |               |               |               |
         |--------------------ADD----------------------->|
         |               |               |               |
         |               |----EDIT------>|               |
         |<----REMOVE----|               |----STAGE----->|
                         |                               |
                         |<----------COMMIT--------------|


```

(Adapted from https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

Note:
This illustrates the states of a file and the actions that make it transition from one to another

---

# Part 2

## Workflow

Note:
The state transitions shown on the previous slide, on that state diagram, correspond to `git` commands.

We'll go through each and explain what it does exactly before we see them in action.

---

### Starting from scratch

Create an empty git repository in the current directory.

```text
$ git init
Initialized empty Git repository in <dir>

$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Note:
This will initialise the current working directory to become an empty git directory.  It doesn't matter if there are files in the directory or not.  It simply marks the directory as a git repository and ignores any files that may already be present.

The git status command shows you the current status of the repository.  We'll be using it after every step to inspect the repository and check our changes.

---

### Starting from scratch

Any files in the directory are **Untracked**.
```git
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        notes.txt

nothing added to commit but untracked files present (use "git add" to track)
```

---

### Add

Add a file in the directory to git

```text
$ git add notes.txt

$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   notes.txt
```

`notes.txt` is now in the **Staged** state.

Note that the file is prefixed with `new file:` under the `Changes to be committed:` heading.

Note:
Tells git to start tracking the file.

There are still no commits, but the file has been added, which puts it in the Staged state.

---

### Commit

Create a commit with the staged changes

```text
$ git commit -m "First commit: Add notes.txt"
[master (root-commit) 3c1915a] First commit: Add notes.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 notes.txt

$ git status
On branch master
nothing to commit, working tree clean
```

- We use `-m` to specify the commit _message_.  It should be short and descriptive.
- The `notes.txt` file is now in the **Unmodified** state.

Note:
Tracked Unmodified files don't appear in the `git status` output.

---

### Edit

```text
$ echo "Repository notes and instructions" >> notes.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   notes.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

The file is **Staged** to be committed.

Note that the file is prefixed with `modified:` under the `Changes not stated for commit:` heading.

---

### Inspect the changes

```diff
$ git diff
diff --git a/notes.txt b/notes.txt
index e69de29..b0037c9 100644
--- a/notes.txt
+++ b/notes.txt
@@ -0,0 +1 @@
+Repository notes and usage instructions
```

Shows the `diff`erences between the working tree (the current state) and the git state.

Note:
`git diff` can be used to show the differences between two states of the same file (or files).  Without arguments, it shows the differences between the current and last committed state.  In other words, it shows the unstaged modifications made to a file since the last commit.

---

### Stage (Add)

```text
$ git add notes.txt
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   notes.txt
```

The file is **Staged** to be committed.

Note that the file is prefixed with `modified:` under the `Changes to be committed:` heading.

---

### Commit

```text
$ git commit -m "Add heading for notes"
[master b29f432] Add heading for notes
 1 file changed, 1 insertion(+)

$ git status
On branch master
nothing to commit, working tree clean
```

We are back in the (clean) **Unmodified** state.

---

### Remove

```text
$ git rm notes.txt
rm 'notes.txt'

$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    notes.txt
```


```text
$ git commit -m "Remove notes file"
[master 78e8ffc] Remove notes file
 1 file changed, 1 deletion(-)
 delete mode 100644 notes.txt
```

Note:
Contrary to what the diagram shows, removal also requires a commit.  It's not a direct transition from **Unmodified**.

---

### History

```text
$ git log
commit 78e8ffcaac604400951ee7d0340be74fb0f58bcf
Author: Achilleas Koutsou <achilleas@example.org>
Date:   Thu Jun 4 21:22:36 2020 +0200

    Remove notes file

commit b29f432d3d7cff92bc5e61e6bd3b04df23b673d9
Author: Achilleas Koutsou <achilleas@example.org>
Date:   Thu Jun 4 21:10:06 2020 +0200

    Add heading for notes

commit 3c1915a5148ac3917be1cfe78a830a383d2d7ad6
Author: Achilleas Koutsou <achilleas@example.org>
Date:   Thu Jun 4 20:22:35 2020 +0200

    First commit: Add notes.txt
```

The history of states is always available


Note:
This is why clean, short commit messages are important.

---

# EOF

Slides and accompanying material at

https://github.com/achilleas-k/understanding-git
https://gin.g-node.org/achilleas/understanding-git
