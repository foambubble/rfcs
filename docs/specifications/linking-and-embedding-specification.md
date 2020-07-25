# Linking and Embedding Specification

- [Terminology](#terminology)
- [Targets](#targets)
  - [Notes](#notes)
  - [Headings](#headings)
  - [Anchors](#anchors)
  - [EBlocks](#eblocks)
  - [Tags](#tags)
  - [Other Local Files](#other-local-files)
  - [External Locations](#external-locations)
- [Linking](#linking)
  - [Linking Local Markdown Files](#linking-local-markdown-files)
    - [Markdown Links](#markdown-links)
    - [Wikilinks](#wikilinks)
      - [Word-based Wikilinks](#word-based-wikilinks)
      - [Path-based Wikilinks](#path-based-wikilinks)
      - [Link Labels](#link-labels)
      - [Linking to aliased notes](#linking-to-aliased-notes)
      - [Using Tags](#using-tags)
  - [Linking Local Non-Markdown Files](#linking-local-non-markdown-files)
  - [Linking External Locations](#linking-external-locations)
- [Embedding](#embedding)
- [Indexing](#indexing)
- [Link Configuration](#link-configuration)
- [User Experience](#user-experience)
  - [Auto-completion](#auto-completion)
  - [Navigation](#navigation)
  - [Creating New Notes](#creating-new-notes)
  - [Go To Defintion / References / Peeking](#go-to-defintion--references--peeking)
  - [Hover Preview](#hover-preview)
  - [Diagnostics](#diagnostics)
    - [Links](#links)
    - [Notes](#notes-1)
    - [Aliases](#aliases)
  - [Link Refactoring](#link-refactoring)
  - [Backlinks](#backlinks)
  - [Freelinks](#freelinks)
  - [Graphs](#graphs)
- [Implementation Details](#implementation-details)
- [Todo](#todo)

The following specification describes how linking and embedding works in Foam.

## Terminology

The list below explains the terms used in this document. 

- **Anchor**: Location in a note that can be pointed at.
  *See Also*: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a
- **AST**: Abstract Syntax Tree; Tree structure where the Markdown content is parsed into.
- **AST Node**: Node in the Abstract Syntax Tree.
- **Block Embedding**: Dynamically inserting content from a note A into a note B. Note A and B can also be the same note, but different location within that note. Embedding is done with a piece of textual syntax.
- **Build-time**: Activities that happen while a markdown file is transformed to another format (e.g. html or another kind of markdown), are said to be build-time activities.
- **Edit-time**: Activities that happen while a markdown file is edited, are said to be edit-time activities.
- **EBlock**: Embeddable block; Piece of text in a note that can be embedded. See block embedding.
- **Frontmatter**: Metadata section at the beginning of a note.
  *See Also*: https://jekyllrb.com/docs/front-matter/
- **Graph**: Graph structure that contains information about all the files in the workspace. Notes in the workspace are analyzed further e.g. for links and link targets.
  *Also known as*: Index.
- **Id**: Element attribute in the generated HTML that can be targeted with links / block embedding. Ids can also be used within the markdown files when defining anchors.
- **Indexing**: Operation that goes through all the markdown files in the Foam workspace, finding all the links, block embeds, and link targets and finally linking these together in a Graph.
- **Link Reference Definitions**: Information in a markdown file that converts wikilinks into regular markdown links.
  *Also known as*: LinkRefDefs.
  *See Also*: https://github.github.com/gfm/#link-reference-definitions
- **Link**: A piece of text that points to a Target.
- **Note**: A file containing text in markdown format.
  *Also known as*: Foam Bubble, Document, Markdown file.
- **Tag**: A piece of text that either acts as a link, or as an abstract categorization.
- **Target**: An item (e.g. note, heading or an anchor) that can be pointed to with a link.
- **Title**: The H1 heading of a note. A note can contain only a single title.
- **Workspace**: The location (folder) where Foam Notes live in.

## Targets

The following items can be targetted via linking and / or embedding. The separation is explained in more detail in [Linking](#linking) and [Embedding](#embedding) chapters.

### Notes

Notes in Foam are individual markdown files. Notes can both reside in the Foam workspace root and any of its sub-folders. Both files and folders are encouraged to follow the kebab-case naming convention (e.g. `some/path-to/a-file.md`).

### Headings

Headings in Foam are denoted with markdown heading syntax, e.g. `## Heading on Second Level`. Each heading will have a corresponding id, a kebab-case version of the text (e.g. `heading-on-second-level`). In case there's multiple headings with the same text, their id is suffixed with a running number (e.g. `my-heading`, `my-heading-1`, `my-heading-2`).

### Anchors

User can add arbitrary anchors into a note with HTML-syntax: `<a name="my-anchor"></a>`. Since these names (identifiers) are written by users, they might end up being non-unique within a note. In these cases, an error is shown to the user. See the reasoning of using the name-attribute instead of the id-attribute from https://stackoverflow.com/a/7335259. In the future, better syntax for the anchors might be developed.

### EBlocks

While anchors define a single location in the document, eblocks define two; the beginning and the end of an eblock. The intent of a block is to specify content bounded by the begin and end markers. The end of an eblock can either be explicit or implicit.

**Explicit eblocks** are defined with the anchor syntax and have two special rules; 1) the name of the beginning anchor starts with `begin:` and the ending anchor starts with `end:`, and 2) the other parts of the name must match. An example of matching anchors is:
 
  ```markdown
  <a name="begin:my-block"></a>
  Sunt cillum occaecat adipisicing aliquip excepteur
  nisi ea nostrud do nostrud.

  Ad magna labore tempor dolor dolore nulla magna Lorem
  magna qui et. Ipsum eiusmod minim ad est nostrud sunt occaecat.
  <a name="end:my-block"></a>
  ```

**Implicit eblocks** are defined by any anchor whose name is not starting with `begin:` or `end:`, and the point where the nesting level of the first non-immediate AST node either remains on the same level or decreases in comparison to the starting place of the anchor. The nesting level is changed e.g. with different levels of headings and bullet list indents. Additionally, the eblock can also be started with a heading, in which case the eblock ends when the chapter following the heading ends. In case the end of a file is found before the nesting level of the markdown content decreases, eblock ends there. The example below illustrate implicit eblocks with various eblock definitions.

```markdown
# My Note

Occaecat esse minim Lorem fugiat mollit eu eiusmod magna dolor.
Velit ea cupidatat pariatur amet pariatur est est laboris culpa
ipsum magna eu nulla esse.

## Second Level

<a name="y"></a>
Cupidatat velit esse est anim consequat enim ad.

- Nostrud incididunt
- <a name="x"></a> consequat non dolore
  - consequat fugiat
  - eiusmod esse ipsum
    - anim nulla aute
- ad reprehenderit

## Another Second Level

Ut reprehenderit sint quis pariatur. Non ullamco eiusmod nulla
do sunt pariatur magna fugiat. Ullamco nulla magna nostrud
in aute eu laborum ex est nostrud nostrud excepteur Lorem.
```

Examples of implicit eblocks defined above:

- `my-note` refers to the whole note (id of the H1 heading of the note).
- `second-level` refers to the "Second Level" chapter (stopping before the heading "Another Second Level")
- `y` refers to the text `Cupidatat velit esse est anim consequat enim ad.`
- `x` refers to the following list fragment:
  ```markdown
  - consequat non dolore
    - consequat fugiat
    - eiusmod esse ipsum
      - anim nulla aute
- `another-second-level` refers to the "Another Second Level" chapter (stopping at the end of the file)

### Aliases

Aliases define alternative names for notes. They are defined in an optional front matter, using an attribute "aliases". The example below shows how this can be done using YAML front matter:

```YAML
---
aliases:
  - My First Alias
  - My Second Alias
---
```

### Tags

Tag is a sequence of non-whitespace characters prefixed with the hash character (e.g. `#my-hash`). Tags can work in one of two ways: 1) it's an abstract symbol that can be searched and inspected across notes (i.e. *abstract tag*) or 2) it's like 1, but simultaneously it's also a note in its own right (i.e. *file-based tag*). In the case of 1, it acts like hashtags in popular social media platforms like Instagram or Twitter. In the case of 2, it's more like a note where the user can write just like into any other note. These notes are stored in the `tags/`-folder within the Foam workspace.

### Other Local Files

Not all content within the workspace has to be markdown files. Files may appear in various other formats, like `json`, `txt`, `jpeg` or even `xlsx`. These can also be linked to from the markdown files.

### External Locations

It's typical to refer to external (i.e. non-local) locations from markdown files. These include e.g. web pages and other network locations. Linking to these are done via normal markdown mechanics.

## Linking

This chapter explains how to create links that point to various targets.

### Linking Local Markdown Files

#### Markdown Links

Linking local markdown files can be done using the regular markdown linking syntax. The link target is relative to the note where the linking defined. Examples:

- `[My Note](my-note.md)` - note is in the same folder.
- `[My Note](../../my-note.md)` - note is in the grand-parent folder

#### Wikilinks

Wikilinks allow a convenient way of linking across notes in the workspace. The same effect can be achieved e.g. with normal markdown links with relative paths (e.g. `[My Note](../../my-note.md)`), but it would be more verbose than the use of wikilinks (e.g. `[[My Note]]` or `[[my-note]]`). Wikilinks can have one of two formats, word-based or path-based.

##### Word-based Wikilinks

Word-based wikilinks blend into the surrounding text, since they are words like any other text in the paragraph; this is also their biggest benefit. The drawback is that in case the notes have meaning associated with their directory structure, this meaning cannot be seen from the link itself (e.g. separation between `active/master-plan`and `obsolete/master-plan` would both look like `[[Master Plan]]`). Furthermore, the edit-time creation of the note from the link is not unambiguous, since the path is missing.

The rules governing the word-based linking are as follows:

- Pointing to a note is done via its H1 heading, e.g. `[[My Note]]`.
- The word-based wikilink in converted to a normal markdown link using the link reference definitions. Example:
  - `[Some Title]: some-file.md`
- In case the workspace contains two or more notes with the same heading, disambiguation is done with the link reference definitions [1]. Example:
  - `[Some Title]: some-file.md`
  - `[Some Title:2]: subdirectory/some-file.md`
- In case the note heading is missing, an arbitrary link can be used, which is then retargeted with the link reference definitions: `[Some Arbitrary Label]: some-file-without-heading.md` [1]
- Pointing to a heading within a note is done using the heading as an anchor: `[Some File#Some Heading]` [2]
- If a note contains multiple headings with the same text, link reference definitions must disambiguate between them using the running number of the identifier, e.g.: `[Some File#Some Heading]: some-file.md#some-heading-1`.
- Pointing to an arbitrary anchor within a note is done via its id (e.g. `[[Some File#foo]]`), since anchors might not a have a title (e.g. `<a id="foo"></a>`) [2].

[1]: This implies that the link reference definitions are not ephemeral, and they cannot be reconstructed solely based on the index.
[2]: This, in its full form (e.g. including the anchor), is recorded into the link reference definitions.

##### Path-based Wikilinks

Path-based wikilinks point to the file names, and if the note is in subfolders in relation to the workspace root, these folders are also included directly into the link itself. The benefit of the path-based links is that they are more unambiguous than the word-based links, and they also expose the potential semantics of the directory structure. They also generate smaller amount of link reference definitions, thus reducing the need for programmatic insertion of text into the files. The drawback is that they look more like "code", and might seem foreign especially to non-coder types.

The rules governing the path-based linking are as follows:

- Pointing to a note is done via its workspace rooted path, e.g. `[[some/path/here.md]]`
- Links can also be written without the file extensions, like `[[some/path/here]]`
- In case there's conflicts due to lack of file extensions (e.g. `here.md` and `here.doc` in the same path), link reference definitions are used to disambiguate (e.g. `[some-file]: some-file.md`) OR the file extension is added into the link itself OR an error is shown to the user.
- Pointing to a heading within a note is done based on id, e.g. `[[foo/bar#my-title]]`
- Pointing to an arbitrary anchor within a note is done also based on id, e.g. `[[foo/bar#my-anchor]]`

##### Link Labels

All wikilinks can have an optional label assigned to them. The assignment is done by separating the link target and the label with a pipe-character: `[[Some Note#Some Anchor|Some Label]]`

##### Linking to aliased notes

As explained in the [aliasing chapter](#aliases), a note can define aliases to itself. These aliases are used only to look up the correct title / path for the link, and they are not recorded into the links themselves. 

##### Using Tags

Tags can be referred (i.e. linked to) using the normal hashtag-syntax anywhere in the text (e.g. `It's raining #cats and #dogs.`). Additionally, tags can be referred to using the front matter in the following way:

  ```yaml
  ---
  tags: cats, dogs
  ---
  ```

Tags in the front matter may contain the `#`-character (i.e. `tags: #cats, #dogs`), they are interpreted in the same way as the tags not containing the character (i.e. `tags: cats, #dogs` is equal and valid).

If a tag is file-based and it contains `/`-characters, this denotes a subfolder in the `tags/`-folder. Tags ending with `/` character act the same as a tag without that character. By default, tags are abstract and can be converted into files by the user.

### Linking Local Non-Markdown Files

Any file in the workspace (e.g. excel-files) can be linked using one of the following 1) normal markdown link syntax, 2) path-based wikilink syntax, or 3) word-based wikilinks with arbitrary text using the link reference definitions.

### Linking External Locations

Linking external locations (e.g. web pages) is always done using one of the normal markdown link syntaxes.

## Embedding

Embedding differs from linking mainly when inspecting the workspace after it has been built e.g. into html format. While linking just adds a link into the page, embedding embeds the whole target into the location where it is referred to.

Embedding is done with the same syntax as linking with one exception, adding an exclamation mark (`!`) in front of the link. Therefore `[[Some Page]]` adds just a link, while `![[Some Page]]` replaces that reference with the actual page. 

While embedding various targets, the following rules apply:

- When embedding a target that is the same as where the embedding is done from, it raises an error (recursion).
- Similarly when embedding a target that itself embeds another target (which might yet embed another target and so on), if the embeds create a cycle, an error is raised.
- When embedding a normal link target (e.g. heading), it automatically converts into an implicit embed.
- Aliases cannot be embedded just like they cannot be linked to.
- Tags cannot be embedded directly. File-based tags can be embedded by referring to the corresponding file (or its title) in the `tags/`-folder.
- Non-markdown local files cannot be embedded since they might have any format, with the exception of the supported image-formats (e.g. jpeg and svg).
- External locations cannot be embedded. If embedding is desired for these targets, use e.g. MDX or iframes.

## Indexing

Indexing the workspace is done in two phases:

1. When the workspace is initially loaded, all the *relevant* files in the workspace are walked through. All the files (with their paths) create nodes in the graph. Notes are analyzed further for link targets (e.g. headings and anchors), and all the links are examined. Redirections are created based on aliases. Tags are found and linked to files where applicable. Any inconsistencies are reported through diagnostics (linting). The graph is stored in memory.
2. Every file change (even changes just in the editor buffer and not yet saved to disk) cause an update to the graph. Only the relevant parts are updated into the memory, which also triggers another round of consistency checking.
   
*Future consideration: serialize the graph also to disk using file checksums? This would enable faster loading for the workspace AND if the format is well defined, e.g. visualizations could be developed against it.*

## Link Configuration

The following describes the relevant configuration options for linking and embedding:

- **includePaths / excludePaths**: Ability to first state which paths within the workspace to scan through (*default = all files within the workspace*), and then filter these down using path exclusions (these can include both sub-folders like `node_modules`, as well as file name matching like dropping all the files starting with `~`-character).
- **prefferedWikilinkFormat**: While the workspace supports both the word-based and path-based wikilinks simultaneously, the editor must know which ones to create when the auto-complete option is selected.
- **buildTargets**: List of various build targets that describes how links and tags are converted so that they would be processed correctly by various static site generators, or shown correctly on various sites. Exact options TBD.

## User Experience

The following chapters explain how user can interact with various links and embeds.

### Auto-completion

When user types `[[`, a dropdown is shown, which displays all possible link targets. The format depends on `prefferedWikilinkFormat` setting:
  
- **path**: format is `path/to/file - Title`
  - If the file doesn't have a title, only the first part is shown (`path/to/file`)
- **title**: format is `Title - path/to/file.md`
  - If the file doesn't have a title, only the last part is shown (`path/to/file.md`)
- In both cases, aliases are shown in the following format: `Alias - path/to/file.md`
- When referring to anchors, the first part will always be the one where the anchor is appended to (e.g. `path/to/file#anchor - Title` vs. `Title#anchor - path/to/file.md`)
- In case there's no match in autocompletion, it denotes a need to create a new note. If this text includes anchors, these will never be automatically created (cannot deduce if it's a heading or some other element). See how [new notes are created](#creating-new-notes).

When user types `#`, a dropdown is shown that displays all possible tags. If a tag is file-based, possible aliases in that note also become tags themselves and are shown in the same dropdown. If one of the aliases is selected, the actual tag is inserted instead. In case there's no match in autocompletion, it denotes a need to create a tag. These are abstract tags by default; see [creating new notes](#creating-new-notes) on how to convert tag to be file-based.

### Navigation

TBD

### Creating New Notes
  
- TBD
  - If the `prefferedWikilinkFormat` ...
  - Convert tags to be filebased
  - CTRL+LMB
  - Automatic creation on autocomplete?

### Go To Defintion / References / Peeking

TBD

### Hover Preview

TBD

### Diagnostics

#### Links

TBD

#### Notes

- TBD
  - multiple anchors with the same id

#### Aliases

TBD

### Link Refactoring

TBD

### Backlinks

TBD

### Freelinks

TBD

### Graphs

TBD

## Implementation Details

- TBD
  - Link refs cannot be created from scratch

## Todo

- Still to be walked through
  - Related PRs and GH discussions
  - Implementation (outlined in https://foambubble.github.io/foam/link-reference-definition-improvements)
  - See gistpad (e.g. create gif)
  - Other notes in foam docs
  - Relevant discord chats
