# Part 0

## Shell commands

Shell commands will be prefixed by `$` and the output will follow without any prefix.

## Variables instead of examples

I'll be using shell variable notation in place of remote names, branch names, commit messages, etc. This makes it clearer which things are commands and flags and which are names or free text. For instance, although the names `origin` and `master` are the default names for the remote and branch respectively, they could be named anything. So by putting up `git push $remote $branch` instead of `git push origin master`, what I want to emphasise is that the words `origin` and `master` hold no special meaning. It also makes it easier to see the form of the command instead of an example of it from which you would need to infer the form.

---

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

At its core, Git is a database of objects and each object is addressable by its SHA-1 hash. Every other feature is implemented by tools and functions that manipulate this database.

---

## Main Git objects

- Blob: The contents of a file.
- Tree: The equivalent of a directory.
- Commit: A single snapshot of the repository.



---

# Part 3
## The good part

---

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
