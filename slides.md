---
title: Understanding Git
theme: black
css: style.css
revealOptions:
  transition: 'none'

---

# Understanding Git

> Achilleas Koutsou

2018-11-14

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

If you're completely unfamiliar with Git, this talk will probably give you some basics to get started, but more importantly, it should give you enough knowledge so that if you go through a beginner's tutorial (of which there are plenty), it will be easier to *understand* the tutorial in more depth.

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

- Git is a key-value store: A database of objects where each object is addressable by its SHA-1 hash
- Every other feature is implemented by tools and functions that manipulate this database.

---

An example SHA-1 hash
```plaintext
32c65e86d72d50be78c536f79d8036604eb713b1
```

Short hash
```plaintext
32c65e8
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
.git/objects
├── 32/
│   └── c65e86d72d50be78c536f79d8036604eb713b1
├── 55/
│   └── 2e539f0238d49549d0f7be59f6b8aae99c57c3
├── dc/
│   └── d0c23a90b9a55df7caa0c5c182254ace2e0be4
├── info/
└── pack/
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
$ git cat-file -t 552e539f0238d49549d0f7be59f6b8aae99c57c3
commit
```
```bash
$ git cat-file -p 552e539f0238d49549d0f7be59f6b8aae99c57c3
tree dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
author Achilleas Koutsou <ak@example.com> 1542119518 +0100
committer Achilleas Koutsou <ak@example.com> 1542119518 +0100

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
$ git cat-file -t dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
tree
```
```bash
$ git cat-file -p dcd0c23a90b9a55df7caa0c5c182254ace2e0be4
100644 blob 32c65e86d72d50be78c536f79d8036604eb713b1    README.md
```

Note:
A `tree` is analogous to a filesystem directory. It contains a list of references to files: in this case, the README file we added to the commit with the `git add` command.

It can also contain references to other trees, the same way a directory can have subdirectories.

Finally, the third object we'll inspect again appears here: It's a reference to the file `README.md`.

---

```bash
$ git cat-file -t 32c65e86d72d50be78c536f79d8036604eb713b1
blob
```
```bash
$ git cat-file -p 32c65e86d72d50be78c536f79d8036604eb713b1
# Understanding Git
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
.git/objects/f1/5a0bca73aef6fa1abac7e574b8143d4164889e
.git/objects/38/781ad1a6eb9cd94c0db2bffabe71a9fc7084a1
.git/objects/94/6086806911f33c726b8e4088bb3ea5a28ac7a2
.git/objects/eb/4863188fedc82c6b3329a5b7955e0068be6d13
.git/objects/dc/d0c23a90b9a55df7caa0c5c182254ace2e0be4
.git/objects/32/c65e86d72d50be78c536f79d8036604eb713b1
```

3 new objects

Note:
There's 6 objects now. If we compare the hashes with the previous time we checked the object store, we'll see that three of the objects are the same. So three new objects were created. Knowing what we know about the state of the object store after the first commit, we can probably guess what the new objects are.

Let's have a look at the new files.

---

```bash
$ git cat-file -t 3730b6b0b056419d7bf73ecc1228e340958a0960
commit
```
```bash
$ git cat-file -p 3730b6b0b056419d7bf73ecc1228e340958a0960
tree 38781ad1a6eb9cd94c0db2bffabe71a9fc7084a1
parent f5c1d92ed327382286b4ebe498a15b32211f99b8
author Achilleas Koutsou <ak@example.com> 1542124625 +0100
committer Achilleas Koutsou <ak@example.com> 1542124625 +0100

Add first presentation slide
```

Note:
Just like last time, the first object is a `commit`. It's our new commit, where we added the presentation slide. Notice anything different?

This commit has one extra bit of information. It has a `parent`! I think we can all guess what that refers to.

If you look at the hash, it's the key of the first (initial) commit. Let's look at the other two new objects and then return to this.

---

```bash
$ git cat-file -t 38781ad1a6eb9cd94c0db2bffabe71a9fc7084a1
tree
```
```bash
$ git cat-file -p 38781ad1a6eb9cd94c0db2bffabe71a9fc7084a1
100644 blob 32c65e86d72d50be78c536f79d8036604eb713b1    README.md
100644 blob 946086806911f33c726b8e4088bb3ea5a28ac7a2    slides.md
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
.git/objects/12/4e8d9f4e78f2fb2e478bbfd57b29c23f04e854
.git/objects/37/1aab1c5585c1b426385128946f772d409a26f8
.git/objects/f5/a1828f86e3aeb2aad46ebb7e69144179849ae4
.git/objects/40/641b176048c07e570f72fae8e0bd5f9d1c4cbf
.git/objects/38/781ad1a6eb9cd94c0db2bffabe71a9fc7084a1
.git/objects/94/6086806911f33c726b8e4088bb3ea5a28ac7a2
.git/objects/eb/8152b977c2bf99528d40efe2e3094ca2b2524e
.git/objects/dc/d0c23a90b9a55df7caa0c5c182254ace2e0be4
.git/objects/32/c65e86d72d50be78c536f79d8036604eb713b1
```

3 new objects

Note:
Again, three new objects. I hope this isn't a surprise by now.

---

```bash
$ git cat-file -t f0d367ac211d2aeaaca5acb82ca5408a698284a5
commit
```

```bash
$ git cat-file -p f0d367ac211d2aeaaca5acb82ca5408a698284a5
tree 5e605f1c012c4da9064a8d7527eed85f4c6c1093
parent ee7c8085bac4e4cfc5a239a8ab1a0cade30ef780
author Achilleas Koutsou <ak@example.com> 1542132946 +0100
committer Achilleas Koutsou <ak@example.com> 1542132946 +0100

Add second presentation slide
```

Note:
The new commit

---

```bash
$ git cat-file -t 5e605f1c012c4da9064a8d7527eed85f4c6c1093
tree
```
```bash
$ git cat-file -p 5e605f1c012c4da9064a8d7527eed85f4c6c1093
100644 blob 32c65e86d72d50be78c536f79d8036604eb713b1	README.md
100644 blob e0097861ecddb9721b29d30e6c08838008fb4d0f	slides.md
```

Note:
The new tree.

Here we see something new. At first glance, the tree may appear to be the same as the old one, but the difference is in the key (the hash) of slides.md. It's the same filename, but it has different contents, since we added more text to it.

As far as Git is concerned, this is a completely new object. The fact that there are two entries in two different trees both called slides.md doesn't matter. It's just a symbolic name. The true key, or identifier of the content of the file is the hash.

---

```bash
$ git cat-file -t e0097861ecddb9721b29d30e6c08838008fb4d0f
blob
```
```bash
$ git cat-file -p e0097861ecddb9721b29d30e6c08838008fb4d0f
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
tree bc248760039ec1aaed7d70d3675a4ad990b6cbad
parent ec4aafb724e8df9431967825cf0d8aeb16618839
author Achilleas Koutsou <ak@example.com> 1542150852 +0100
committer Achilleas Koutsou <ak@example.com> 1542150852 +0100

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
10f21b8ddd8fa6ea61fc1e3bb72ae448c5129515
```

Translates symbolic name `HEAD` to the `SHA-1` key

---

```bash
$ git rev-parse HEAD
129245abafe22dc446e0fc99a8aa00c1aafb1637
```
```bash
$ git rev-parse master
129245abafe22dc446e0fc99a8aa00c1aafb1637
```
```bash
$ git cat-file -p HEAD
tree 3a636d581e8d0ea6394d38ec214037000f454c4f
parent d622069ae0d126de06a3a7add6eeed0cc02d3a8b
author Achilleas Koutsou <ak@example.com> 1542152437 +0100
committer Achilleas Koutsou <ak@example.com> 1542152437 +0100

Add second presentation slide
```

```bash
$ git rev-parse HEAD~1
d622069ae0d126de06a3a7add6eeed0cc02d3a8b
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

1. `git rev-parse c1:`
  - A: t1
2. `git rev-parse c2~2`
  - A: c0
3. `git cat-file -t c2~1:`
  - A: tree

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
0488c139d44019bc518d5dd8e1864dd31bb19aa1
0488c139d44019bc518d5dd8e1864dd31bb19aa1
0488c139d44019bc518d5dd8e1864dd31bb19aa1
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
d87d662d1ea13098debd6d1684a1b2745465d623
```
```bash
$ git rev-parse demo-code
3652cf7404b11479fb5f94df0b234b949001369c
```
```bash
$ git rev-parse HEAD
3652cf7404b11479fb5f94df0b234b949001369c
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
e20487a13b849606cdd5a1e55a7de913326bd8ba
```
```bash
$ git rev-parse demo-code
5d5cc9b75aa202f859f916ecc113e9917f212607
```
```bash
$ git rev-parse HEAD
e20487a13b849606cdd5a1e55a7de913326bd8ba
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
Thins are starting to look like a graph now.

The new interesting thing to note now is that c3 and c4 both have the same parent, c2.

It should also be clearer now why branches are called branches.

---

```bash
$ git rev-parse master~1 demo-code~1
0b7bfa715f8f2db121acec9c1e4f826edeb68529
0b7bfa715f8f2db121acec9c1e4f826edeb68529
```

Note:
Let's verify our diagram

---



---

# Part 3.2
## Detour: Packs and Deltas

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
