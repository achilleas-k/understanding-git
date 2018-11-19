# Understanding Git: Plumbing and Porcelain

Version control systems are regarded as essential tools for writing code. Whether you're working alone or in a team, on small or large projects, version control is an invaluable tool for tracking code changes. Git is, arguably, the most popular and perhaps the most important version control system, especially in the open source world.

While learning to use Git is important and most useful aspects of Git can be leveraged by basic workflows, understanding how Git works can help unlock its true power and avoid many pitfalls that come with more complicated workflows.

In this 90-minute talk, I hope to demystify the most important aspects of Git's internals. We will begin by going through the simple internal structures that Git uses to track changes and follow with an explanation of how these internal components are used to build the basic Git functions that we all use. Finally, we will review some basic, high-level Git workflows, from a lower-level perspective.

Note: No experience or knowledge of Git is assumed for this talk.

## Presentation slides

Slides are written in markdown and were presented using [reveal-md](http://webpro.github.io/reveal-md/)
```bash
reveal-md slides.md
```

The header in [slides file](./slides.md) sets the black theme and sets a few modifications found in the [custom stylesheet](./style.css) by default.

## Notes

Some presentation notes can be found in the [notes file](./notes.md). This includes notes and thoughts that were added to the presentation as well as ideas that were not included for time and complexity. It may, in the future, turn into a more coherent accompanying text for the talk.

## Versions

- [v1.0](85d543dfd0239d9a22e7d11f3d6bf9da22b5ae13): The slides in the form they were presented during the [Super Python Talk](https://www.meetup.com/SuperPythonTalks/events/256145090/) (14 Nov 2018).

The slides, notes, and script may be modified in the future. New versions will be tagged when appropriate.

## License

All content in this repository is licensed under the [Creative Commons Attribution Share Alike 4.0 International](./LICENSE) license.
