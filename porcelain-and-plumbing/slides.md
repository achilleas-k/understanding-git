---
title: Understanding Git
theme: black
css: ../style.css
revealOptions:
  transition: 'none'

---

# Understanding Git

> Achilleas Koutsou

2018-11-14

---

# Part 0
## Preface

Note:
Who here:
- has worked with Git before?
- uses Git regularly?
- uses Git to collaborate with other people?

If you're completely unfamiliar with Git, this talk will probably give you some basics to get started, but more importantly, it should give you enough knowledge so that if you go through a beginner's tutorial (of which there are plenty), it will be easier to *understand* the tutorial in more depth.

---

# Part 1
## Porcelain and Plumbing

Note:
The Git manual and documentation pages separate commands into *porcelain* and *plumbing* commands. These are high level and low level commands, respectively. The metaphor being that *plumbing* is what happens behind the scenes, out of sight. In contrast, *porcelain* refers to the parts of the system that the user interacts with directly.

---

## Porcelain

`init`, `add`, `commit`, `push`, `pull`, `log`, `fetch`, `merge`, `rebase`, `clone`, `diff`, ...

*High level commands*

Note:
Anyone learning Git will first come into contact with *porcelain* commands: `add`, `commit`, `push`, `pull`, `log`, `fetch`, `merge`.

---

## Plumbing: Low level commands

`rev-parse`, `cat-file`, `ls-files`, `config`, `apply`, ...

*Low level commands*

Note:
Basic operations. Porcelain commands call one or (usually) more plumbing commands. Plumbing commands perform a single operation and are meant to be relied on for scripting or for building your own porcelain (interfaces to Git). They're stable, in other words, their output and operational details aren't expected to change often.

Examples: `apply`, `rev-parse`, `config`, `cat-file`, `ls-files`

---

## Our goal

Inspect the *plumbing* to gain a deeper understanding of the *porcelain*

Note:
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

- Git is a version-control system for source code
    - Kinda
- Git keeps track of file changes across versions
    - Kinda
- Git is a backup tool
    - Not really

---

# Part 2.5
## Hash functions

Note:
Before we talk more about Git, let me take a minute to talk about hash functions.

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

- Git is a key-value store: A database of objects where each object is addressable by its SHA-1 hash
- Every other feature is implemented by tools and functions that manipulate this database.

---

Example SHA-1 hashes
```plaintext
6aeb411c21381069e37981f0e97e1b6bd26f9ff5
109c6766058225867c46ebe79d1ff113ac7edf6d
```

Short hashes
```plaintext
6aeb411
109c676
```

Note:
A SHA-1 hash looks like a *seemingly random* sequence of 40 hexadecimal digits [0-9, a-f]
(emphasis on *seemingly*)

Git uses these hashes to identify all the objects it uses internally.

Git also accepts short versions of any hash, by using the first few characters, as long as it's unambiguous inside the repository (minimum 4).

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
.git/objects
├── info/
└── pack/

2 directories, 0 files
```
Note:
Let's start a Git repository from scratch. We'll make a couple of snapshots (commits) and drill down into each object to start building our understanding of the whole system.

We initialise a directory with the simple `git init` command. This just sets up the current directory we're working in as a Git repository.

Everything related to Git is stored under the `.git` directory, which resides under the directory where we ran `git init`. All the Git objects are stored in the `.git/objects` subdirectory. Since this is a fresh new repository, the object store is empty.

---

Add a readme
```bash
$ echo "# Understanding Git: Presentation slides repository" > README.md
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

Note:
Let's create a README and add the first commit. To create a first commit, we first need to *add* the changes we want to record into the *commit*. Remember, a commit is a snapshot of the repository, but we need to tell Git which files and file changes we want to add to the snapshot. We *add* the README to Git using the `add` command.  We create a *commit* using the `commit` command. The `-m` option in the co`commit` command lets us add a message to describe the snapshot. If we didn't add this, Git would open a text editor to ask us for a message.

Nb: Empty commit messages are allowed, but it's almost never a good idea to create a commit without a descriptive message, so by default Git wont let you create a commit without a message.

---

Check the object store again
```bash
$ tree -F .git/objects
.git/objects
├── 10/
│   └── 9c6766058225867c46ebe79d1ff113ac7edf6d
├── 6a/
│   └── eb411c21381069e37981f0e97e1b6bd26f9ff5
├── 82/
│   └── 7b7984168fe0833602c98c31dac93bd50facd8
├── info/
└── pack/

5 directories, 3 files
```

Three objects. What do they represent?

Note:
Checking the object store again, we see that Git has created some objects.

Git stores its contents (zlib) compressed, so we can't just open the files to see inside, but we can start using some *plumbing* commands to inspect Git's internals.

---

Our first plumbing command
- `cat-file`: shows information about or content of a Git object
    - `-t`: print type of object
    - `-p`: pretty-print contents of object

Note:

`cat-file`: shows information about or content of a Git object

We will use two options or variants of the `cat-file` command.
- `-t`: print type of object
- `-p`: pretty-print contents of object

`pretty-print` is a commonly used term for printing data in a human-readable format. It usually implies that data is indented or coloured in ways that make it easier to understand but wouldn't be good to work with programmatically.

---

```bash
$ git cat-file -t 827b7984168fe0833602c98c31dac93bd50facd8
commit
```
```bash
$ git cat-file -p 827b7984168fe0833602c98c31dac93bd50facd8
tree 109c6766058225867c46ebe79d1ff113ac7edf6d
author Achilleas Koutsou <ak@example.com> 1542209857 +0100
committer Achilleas Koutsou <ak@example.com> 1542209857 +0100

Initial commit: Add README
```

Note:
First object is a `commit`. It's the revision or snapshot we made with the `git commit` command. If we print its contents, we can see all the information associated with the commit.

Note the commit message we used.

The author and committer with the email addresses are added automatically, based on the global (or local) git configuration.
We'll refer to this information, along with the date and timestamps, as the *metadata* of the commit.

You may notice one more piece of information there: The `tree`. This is a reference to the next object we'll inspect.

---

```bash
$ git cat-file -t 109c6766058225867c46ebe79d1ff113ac7edf6d
tree
```
```bash
$ git cat-file -p 109c6766058225867c46ebe79d1ff113ac7edf6d
100644 blob 6aeb411c21381069e37981f0e97e1b6bd26f9ff5	README.md
```

Note:
A `tree` is analogous to a filesystem directory. It contains a list of references to files: in this case, the README file we added to the commit with the `git add` command.

It can also contain references to other trees, the same way a directory can have subdirectories.

Finally, the third object we'll inspect again appears here: It's a reference to the file `README.md`.

---

```bash
$ git cat-file -t 6aeb411c21381069e37981f0e97e1b6bd26f9ff5
blob
```
```bash
$ git cat-file -p 6aeb411c21381069e37981f0e97e1b6bd26f9ff5
# Understanding Git: Presentation slides repository
```

Note:
Git uses the term `blob` to refer to file contents. Note that the file name `README.md` doesn't appear anywhere in the blob, only its contents. At the blob level, Git doesn't care about the name of the file. That information is stored in the tree which we saw earlier. The entry in the `tree` object is linked to the `blob` object using the key (hash).

---

## Git objects

- Commit: A single snapshot of the repository
- Tree: The equivalent of a directory
- Blob: The contents of a file

Note:
That's basically all the main objects that Git uses.

A `commit` is a snapshot of the repository. It contains a message, some metadata, and a reference to a tree.

A `tree` is the equivalent of a directory. It contains references to other trees and blobs along with the name (and mode) for each.

A `blob` is the equivalent of a file. It is simply the contents of a file checked into Git.

---
Second commit

```bash
echo "# Understanding Git

> Achilleas Koutsou

2018-11-14" > slides.md
```
```bash
$ git add slides.md
```
```bash
$ git commit -m "Add first presentation slide"
[master e9cb79d] Add first presentation slide
 1 file changed, 1 insertion(+)
  create mode 100644 slides.md
```

Note:
Let's create a second commit and run through our inspection process again.

---
```bash
$ find .git/objects -type f
.git/objects/ec/9d120bad321f68f257d730f893f1a9e8d96854
.git/objects/12/71443a51490d9457bbf0ccd05805d9948cdc2f
.git/objects/94/6086806911f33c726b8e4088bb3ea5a28ac7a2
.git/objects/82/7b7984168fe0833602c98c31dac93bd50facd8
.git/objects/10/9c6766058225867c46ebe79d1ff113ac7edf6d
.git/objects/6a/eb411c21381069e37981f0e97e1b6bd26f9ff5
```

3 new objects

Note:
There's 6 objects now. If we compare the hashes with the previous time we checked the object store, we'll see that three of the objects are the same. So three new objects were created. Knowing what we know about the state of the object store after the first commit, we can probably guess what the new objects are.

Let's have a look at the new files.

---

```bash
$ git cat-file -t ec9d120bad321f68f257d730f893f1a9e8d96854
commit
```
```bash
$ git cat-file -p ec9d120bad321f68f257d730f893f1a9e8d96854
tree 1271443a51490d9457bbf0ccd05805d9948cdc2f
parent 827b7984168fe0833602c98c31dac93bd50facd8
author Achilleas Koutsou <ak@example.com> 1542209857 +0100
committer Achilleas Koutsou <ak@example.com> 1542209857 +0100

Add first presentation slide
```

Note:
Just like last time, the first object is a `commit`. It's our new commit, where we added the presentation slide. Notice anything different?

This commit has one extra bit of information. It has a `parent`! I think we can all guess what that refers to.

If you look at the hash, it's the key of the first (initial) commit. Let's look at the other two new objects and then return to this.

---

```bash
$ git cat-file -t 1271443a51490d9457bbf0ccd05805d9948cdc2f
tree
```
```bash
$ git cat-file -p 1271443a51490d9457bbf0ccd05805d9948cdc2f
100644 blob 6aeb411c21381069e37981f0e97e1b6bd26f9ff5	README.md
100644 blob 946086806911f33c726b8e4088bb3ea5a28ac7a2	slides.md
```

Note:
Again, the second object is a `tree`. The tree now has two entries. The README.md from before and the slides.md we just added.

Note that the key for the README.md file is the same as before. Since the file wasn't modified, it's contents are the same and so the hash of the contents are the same.

---

```bash
$ git cat-file -t 946086806911f33c726b8e4088bb3ea5a28ac7a2
blob
```
```bash
$ git cat-file -p 946086806911f33c726b8e4088bb3ea5a28ac7a2
# Understanding Git

> Achilleas Koutsou

2018-11-14
```

Note:
The third new object is the contents of slides.md, as expected.

---

## New information
- The new commit has a `parent`
- The new `tree` contains one new and one old object: Unchanged files don't create new objects

Note:
So what we've learned here is that the new commit has a `parent` reference, which points to the previous (first) commit. Also, the new tree is completely new: one of the two objects is the same as it was in the previous tree.

---

## Let's revise

- A revision (commit) consists of:
   - `parent`: A reference to a parent commit (the *previous* commit in history; except when it's the first commit)
   - `tree`: A reference to a tree (the top-level directory)
   - Metadata: Author, committer, message, date & time

Note:
A commit consists of a tree, a reference to a parent commit (unless it's the first commit), and a bunch of metadata. We wont talk much about the metadata since it's not very interesting. It's part of the content of the commit and it's used to compute the hash, but beyond that, it doesn't mean much for the Git internals.

---

- A tree consists of references to blobs and other trees
- A blob is simply the contents of a file

Note:
A tree is like a directory. It just references other trees and blobs. The filesystem names of trees and blobs, in other words, the directory and file names as they appear on the filesystem, are found in the *parent* tree. Filesystems work in very much the same way. Filenames are simply references to blocks on a storage device. The names themselves are part of a directory listing. The top level directory has no name; it's simply the root directory (both in a filesystem and in Git).

---

Third commit

```bash
$ echo "---

# Part 1
## Porcelain and Plumbing" >> slides.md
```
```bash
$ git add slides.md
```
```bash
$ git commit -m "Add second presentation slide"
[master f0d367a] Add second presentation slide
 1 file changed, 4 insertions(+)
```

Note:
Let's do one more commit. This time, we'll modify an existing file. We'll add a second presentation slide to slides.md

---

```bash
$ find .git/objects -type f
.git/objects/99/96248128ecd668be86434a05906e4edbbcc58e
.git/objects/3a/636d581e8d0ea6394d38ec214037000f454c4f
.git/objects/52/5a01c6209eab6d44bab1274f75763b57cfcd3f
.git/objects/ec/9d120bad321f68f257d730f893f1a9e8d96854
.git/objects/12/71443a51490d9457bbf0ccd05805d9948cdc2f
.git/objects/94/6086806911f33c726b8e4088bb3ea5a28ac7a2
.git/objects/82/7b7984168fe0833602c98c31dac93bd50facd8
.git/objects/10/9c6766058225867c46ebe79d1ff113ac7edf6d
.git/objects/6a/eb411c21381069e37981f0e97e1b6bd26f9ff5
```

3 new objects

Note:
Again, three new objects. I hope this isn't a surprise by now.

---

```bash
$ git cat-file -t 9996248128ecd668be86434a05906e4edbbcc58e
commit
```
```bash
$ git cat-file -p 9996248128ecd668be86434a05906e4edbbcc58e
tree 3a636d581e8d0ea6394d38ec214037000f454c4f
parent ec9d120bad321f68f257d730f893f1a9e8d96854
author Achilleas Koutsou <ak@example.com> 1542209858 +0100
committer Achilleas Koutsou <ak@example.com> 1542209858 +0100

Add second presentation slide
```

Note:
The new commit

---

```bash
$ git cat-file -t 3a636d581e8d0ea6394d38ec214037000f454c4f
tree
```
```bash
$ git cat-file -p 3a636d581e8d0ea6394d38ec214037000f454c4f
100644 blob 6aeb411c21381069e37981f0e97e1b6bd26f9ff5	README.md
100644 blob 525a01c6209eab6d44bab1274f75763b57cfcd3f	slides.md
```

Note:
The new tree.

Here we see something new. At first glance, the tree may appear to be the same as the old one, but the difference is in the key (the hash) of slides.md. It's the same filename, but it has different contents, since we added more text to it.

As far as Git is concerned, this is a completely new object. The fact that there are two entries in two different trees both called slides.md doesn't matter. It's just a symbolic name. The true key, or identifier of the content of the file is the hash.

---

```bash
$ git cat-file -t 525a01c6209eab6d44bab1274f75763b57cfcd3f
blob
```
```bash
$ git cat-file -p 525a01c6209eab6d44bab1274f75763b57cfcd3f
# Understanding Git

> Achilleas Koutsou

2018-11-14


# Part 1
## Porcelain and Plumbing
```

Note:
So if we check the contents of the new hash, we get the new file in its entirety.

---

We can still retrieve the first version of the file
```bash
$ git cat-file -p 946086806911f33c726b8e4088bb3ea5a28ac7a2
# Understanding Git

> Achilleas Koutsou

2018-11-14
```

Note:
The original version of the file is of course still available using its hash. The object is still available in the object store, it just isn't part of the working directory. In Git terms: the original blob is not referenced by the tree of the current commit.

---

# Part 3.1
## Hashes are not fit for human consumption

---

So far, we've been inspecting objects using their hashes. Nice for learning, but it's no way to live.

Note:
So far we've been using `cat-file` to check on objects in the data store. This is great for poking around Git's internals, since it brings us down to the core of the object store, but in normal usage, we rarely (if ever) use hashes directly.

---

Git provides symbolic names and shorthand for referencing objects

---

- Symbolic names: `refs`
  - `HEAD`: The current commit being worked on
  - branches (e.g., `master`): The most recent commit (head) of a given branch
  - tags (e.g., `v1.0`): A specific, named commit

Note:
`HEAD`: Refers to the currently *checked out* commit. *Checkout* is a Git (and other SCM) term and refers to changing the current working tree to a specific commit. A *checked out* revision refers to the commit currently being worked on.

Branches: By default, when we run `git init`, a reference is created called `master`. Every time a new commit is created, the `master` is updated to point to the new commit. We refer to this as a branch. A branch is simply a sequence of commits, starting from a HEAD commit (the latest in the branch) and going back through the parent references until the initial commit. In a bit, we'll see how other branches can be created and how that works. The name `master` holds no special meaning other than it being the conventional default.

Tags: At any time, we can tag a specific commit and give it a name. Much like a branch, it's a reference that points to a commit along with its implied history. The difference is that tags are never automatically updated to point to newer commits and in fact aren't meant to be changed.

---

- Referencing ancestors to a commit
  - `<ref>~<n>`: The `<n>th` ancestor of the named reference (e.g., `HEAD~3` the third ancestor of `HEAD`)

Note:
The tilde can be used to refer to *ancestors* of a commit (parents of parents, up the sequence of revisions).

---

- Tree and blob references
  - `<ref>:<path>` (e.g., `master:README.md`)
  - Can be combined with above to refer to objects from other revisions
    - `master~2:slides.md`
  - The root tree of a commit can be referenced by omitting the `<path>`
    - `master:`

Note:
If we append a colon to a ref, we start referencing the other objects, trees and blobs. We can reference them by their directory or filename directly. This is unambiguous since it refers to the version of a file in a specific commit.

These can all be combined arbitrarily. We could even use the hashes for any of the objects, the notation would be the same.

---
```bash
$ git cat-file -t HEAD
commit
```
```bash
$ git cat-file -t master
commit
```
```bash
$ git cat-file -p master
tree 3a636d581e8d0ea6394d38ec214037000f454c4f
parent ec9d120bad321f68f257d730f893f1a9e8d96854
author Achilleas Koutsou <ak@example.com> 1542209858 +0100
committer Achilleas Koutsou <ak@example.com> 1542209858 +0100

Add second presentation slide
```

Note:
Some examples

Head and master are both commits. In fact they're currently both the same commit, the most recent one, where we added the second presentation slide.

---
```bash
$ git cat-file -t master~1
commit
```
```bash
$ git cat-file -t master~1:
tree
```
```bash
$ git cat-file -p master~1:slides.md
# Understanding Git

> Achilleas Koutsou

2018-11-14
```

Note:
`master~1` is also a commit. If we add a colon to the end, it becomes a reference to the root tree.

`master~1:slides.md` refers to the file names slides.md in the parent commit of master, which is the previous version of slides.md, before we added the second slide.

---

```bash
$ git cat-file -p HEAD~1:README.md
# Understanding Git: Presentation slides repository
```
```bash
$ git cat-file -p HEAD:README.md
# Understanding Git: Presentation slides repository
```

Note:
Since README hasn't changed (since the first commit), both the current and previous revisions show the same content

Keep in mind that we're not asking for a previous version of a file. We're asking for the version of a file that was part of a given snapshot (commit).

---

- New plumbing command: `rev-parse`
  - Used internally for parsing flags and refs

Note:
`rev-parse` is a pretty powerful plumbing command. Internally, it's used to parse arguments and references.

We can use it for just one of its features: getting the object hash from a ref

---

```bash
$ git rev-parse HEAD
9996248128ecd668be86434a05906e4edbbcc58e
```

Translates symbolic name `HEAD` to the `SHA-1` key

---

```bash
$ git rev-parse HEAD
9996248128ecd668be86434a05906e4edbbcc58e
```
```bash
$ git rev-parse master
9996248128ecd668be86434a05906e4edbbcc58e
```
```bash
$ git cat-file -p HEAD
tree 3a636d581e8d0ea6394d38ec214037000f454c4f
parent ec9d120bad321f68f257d730f893f1a9e8d96854
author Achilleas Koutsou <ak@example.com> 1542209858 +0100
committer Achilleas Koutsou <ak@example.com> 1542209858 +0100

Add second presentation slide
```

```bash
$ git rev-parse HEAD~1
ec9d120bad321f68f257d730f893f1a9e8d96854
```

Note:
Let's use `rev-parse` for a quick revision

HEAD and master (currently) refer to the same revision. The latest commit we made.

A commit holds a reference to its parent. We can easily get a reference to this parent with the `~1` shorthand.

---

```bash
$ git cat-file -p master~1:
100644 blob 6aeb411c21381069e37981f0e97e1b6bd26f9ff5	README.md
100644 blob 946086806911f33c726b8e4088bb3ea5a28ac7a2	slides.md
```
```bash
$ git rev-parse master~1:README.md
6aeb411c21381069e37981f0e97e1b6bd26f9ff5
```

Note:
Appending colon to a ref refers to a tree or blob in that revision/commit.

We can also rev-parse references to blobs or trees to get their hash.

---

The current state of our repository

```bash
          .----------.      .----------.      .----------.
          | c0       | <--- | c1       | <--- | c2       |
          |----------|      |----------|      |----------|
          |          |      | par:  c0 |      | par:  c1 |
          | tree: t0 |      | tree: t1 |      | tree: t2 |
          ·----------·      ·----------·      ·----------·
```

Quiz time! What is the output of:
1. `git rev-parse c1:`
2. `git rev-parse c2~2`
3. `git cat-file -t c2:`

Note:
I hope you enjoy ASCII art

This is a representation of the current state of our repository. Let's pretend c0 - c2 are hashes for commits and t0 - t2 are hashes for trees.

See if you can predict the output of the tree commands.

---

The current state of our repository

```bash
          .----------.      .----------.      .----------.
          | c0       | <--- | c1       | <--- | c2       |
          |----------|      |----------|      |----------|
          |          |      | par:  c0 |      | par:  c1 |
          | tree: t0 |      | tree: t1 |      | tree: t2 |
          ·----------·      ·----------·      ·----------·
```

Quiz time! What is the output of:
1. `git rev-parse c1:` — t1
2. `git rev-parse c2~2` — c0
3. `git cat-file -t c2:` — tree

Note:
`c1:` is the root tree of the second commit, `t1`
`c2~2` is the second ancestor (parent of parent) of `c2`, `c0`
`c2~1:` asks for the type of the root tree of the parent of `c2`, which is a tree

---

Trees and blobs

```bash
   .----------------.    .----------------.    .----------------.
   | t0             |    | t1             |    | t2             |
   |----------------|    |----------------|    |----------------|
   | README.md: b0  |    | README.md: b0  |    | README.md: b0  |
   |                |    | slides.md: b1  |    | slides.md: b2  |
   ·----------------·    ·----------------·    ·----------------·
```

Note:
An illustration of what our trees look like so far. Again, let's pretend that b0 - b2 are hashes for blobs. Note that README.md always points to the same object, b0.

---

Commits point to a parent

Commits have a sequence, an order

Note:
Since each commit points to its parent, commits have an order, a sequence. The reference order goes from most recent to the initial commit.

---

But a repository's commits don't form a chain, they form a graph

Note:
But commits in Git don't just specify a chain. In fact they're a graph.

---

# Part 3.2
## Branches

Note:
Branching is an important feature of version control. A history of commits is nice, but simply going back and forward in time isn't that interesting. If you're collaborating on a project, you want multiple people editing things in a repository simultaneously. That's where branches come in.

---

```bash
$ git branch demo-code
```

Note:
In its simple form, `git branch <branchname>` creates a new branch at the current `HEAD`.

Remember, `master` is a branch. It's the default name for the main branch.

After this command, we have a new branch called `demo-code`. We're going to use this branch to write some demonstration code for our talk. We're using a branch because we don't want to affect the main branch until we're happy with the code. We might decide not to keep the code for instance, or maybe we plan to collaborate with someone else and one person will be making the slides while someone else writes the demo.

---

```bash
          .----------.      .----------.      .----------.
          | c0       | <--- | c1       | <--- | c2       |
          |----------|      |----------|      |----------|
          |          |      | par:  c0 |      | par:  c1 |
          | tree: t0 |      | tree: t1 |      | tree: t2 |
          ·----------·      ·----------·      ·----------·
                                                    |
                                                    |
                                                   HEAD
                                                   master
                                                   demo-code
```

Note:
We now have three symbolic names that all refer to `c2`. HEAD is the currently checked out commit, master is the head of the master branch, and demo-code points to the head of the demo-code branch, which we just created on HEAD.

---

```bash
$ git rev-parse master demo-code HEAD
9996248128ecd668be86434a05906e4edbbcc58e
9996248128ecd668be86434a05906e4edbbcc58e
9996248128ecd668be86434a05906e4edbbcc58e
```

Note:
If we inspect all three references using rev-parse, we can verify that they are indeed the same commit.

---

Switch to the new branch
```bash
$ git checkout demo-code
```

Note:
When we create a branch, the active branch doesn't change. We need to *checkout* the new branch to make it active.

---

Add the script and commit
```bash
$ echo "#!/usr/bin/env bash" > demoscript
```
```bash
$ chmod +x demoscript
```
```bash
$ git add demoscript
```
```bash
$ git commit -m "Add demoscript"
[demo-code aa59879] Add demoscript
 1 file changed, 1 insertion(+)
 create mode 100755 demoscript
```
---

```bash
.----------.      .----------.      .----------.      .----------.
| c0       | <--- | c1       | <--- | c2       | <--- | c3       |
|----------|      |----------|      |----------|      |----------|
|          |      | par:  c0 |      | par:  c1 |      | par:  c2 |
| tree: t0 |      | tree: t1 |      | tree: t2 |      | tree: t3 |
·----------·      ·----------·      ·----------·      ·----------·
                                          |                 |
                                          |                 |
                                         master            HEAD
                                                           demo-code
```

Note:
With the new commit, we've created a new snapshot on the active branch `demo-code`. The head of the branch, as well as the working HEAD have been moved to point to this new commit. `master` is right where we left it, pointing to c2.

---

```bash
$ git rev-parse master
9996248128ecd668be86434a05906e4edbbcc58e
```
```bash
$ git rev-parse demo-code
d93ab342e3f19e2551605c2bee760d68b1115760
```
```bash
$ git rev-parse HEAD
d93ab342e3f19e2551605c2bee760d68b1115760
```

Note:
Let's again verify using rev-parse to see our object hashes

---

Where's the graph?

Note:
Our revision sequence still looks like a flat chain. I promised to show you a graph. So let's check that out.

---

Move `HEAD` to point to `master`
```bash
$ git checkout master
```
```bash
$ git rev-parse master
9996248128ecd668be86434a05906e4edbbcc58e
```
```bash
$ git rev-parse demo-code
d93ab342e3f19e2551605c2bee760d68b1115760
```
```bash
$ git rev-parse HEAD
9996248128ecd668be86434a05906e4edbbcc58e
```

---
```bash
.----------.      .----------.      .----------.      .----------.
| c0       | <--- | c1       | <--- | c2       | <--- | c3       |
|----------|      |----------|      |----------|      |----------|
|          |      | par:  c0 |      | par:  c1 |      | par:  c2 |
| tree: t0 |      | tree: t1 |      | tree: t2 |      | tree: t3 |
·----------·      ·----------·      ·----------·      ·----------·
                                          |                 |
                                          |                 |
                                         HEAD              demo-code
                                         master
```
---

```bash
$ ls
README.md
slides.md
```

Note:
Checkout doesn't *just* move the HEAD. It's true purpose is *checking out* the tree of a given revision.

So now our working directory is the tree of `master`, `c2`, the commit before we introduced the demo script. `demoscript` is removed, because it doesn't exist in `c2`.

Note that git will never remove a file it doesn't know from your working directory. It wont delete a file that wasn't added and it wont delete or modify a tracked file that has uncommitted changes.

---

```bash
$ echo -e "\n\n# Part 2\n## What is Git?" >> slides.md
```
```bash
$ git add slides.md
```
```bash
$ git commit -m "Add third presentation slide"
[master b90a284] Add third presentation slide
 1 file changed, 4 insertions(+)
```

Note:
We continue working on the presentation. We add a third slide to slides.md and commit.

Let's discuss what the revision history will look like.

---

```bash
                                             master, HEAD
                                              |
                                          .----------.
     .----------.                         | c4       |
<--- | c2       | <-----------------------|----------|
     |----------|     |                   | par:  c2 |
     | par:  c1 |     |     .----------.  | tree: t4 |
     | tree: t2 |     `-----| c3       |  ·----------·
     ·----------·           |----------|
                            | par:  c2 |
                            | tree: t3 |
                            ·----------·
                                  |
                                 demo-code
```

Note:
Things are starting to look like a graph now.

The new interesting thing to note now is that c3 and c4 both have the same parent, c2.

It should also be clearer now why branches are called branches.

---

```bash
$ git rev-parse master~1 demo-code~1
9996248128ecd668be86434a05906e4edbbcc58e
9996248128ecd668be86434a05906e4edbbcc58e
```

Note:
Let's verify our diagram

---

# Part 3.2
## The Log

---

`git log` is your tool for exploring the commit history

Note:
git log is a powerful tool for exploring the commit history. It prints a sequence of commits, by default in reverse chronological order

---

```bash
$ git log -n 2
commit fe10223bade3ae70211810135c636ed4c9461410
Author: Achilleas Koutsou <ak@example.com>
Date:   Wed Nov 14 16:37:40 2018 +0100

    Add third presentation slide

commit 9996248128ecd668be86434a05906e4edbbcc58e
Author: Achilleas Koutsou <ak@example.com>
Date:   Wed Nov 14 16:37:38 2018 +0100

    Add second presentation slide
```

Note:
The -n flag limits the number of commits to show

By default, it starts from HEAD and goes back through parents

---

Formatting the Log

```bash
$ git log --pretty=format:"Commit: %h | Parent: %p | Tree: %t %n Message: %s%n"
Commit: fe10223 | Parent: 9996248 | Tree: 10b7222 
 Message: Add third presentation slide

Commit: 9996248 | Parent: ec9d120 | Tree: 3a636d5 
 Message: Add second presentation slide

Commit: ec9d120 | Parent: 827b798 | Tree: 1271443 
 Message: Add first presentation slide

Commit: 827b798 | Parent:  | Tree: 109c676 
 Message: Initial commit: Add README
```

Note:
We can pass formatting directives to add information like the parent

---

```bash
$ git log --graph --oneline master demo-code
* fe10223 Add third presentation slide
| * d93ab34 Add demoscript
|/  
* 9996248 Add second presentation slide
* ec9d120 Add first presentation slide
* 827b798 Initial commit: Add README
```

Note:
And with the graph and the oneline option, we can get a nice condensed view of the log graph

---

# Part 3.3
## Back to branches

---

Demo code is ready. Now what?

Note:
So let's fast forward and assume that the work we were doing in the demo code branch is done.

We're now ready to integrate it into the main part of the project.

---

```bash
$ git log --graph --decorate --oneline master demo-code
* fe10223 (master) Add third presentation slide
| * fde81d0 (HEAD -> demo-code) Finalising script
| * 29454af Writing script
| * d93ab34 Add demoscript
|/  
* 9996248 Add second presentation slide
* ec9d120 Add first presentation slide
* 827b798 Initial commit: Add README
```

Note:
This is the current state of our repository. The decorate flag adds the names of refs like HEAD and branch names to the log.

So the demo-code branch has a few commits in it and we want to bring those into master

---

`git merge <branch>` joins multiple development histories together

Note:
The merge command brings two or more branches together.

It replays the changes that were made in the named branch since the branch point and then creates a *merge commit* that incorporates those changes into the base branch (the branch we're running the command from).

---

```bash
$ git checkout master
Switched to branch 'master'
```
```bash
$ git merge demo-code
Merge made by the 'recursive' strategy.
 demoscript | 3 +++
 1 file changed, 3 insertions(+)
 create mode 100755 demoscript
```

Note:
To merge the demo code branch into master, we first switch to master since we want it to be our base branch, and then we git merge the code branch.

---

```bash
$ git log --graph --decorate --oneline master demo-code
*   6f9ec6e (HEAD -> master) Merge branch 'demo-code'
|\  
| * fde81d0 (demo-code) Finalising script
| * 29454af Writing script
| * d93ab34 Add demoscript
* | fe10223 Add third presentation slide
|/  
* 9996248 Add second presentation slide
* ec9d120 Add first presentation slide
* 827b798 Initial commit: Add README
```

Note:
If we look at our graph now for the commit history, we can see the two branches merging

---

```bash
$ git log --oneline master
6f9ec6e Merge branch 'demo-code'
fe10223 Add third presentation slide
fde81d0 Finalising script
29454af Writing script
d93ab34 Add demoscript
9996248 Add second presentation slide
ec9d120 Add first presentation slide
827b798 Initial commit: Add README
```

Note:
Note that if we simply look at master branch on its own, the changes that were made in the code branch are now part of master's history, as well as the one commit we made on the master branch after the branching point.

---

The merge commit

```bash
$ git cat-file -p HEAD
tree 276ee42784728b06ab2a29cd9b9c4a9a5780532c
parent fe10223bade3ae70211810135c636ed4c9461410
parent fde81d040de70c84f8661148fba797ac0ffe266f
author Achilleas Koutsou <ak@example.com> 1542209860 +0100
committer Achilleas Koutsou <ak@example.com> 1542209860 +0100

Merge branch 'demo-code'
```

Note:
This is what the merge commit looks like. Notice anything interesting?

It has two parents. This was expected, since it merged two branches.

So we learned something new again about commits: They don't always have just one parent. In fact they can have any number of parents.

Fun bit of trivia: In the Linux Kernel history, there are many commits with dozens of parents. The biggest one I know of has 66 parents!

https://www.destroyallsoftware.com/blog/2017/the-biggest-and-weirdest-commits-in-linux-kernel-git-history

---

# Part 4
## Situational Awareness

Note:
Many times we find ourselves in situations where we want to experiment with some code, or maybe check if we can merge some branches, or we want to go looking for a specific version of a file.

Given everything we learned so far, I want to talk about some ways in which you can avoid losing data, whether that's file contents, or revision history.

---

Which branch am I on?
- `git branch`
    - With no arguments, shows a list of branches and highlights the current one

Note:
Always keep in mind which branch you're on

---

```bash
$ git branch
  demo-code
* master
```

---

What does the revision graph look like?
- `git log --all --graph`
    - The `--all` flag will show all branches and refs

Note:
Check what's going on with respect to other branches

---

```bash
$ git log --all --graph
*   commit 6f9ec6ebfdc1423d0071a7520607a1d9b09c761c
|\  Merge: fe10223 fde81d0
| | Author: Achilleas Koutsou <ak@example.com>
| | Date:   Wed Nov 14 16:37:40 2018 +0100
| | 
| |     Merge branch 'demo-code'
| | 
| * commit fde81d040de70c84f8661148fba797ac0ffe266f
| | Author: Achilleas Koutsou <ak@example.com>
| | Date:   Wed Nov 14 16:37:40 2018 +0100
| | 
| |     Finalising script
| | 
| * commit 29454af9635f9d280507eaba2b0866520cf73b45
| | Author: Achilleas Koutsou <ak@example.com>
| | Date:   Wed Nov 14 16:37:40 2018 +0100
| | 
| |     Writing script
| | 
| * commit d93ab342e3f19e2551605c2bee760d68b1115760
| | Author: Achilleas Koutsou <ak@example.com>
| | Date:   Wed Nov 14 16:37:39 2018 +0100
| | 
| |     Add demoscript
| | 
* | commit fe10223bade3ae70211810135c636ed4c9461410
|/  Author: Achilleas Koutsou <ak@example.com>
|   Date:   Wed Nov 14 16:37:40 2018 +0100
|   
|       Add third presentation slide
| 
* commit 9996248128ecd668be86434a05906e4edbbcc58e
| Author: Achilleas Koutsou <ak@example.com>
| Date:   Wed Nov 14 16:37:38 2018 +0100
| 
|     Add second presentation slide
| 
* commit ec9d120bad321f68f257d730f893f1a9e8d96854
| Author: Achilleas Koutsou <ak@example.com>
| Date:   Wed Nov 14 16:37:37 2018 +0100
| 
|     Add first presentation slide
| 
* commit 827b7984168fe0833602c98c31dac93bd50facd8
  Author: Achilleas Koutsou <ak@example.com>
  Date:   Wed Nov 14 16:37:37 2018 +0100
  
      Initial commit: Add README
```

---

Unsaved changes?
- `git status`
  - Shows the current branch and files with uncommitted changes
- `git diff`
  - Shows the uncommitted changes in patch format (additions and removals of lines)

---

```bash
$ echo "See slides.md for slides" >> README.md
```
```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

---

```bash
$ git diff
diff --git a/README.md b/README.md
index 6aeb411..2f1af42 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
 # Understanding Git: Presentation slides repository
+See slides.md for slides
```

---

- How your current branch differs from another (e.g., `master`)
  - `git diff master`
    - Shows how the content of the files in the current working tree differ from the master tree
  - `git diff --name-only master`
    - List the names of the files that differ

---

```bash
$ git diff demo-code
diff --git a/README.md b/README.md
index 6aeb411..2f1af42 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
 # Understanding Git: Presentation slides repository
+See slides.md for slides
diff --git a/slides.md b/slides.md
index 525a01c..036f63b 100644
--- a/slides.md
+++ b/slides.md
@@ -7,3 +7,7 @@
 
 # Part 1
 ## Porcelain and Plumbing
+
+
+# Part 2
+## What is Git?
```

---

## Packs and Deltas

Note:
Remember when we said that git KINDA keeps track of changes?

We've seen so far that git keeps snapshots of the tree every time we commit. If we inspect each version of the README, for instance, we see that each object is the full text of the file.

---

```bash
$ git cat-file -s HEAD:README.md
77
```
```bash
$ git cat-file -p HEAD:README.md
# Understanding Git: Presentation slides repository
See slides.md for slides
```

```bash
$ git cat-file -s HEAD~1:README.md
52
```
```bash
$ git cat-file -p HEAD~1:README.md
# Understanding Git: Presentation slides repository
```

Note:
The two versions of README are the full file. We can even see it from the size. This can become wasteful when projects grow to thousands lines of code. It probably wont create a storage problem, but it's certainly inefficient.

---

`git gc`

- Cleanup unnecessary files and optimize the local repository

---

```bash
$ find .git/objects -type f
.git/objects/58/57b8574a524a0d69fe85cc15dfe4875caf699b
.git/objects/86/968948ebcad93c6f01e1282a28d8e149b50be9
.git/objects/2f/1af4274b97d1909668d51c8a67ec1f5b03645f
.git/objects/6f/9ec6ebfdc1423d0071a7520607a1d9b09c761c
.git/objects/27/6ee42784728b06ab2a29cd9b9c4a9a5780532c
.git/objects/fd/e81d040de70c84f8661148fba797ac0ffe266f
.git/objects/6b/96c395390b4571635f518fec03c6bc7195adea
.git/objects/7f/af35404250750545bb376ae09d7a57a4a19304
.git/objects/a9/98cdf0a62dd456742892c3ad800524565f775d
.git/objects/56/cf4980fd48882b281cb29564aef64e99a8a9fd
.git/objects/fe/10223bade3ae70211810135c636ed4c9461410
.git/objects/03/6f63b2cdbb77e5e964fcc1ca9d993d548b15fa
.git/objects/d9/3ab342e3f19e2551605c2bee760d68b1115760
.git/objects/29/454af9635f9d280507eaba2b0866520cf73b45
.git/objects/29/f3338460c2def04cc34c9e46457fcd7a40b371
.git/objects/f1/f641af19bf623505554b29f35499b5d15f3fd3
.git/objects/99/96248128ecd668be86434a05906e4edbbcc58e
.git/objects/3a/636d581e8d0ea6394d38ec214037000f454c4f
.git/objects/52/5a01c6209eab6d44bab1274f75763b57cfcd3f
.git/objects/ec/9d120bad321f68f257d730f893f1a9e8d96854
.git/objects/12/71443a51490d9457bbf0ccd05805d9948cdc2f
.git/objects/94/6086806911f33c726b8e4088bb3ea5a28ac7a2
.git/objects/82/7b7984168fe0833602c98c31dac93bd50facd8
.git/objects/10/b72226f7e5c353447e78088f8a53bfd5c7e765
.git/objects/10/9c6766058225867c46ebe79d1ff113ac7edf6d
.git/objects/6a/eb411c21381069e37981f0e97e1b6bd26f9ff5
```

```bash
$ du -s .git/objects
104	.git/objects
```

Note:
Let's have a look at the object store as it is now

---

```bash
$ git gc
```
```bash
$ find .git/objects -type f
.git/objects/info/packs
.git/objects/pack/pack-bb9e6fb51a758739992e71a094075bfab17ae0ab.idx
.git/objects/pack/pack-bb9e6fb51a758739992e71a094075bfab17ae0ab.pack
```
```bash
$ du -s .git/objects
12	.git/objects
```

Note:
Run the gc command (garbage collector) to optimise the storage and check the store again

---

```bash
$ git verify-pack -v .git/objects/pack/pack-bb9e6fb51a758739992e71a094075bfab17ae0ab.pack
5857b8574a524a0d69fe85cc15dfe4875caf699b commit 241 170 12
fde81d040de70c84f8661148fba797ac0ffe266f commit 234 162 182
fe10223bade3ae70211810135c636ed4c9461410 commit 245 168 344
29454af9635f9d280507eaba2b0866520cf73b45 commit 231 160 512
6f9ec6ebfdc1423d0071a7520607a1d9b09c761c commit 289 195 672
d93ab342e3f19e2551605c2bee760d68b1115760 commit 85 97 867 1 fe10223bade3ae70211810135c636ed4c9461410
9996248128ecd668be86434a05906e4edbbcc58e commit 246 168 964
827b7984168fe0833602c98c31dac93bd50facd8 commit 195 140 1132
ec9d120bad321f68f257d730f893f1a9e8d96854 commit 245 169 1272
2f1af4274b97d1909668d51c8a67ec1f5b03645f blob   77 76 1441
7faf35404250750545bb376ae09d7a57a4a19304 blob   48 48 1517
036f63b2cdbb77e5e964fcc1ca9d993d548b15fa blob   117 112 1565
86968948ebcad93c6f01e1282a28d8e149b50be9 tree   112 119 1677
6b96c395390b4571635f518fec03c6bc7195adea tree   112 119 1796
6aeb411c21381069e37981f0e97e1b6bd26f9ff5 blob   4 15 1915 1 2f1af4274b97d1909668d51c8a67ec1f5b03645f
525a01c6209eab6d44bab1274f75763b57cfcd3f blob   4 15 1930 1 036f63b2cdbb77e5e964fcc1ca9d993d548b15fa
10b72226f7e5c353447e78088f8a53bfd5c7e765 tree   74 80 1945
a998cdf0a62dd456742892c3ad800524565f775d tree   112 119 2025
56cf4980fd48882b281cb29564aef64e99a8a9fd blob   34 44 2144
276ee42784728b06ab2a29cd9b9c4a9a5780532c tree   25 38 2188 1 6b96c395390b4571635f518fec03c6bc7195adea
29f3338460c2def04cc34c9e46457fcd7a40b371 tree   112 119 2226
f1f641af19bf623505554b29f35499b5d15f3fd3 blob   20 30 2345
3a636d581e8d0ea6394d38ec214037000f454c4f tree   74 79 2375
109c6766058225867c46ebe79d1ff113ac7edf6d tree   37 48 2454
1271443a51490d9457bbf0ccd05805d9948cdc2f tree   74 79 2502
946086806911f33c726b8e4088bb3ea5a28ac7a2 blob   4 15 2581 1 036f63b2cdbb77e5e964fcc1ca9d993d548b15fa
non delta: 21 objects
chain length = 1: 5 objects
.git/objects/pack/pack-bb9e6fb51a758739992e71a094075bfab17ae0ab.pack: ok
```

---

This is done automatically on occasion, so you don't need to worry about running `gc` manually.

---

# EOF

This talk and accompanying material will be available at

`https://github.com/achilleas-k/understanding-git`
