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
Lets run through an example workflow where we visit all the states and perform all the transition actions

---

## Begin

::init::

::status1::

---

## File states: git status

```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

All files in the directory (if any) are *Unmodified*.

Note:
The state of a repository where everything known to 

---

# EOF

These slides and accompanying material are available at

`https://github.com/achilleas-k/understanding-git`
and
`https://gin.g-node.org/achilleas/understanding-git`
