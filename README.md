# Understanding Git

This repository contains two presentations
- [Basics](#basics): A slideshow and demonstration script for Git basics, explaining file states and file state transitions.
- [Porcelain and plumbing](#porcelain-and-plumbing): A slideshow exploring Git internals with the use of git *plumbing* commands.

The [custom stylesheet](./style.css) in the root of the repository is used by both slide decks for theming tweaks.


## Basics

Basic Git concepts, file stages, and stage transitions.

### Presentation slides

Slides are written in markdown and were presented using [reveal-md](http://webpro.github.io/reveal-md/)
```bash
reveal-md ./basics/slides.md
```

### Demo

Script runs through file stages demonstrating, with comments and descriptions, how to transition between all file stages as shown in the slides (and in reverse)
```bash
./basics/git-states
```

Branch demo script `./basics/git-branches` is still a work in progress.

## Porcelain and Plumbing

Version control systems are regarded as essential tools for writing code.  Whether you're working alone or in a team, on small or large projects, version control is an invaluable tool for tracking code changes.  Git is, arguably, the most popular and perhaps the most important version control system, especially in the open source world.

While learning to use Git is important and most useful aspects of Git can be leveraged by basic workflows, understanding how Git works can help unlock its true power and avoid many pitfalls that come with more complicated workflows.

In this 90-minute talk, I hope to demystify the most important aspects of Git's internals.  We will begin by going through the simple internal structures that Git uses to track changes and follow with an explanation of how these internal components are used to build the basic Git functions that we all use.  Finally, we will review some basic, high-level Git workflows, from a lower-level perspective.

Note: No experience or knowledge of Git is assumed for this talk.

### Presentation slides

Slides are written in markdown and were presented using [reveal-md](http://webpro.github.io/reveal-md/)
```bash
reveal-md ./porcelain-and-plumbing/slides.md
```

### Notes

Some presentation notes can be found in the [notes file](./porcelain-and-plumbing/notes.md).  This includes notes and thoughts that were added to the presentation as well as ideas that were not included for time and complexity.  It may, in the future, turn into a more coherent accompanying text for the talk.

## Versions

- [v2.0](https://github.com/achilleas-k/understanding-git/tree/v2.0): Addition of new Basics presentation and demo.  *Work in progresss*
- [v1.0](https://github.com/achilleas-k/understanding-git/tree/v1.0): The slides in the form they were presented during the [Super Python Talk](https://www.meetup.com/SuperPythonTalks/events/256145090/) (14 Nov 2018).

The slides, notes, and script may be modified in the future. New versions will be tagged when appropriate.

## License

All content in this repository is licensed under the [Creative Commons Attribution Share Alike 4.0 International](./LICENSE) license.
