---
Start Date: 2020-07-31
RFC PR: (leave this empty)
Foam Issue: (leave this empty)
---

# 0002 Materialized Links

## Summary

In Foam, you can link between documents using `[[wiki-link]]` syntax. We parse all the markdown files (notes) in your Foam workspace, and create a graph data structure (not to be confused with the graph visualization) that links all the documents together. Each note has two kinds of links:

- Forward links (links from that note to other notes)
- Backlinks (links from other notes to that note)

"Materialized links" is a Foam term for outputting the links from and to a document into the document itself. For benefits and use cases, see [Motivation](#motivation).

## Basic example

Assume a following three markdown documents:

`note-a.md`
```md
# Note A
This is a link to [[note-b]].
```

`note-b.md`
```md
# Note B
This links back to [[note-a]].
```

`note-c.md`
```md
# Note C
This links to [[note-a]].
```

If we generated both Forward links and Backlinks for `note-a.md`, it would look like follows:

```md
# Note A
This is a link to [[note-b]].

## Foam:Links
- [Note B](note-b)

## Foam:Backlinks
- [Note B](note-b)
- [Note C](note-c)
```

Note the following:

- 

## Motivation

### Backlinks

You can already view backlinks inside the Markdown Notes backlinks explorer. Ma

- Including the backlink information for benefit of other tools that do not have access to the Foam graph.
- Used to generate reference lists
  - E.g. "todo.md" can show every place `[[todo]]` is mentioned
- Useful for publishing static websites, PDFs etc from your workspace and including backlink information.
  - Note that a future [Foam static site generator](https://github.com/foambubble/rfcs/pull/2) may generate alternative representations of the backlinks, in which case materialization is not required for publishing.

### Forward links

There is currently no easy way to see which other notes a note links to.

- This feature is useful for digital gardeners, who may want to include a link list at the end of their documents.
- On its own, this feature may not be worth implementing, but as part of "materialized links", it makes sense to support linking both ways.

## Detailed design

### Link format

Links are formatted as traditional Markdown links `[Title](target.md)`

- This prevents a materialized backlink being treated as a regular link, which would cause every linking relationship automatically becoming two-way
- **Link title**: Link title is the title of the target note at the time of the link's generation/update
- **Link target**: Link target is a relative path to the document where link originated
  - E.g. `[Title](target.md)`, `[Title](../target.md)`, or `[Title](subdirectory/target.md)`
- In the future, the link may also include anchor information to a particular section of the target note (see: https://github.com/foambubble/rfcs/pull/3) but that is out of scope for this proposal.
  
### Configuration

This feature will introduce the following VS Code configuration settings

- `foam.materialize.links.mode`: 
  - mode: `always` | `never` | `nonEmpty`   | `updateOnly`
  - default: `never`
- `foam.materialize.links.heading`: 
  - type: `String`
  - default: "Foam:Links"
- `foam.materialize.backlinks.mode`: 
  - mode: `always` | `never` | `nonEmpty`   | `updateOnly`
  - default: `never`
- `foam.materialize.backlinks.heading`:
  - type: `String`
  - default: "Foam:Backlinks"

#### `mode`

Link generation mode has the following values:

- `always`: Relevant link section (links, backlinks) will be generated for all pages if they don't already exist, whether or not there are links from/to that document. The sections will be inserted at the end of the document, in order: Links, Backlinks. User may change the order of these sections in the document, and the new order will be honored.
- `never`: (**default**): Link sections will not be generated. Existing link sections are not automatically removed.
- `nonEmpty`: Relevant link section (links, backlinks) will be generated for all pages if they don't already exist, but only when there are links from/to that document. The sections will be inserted at the end of the document, in order: Links, Backlinks. User may change the order of these sections in the document, and the new order will be honored. If a list becomes empty by a removal of a link, the heading is not automatically removed. 
- `updateOnly`: Relevant link sections are updated, but not created. This allows the user to opt into links and backlinks per-document basis, e.g. only generate backlinks for documents whose purpose is to act as [reference lists](https://foambubble.github.io/foam/reference-lists)

#### `heading`

Link headings are always generated as Markdown H2-elements, and the text is customizable using the `heading` setting.

Using headings as markers has the downside that they can collide with user-defined headings (e.g. if user had a section called "Links", they would be overwritten by this feature). The default values "Foam:Links" and "Foam:Backlinks" seek to minimize the odds of such a collision, but can be overridden.

### Output format

Link lists are output in the following format

```md
# Sample document

[[page-c]]  [[page-a]] [[page-b]] [[page-b]]

<!-- One empty line before heading -->

## Foam:Links
<!-- One empty line after heading -->

- [Page A](page-a.md) <!-- Title is read from linked document -->
- [Page B](page-b.md) <!-- Links are deduplicated -->
- [Page C](page-c.md) <!-- Links are in alphabetical order, title ascending -->

## Foam:Backlinks
<!-- One empty line after heading -->

- [Page A](page-a.md) <!-- Link can appear both in forward and backlinks -->
- [Page D](page-d.md) <!-- Links from documents -->

```

If document has no links or backlinks, the pertinent section is not generated.

### Generating/updating lists

Lists are updated as follows:

- In VSCode, when file is saved (similar to link reference definitions)
- In VSCode, When new link is added, the linked-to document is updated with the backlink
  - Logic for background updating exists in Foam Janitor
- In VSCode, when Janitor is executed
- On `foam-cli janitor`
  - Currently CLI Janitor has no way of reading workspace settings file. We'll need to add this ability

The extension will replace the entire immediate content of the section following the H2-heading, either until:

- Next scope (next H2 or H1 heading)
- End of file (or the link reference definition list)

### Documentation

We'll need to convert  the following roadmap items to recipes
- https://foambubble.github.io/foam/materialized-backlinks
- https://foambubble.github.io/foam/reference-lists

The documentation should make it clear that the generated sections are "managed" by Foam, and will be overridden.

## Drawbacks

Why should we *not* do this? Please consider:

- Mucking about user files can be fickle
- Non-trivial effort to implement and test the feature.
- (Current implementation): Using headings as link section delimiters can lead to destructivev updates, if user writes any content inside a Foam-managed section

## Alternatives

- Publishing use case: Generating links and backlinks in publishing pipeline only
- Storage/transport use case: Generating a secondary document that lists all links to/from each document
- Reference list use case: Improving the Backlinks explorer to make it more usable 

## Adoption strategy

Released "off by default" behind configuration flag. Purely opt-in feature that should not affect existing users.
