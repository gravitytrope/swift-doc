# swift-doc

![CI][ci badge]

A package for generating documentation for Swift projects.

**This project is under active development
and is expected to change significantly before its first stable release.**

Given a directory of Swift files,
`swift-doc` generates CommonMark (Markdown) files
for each class, structure, enumeration, and protocol
as well as top-level type aliases, functions, and variables.

For an example of generated documentation,
[check out the Wiki for our fork of Alamofire][alamofire wiki].

> **Note**:
> Output is currently limited to CommonMark,
> but the plan is to support HTML and other formats as well.

## Requirements

- Swift 5.2

## Command-Line Utility

`swift-doc` can be used from the command-line on macOS and Linux.

### Installation

#### Homebrew

Run the following command to install using [Homebrew](https://brew.sh/):

```terminal
$ brew install swiftdocorg/formulae/swift-doc
```

#### Manually

Run the following commands to build and install manually:

```terminal
$ git clone https://github.com/SwiftDocOrg/swift-doc
$ cd swift-doc
$ make install
```

### Usage

`swift-doc` takes one or more paths and enumerates them recursively,
collecting all Swift files into a single "module"
and generating documentation accordingly.

```terminal
$ swift doc generate path/to/SwiftProject/Sources --output Documentation
$ tree Documentation
$ Documentation/
├── Home
├── (...)
├── _Footer.md
└── _Sidebar.md
```

By default,
output files are written to `.build/documentation`,
but you can change that with the `--output` option flag.

#### swift-doc coverage

The `coverage` subcommand 
generates documentation coverage statistics for Swift files.

```terminal
$ git clone https://github.com/SwiftDocOrg/SwiftSemantics.git

$ swift run swift-doc coverage SwiftSemantics/Sources/ --output "dcov.json" 
$ cat dcov.json | jq ".data.totals"
{
  "count": 207,
  "documented": 199,
  "percent": 96.1352657004831
}

$ cat dcov.json | jq ".data.symbols[] | select(.documented == false)"
{
  "file": "SwiftSemantics/Supporting Types/GenericRequirement.swift",
  "line": 67,
  "column": 6,
  "name": "GenericRequirement.init?(_:)",
  "type": "Initializer",
  "documented": false
}
...
```

While there are plenty of tools for assessing test coverage for code,
we weren't able to find anything analogous for documentation coverage.
To this end,
we've contrived a simple JSON format
[inspired by llvm-cov](https://reviews.llvm.org/D22651#change-xdePaVfBugps).

If you know of an existing standard
that you think might be better suited for this purpose,
please reach out by [opening an Issue][open an issue]!

#### swift-doc diagram

The `diagram` subcommand
generates a graph of APIs in [DOT format][dot]
that can be rendered by [GraphViz][graphviz] into a diagram.

```terminal
$ swift run swift-doc diagram Alamofire/Source > graph.dot
$ head graph.dot
digraph Anonymous {
  "Session" [shape=box];
  "NetworkReachabilityManager" [shape=box];
  "URLEncodedFormEncoder" [shape=box,peripheries=2];
  "ServerTrustManager" [shape=box];
  "MultipartFormData" [shape=box];

  subgraph cluster_Request {
    "DataRequest" [shape=box];
    "Request" [shape=box];

$ dot -T svg graph.dot > graph.svg
```

Here's an excerpt of the graph generated for Alamofire:

![Excerpt of swift-doc-api Diagram for Alamofire](https://user-images.githubusercontent.com/7659/73189318-0db0e880-40d9-11ea-8895-341a75ce873c.png)

## GitHub Action

This repository also hosts a [GitHub Action][github actions]
that you can incorporate into your project's workflow.

The CommonMark files generated by `swift-doc`
are formatted for publication to your project's [GitHub Wiki][github wiki],
which you can do with
[github-wiki-publish-action][github-wiki-publish-action].
Alternatively,
you could publish `swift-doc`-generated documentation to GitHub Pages,
or bundle them into a release artifact.

### Inputs

- `inputs`:
  One or more paths to Swift files in your workspace.
  (Default: `"./Sources"`)
- `output`:
  The path for generated output.
  (Default: `"./.build/documentation"`)

### Example Workflow

```yml
# .github/workflows/documentation.yml
name: Documentation

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v1
      - name: Generate Documentation
        uses: SwiftDocOrg/swift-doc@master
        with:
          inputs: "Source"
          output: "Documentation"
      - name: Upload Documentation to Wiki
        uses: SwiftDocOrg/github-wiki-publish-action@v1
        with:
          path: "Documentation"
        env:
          GITHUB_PERSONAL_ACCESS_TOKEN: ${{ secrets.GITHUB_PERSONAL_ACCESS_TOKEN }}
```

* * *

## Motivation

From its earliest days,
Swift has been fortunate to have [Jazzy][jazzy],
which is a fantastic tool for generating documentation
for both Swift and Objective-C projects.
Over time, however,
the way we write Swift code —
and indeed the language itself —
has evolved to incorporate patterns and features
that are difficult to understand using
the same documentation standards that served us well for Objective-C.

Whereas in Objective-C,
you could get a complete view of a type's functionality from its class hierarchy,
Swift code today tends to layer and distribute functionality across
[a network of types][swift number protocols diagram].
While adopting a
[protocol-oriented paradigm][protocol-oriented programming]
can make Swift easier and more expressive to write,
it can also make Swift code more difficult to understand.

Our primary goal for `swift-doc`
is to make Swift documentation more useful
by surfacing the information you need to understand how an API works
and presenting it in a way that can be easily searched and accessed.
We want developers to be empowered to use Swift packages to their full extent,
without being reliant on (often outdated) blog posts or Stack Overflow threads.
We want documentation coverage to become as important as test coverage:
a valuable metric for code quality,
and an expected part of first-rate open source projects.

Jazzy styles itself after Apple's official documentation circa 2014
(code-named "Jazz", as it were),
which was well-suited to understanding Swift code as we wrote it back then
when it was more similar to Objective-C.
But this design is less capable of documenting the behavior of
generically-constrained types,
default implementations,
[dynamic member lookup][se-0195],
[property wrappers][se-o258], or
[function builders][se-xxxx].
(Alas,
Apple's [most recent take][apple documentation] on reference documentation
hasn't improved matters,
having instead focused on perceived cosmetic issues.)

Without much in the way of strong design guidance,
we're not entirely sure what Swift documentation _should_ look like.
But we do think plain text is a good place to start.
We look forward to 
soliciting feedback and ideas from everyone 
so that we can identify those needs 
and figure out the best ways to meet them.

In the meantime,
we've set ourselves up for success
by investing in the kind of foundation we'll need
to build whatever we decide best solves the problems at hand.
`swift-doc` is built on top of a constellation of projects
that take advantage of modern infrastructure and tooling:

- [SwiftSemantics][swiftsemantics]:
  Parses Swift code into its constituent declarations
  using [SwiftSyntax][swiftsyntax]
- [SwiftMarkup][swiftmarkup]:
  Parses Swift documentation comments into structured entities
  using [CommonMark][commonmark]
- [github-wiki-publish-action][github-wiki-publish-action]:
  Publishes the contents of a directory to your project's wiki

These new technologies have already yielded some promising results.
`swift-doc` is built in Swift,
and can be installed on both macOS and Linux as a small, standalone binary.
Because it relies only on a syntactic reading of Swift source code,
without needing code first to be compiled,
`swift-doc` is quite fast.
As a baseline,
compare its performance to Jazzy
when generating documentation for [SwiftSemantics][swiftsemantics]:

```terminal
$ cd SwiftSemantics

$ time swift-doc Sources
        0.21 real         0.16 user         0.02 sys

$ time jazzy # fresh build
jam out ♪♫ to your fresh new docs in `docs`
       67.36 real        98.76 user         8.89 sys


$ time jazzy # with build cache
jam out ♪♫ to your fresh new docs in `docs`
       17.70 real         2.17 user         0.88 sys
```

Of course,
some of that is simply Jazzy doing more,
generating HTML, CSS, and a search index instead of just text.
Compare its [generated HTML output][jazzy swiftsemantics]
to [a GitHub wiki generated with `swift-doc`][swift-doc swiftsemantics].

## License

MIT

## Contact

Mattt ([@mattt](https://twitter.com/mattt))

[ci badge]: https://github.com/SwiftDocOrg/swift-doc/workflows/CI/badge.svg

[alamofire wiki]: https://github.com/SwiftDocOrg/Alamofire/wiki
[github wiki]: https://help.github.com/en/github/building-a-strong-community/about-wikis
[github actions]: https://github.com/features/actions
[swiftsyntax]: https://github.com/apple/swift-syntax
[swiftsemantics]: https://github.com/SwiftDocOrg/SwiftSemantics
[swiftmarkup]: https://github.com/SwiftDocOrg/SwiftMarkup
[commonmark]: https://github.com/SwiftDocOrg/CommonMark
[github-wiki-publish-action]: https://github.com/SwiftDocOrg/github-wiki-publish-action
[open an issue]: https://github.com/SwiftDocOrg/swift-doc/issues/new
[jazzy]: https://github.com/realm/jazzy
[swift number protocols diagram]: https://nshipster.com/propertywrapper/#swift-number-protocols
[protocol-oriented programming]: https://asciiwwdc.com/2015/sessions/408
[apple documentation]: https://developer.apple.com/documentation
[se-0195]: https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md
[se-o258]: https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md
[se-xxxx]: https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md
[swiftdoc.org]: https://swiftdoc.org
[jazzy swiftsemantics]: https://swift-semantics-jazzy.netlify.com
[swift-doc swiftsemantics]: https://github.com/SwiftDocOrg/SwiftSemantics/wiki
[@natecook1000]: https://github.com/natecook1000
[nshipster]: https://nshipster.com
[dependency hell]: https://github.com/apple/swift-package-manager/tree/master/Documentation#dependency-hell
[pcre]: https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions
[dot]: https://en.wikipedia.org/wiki/DOT_(graph_description_language)
[graphviz]: https://www.graphviz.org
