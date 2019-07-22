# Part 0

## Shell commands

Shell commands will be prefixed by `$` and the output will follow without any prefix.

# Part 1
## Porcelain and Plumbing

The Git manual and documentation pages separate commands into *porcelain* and *plumbing* commands. These are high level and low level commands, respectively. The metaphor being that *plumbing* is what happens behind the scenes, out of sight. In contrast, *porcelain* refers to the parts of the system that the user interacts with directly.

### Porcelain: High level commands

Anyone learning Git will first come into contact with *porcelain* commands: `add`, `commit`, `push`, `pull`, `log`, `fetch`, `merge`.

### Plumbing: Low level commands

Basic operations. Porcelain commands call one or (usually) more plumbing commands. Plumbing commands perform a single operation and are meant to be relied on for scripting or for building your own porcelain (interfaces to Git). They're stable, in other words, their output and operational details aren't expected to change often.

Examples: `apply`, `rev-parse`, `config`, `cat-file`, `ls-files`

I wont be talking much about individual plumbing commands, but we will be using some to inspect the inner workings of Git as we go along. For me today it's not important that you walk away with a bunch of new commands that you can use. I want you to walk away with a deeper understanding of Git's inner workings, to the point where you can apply that knowledge to create more powerful workflows yourselves.

---

# Part 2
## What is Git and what does it do?

---

Which of these statements are true?
- Git is a version-control system for source code
- Git keeps track of file changes across versions
- Git is a backup tool

---

Kinda x3

- Git is a version-control system. It's not specific to source code. In fact it's not even specific to plain text files, though it works much better with text. More on this later.
- Each commit (revision) is a snapshot of the files (directory tree, filenames, file contents). It doesn't track changes in the strictest sense, but it does compute changes (eventually). More on this later.
- Git is not a backup tool. The third statement is the weakest of that list. It's actually *mostly false*. There's some truth to it: Git provides a way to keep and retrieve file versions, you can store all your versions in multiple locations (remotes), it can verify data integrity. However, it doesn't handle binary data well, it doesn't track extended attributes and filesystem metadata (owners, groups, permissions).

---

# Part 2.5
## Hash functions

What is a hash function?

---

Generally: Functions that compute output of a fixed size (a hash) from input data of arbitrary size.

**Cryptographic** hash functions are:
- Deterministic
- Quick to compute
- One-way: easy to compute on any input; impossible (infeasible) to invert given output
- Small changes in input map to large changes in output
- Collision free: infeasible to find two different inputs that map to the same output

---

## Why do we care?

---

Git uses cryptographic hashes (SHA-1) to identify objects.

> [...] Git is fundamentally a content-addressable filesystem with a VCS user interface written on top of it.

Source: https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain

At its core, Git is a key-value store, database of objects where each object is addressable by its SHA-1 hash. Every other feature is implemented by tools and functions that manipulate this database.

We insert objects into the database, Git gives us a key (SHA-1 hash) of the objects, and then we use the keys to retrieve the objects.

---

## Main Git objects

- Blob: The contents of a file
- Tree: The equivalent of a directory
- Commit: A single snapshot of the repository

---

# Part 3
## Get to the point

---

Let's start a Git repository from scratch. We'll make a couple of snapshots (commits) and drill down into each object to start building our understanding of the whole system.

We initialise a directory with the simple `git init` command. This just sets up the current directory we're working in as a Git repository.

Everything related to Git is stored under the `.git` directory, which resides under the directory where we ran `git init`. All the Git objects are stored in the `.git/objects` subdirectory. Since this is a fresh new repository, the object store is empty.

---

Let's create a README and add the first commit. To create a first commit, we first need to *add* the changes we want to record into the *commit*. Remember, a commit is a snapshot of the repository, but we need to tell Git which files and file changes we want to add to the snapshot. We *add* the README to Git using the `add` command.  We create a *commit* using the `commit` command. The `-m` option in the co`commit` command lets us add a message to describe the snapshot. If we didn't add this, Git would open a text editor to ask us for a message.

Nb: Empty commit messages are allowed, but they're generally frowned upon so by default Git wont let you create a commit without a message.

---

Checking the object store again, we see that Git has created some objects.

Git stores its contents compressed, so we can't just open the files to see inside, but we can start using some *plumbing* commands to inspect Git's internals.

---

Our first plumbing command
- `cat-file`: shows information about or content of a Git object
    - `-t`: print type of object
    - `-p`: pretty-print contents of object

`pretty-print` is a commonly used term for printing data in a human-readable format. It usually implies that data will be printed indented or coloured in ways that make it easier to read but wouldn't be good to work with programmatically.

Running both `git cat-file -t` and `git cat-file -p` on each of the objects in our store will show us their type and contents.

```bash
$ git cat-file -t 76c979922c8d1938330c4f659806bab23109def8
commit
```
```bash
$ git cat-file -p 76c979922c8d1938330c4f659806bab23109def8
tree dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
author Achilleas Koutsou <achilleas.k@gmail.com> 1542117871 +0100
committer Achilleas Koutsou <achilleas.k@gmail.com> 1542117871 +0100

Initial commit: Add README
```

```bash
$ git cat-file -t dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
tree
```
```bash
$ git cat-file -p dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
100644 blob 32c65e86d72d50be78c536f79d8036604eb713b1    README.md
```

```bash
$ git cat-file -t 32c65e86d72d50be78c536f79d8036604eb713b1
blob
```
```bash
$ git cat-file -p 32c65e86d72d50be78c536f79d8036604eb713b1
# Understanding Git
```
---

# Part N

## Workflows

### Add Commit Push

```bash
$ git add $filename
$ git commit -m $commitmsg
$ git push $remote $branch
```

#### Break it down

`git add`: ...

`git commit`: ...

`git push`: ...

### Pull (Fetch & Merge)

Pull is a more interesting case. It's not just a high level command, it's actually a combination of two high level commands: `fetch` and `merge`.

```bash
$ git fetch $remote
$ git merge $remote/$branch
```

#### Break it down

`git fetch`: ...

`git merge`: ...

`git push`: ...
