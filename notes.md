# Part 0

## Shell commands

Shell commands will be prefixed by `$` and the output will follow without any prefix.

## Variables instead of examples

I'll be using shell variable notation in place of remote names, branch names, commit messages, etc. The purpose is to point out which things are commands and flags and which are names or free text. For instance, although the names `origin` and `master` are convention and the default names for the remote and branch respectively, they could be named anything. So by putting up `git push $remote $branch` instead of `git push origin master`, what I'm trying to emphasise is that the words `origin` and `master` hold no special meaning. It also makes it easier to see the form of the command instead of an example of it from which you would need to infer the form.

---

# Part 1
## Porcelain and Plumbing

The Git manual and documentation pages separate commands into *porcelain* and *plumbing* commands. These are high level and low level commands, respectively. The metaphor being that *plumbing* is what happens behind the scenes, out of sight. In contrast, *porcelain* refers to the parts of the system that the user interacts with directly.

### Porcelain: High level commands

Anyone learning Git will first come into contact with *porcelain* commands: `add`, `commit`, `push`, `pull`, `log`, `fetch`, `merge`.

### Plumbing: Low level commands

Basic operations. Porcelain commands call one or (usually) more plumbing commands. Plumbing commands perform a single operation and are meant to be relied on for scripting or for building your own porcelain (interfaces to Git). They're stable, in other words, their output and operational details aren't expected to change often.

Examples: `apply`, `rev-parse`, `config`, `cat-file`, `ls-files`

I wont be talking much about plumbing commands, but we will be using some to inspect the inner workings of Git as we go along. For me today it's not important that you walk away with a bunch of new commands that you can use. I want you to walk away with a deeper understanding of Git's inner workings, to the point where you can apply that knowledge to create more powerful workflows yourselves.

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
