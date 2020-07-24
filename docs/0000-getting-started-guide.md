- Start Date: 2020-07-23
- RFC PR: (leave this empty)
- Foam Issue: (leave this empty)

# Summary

Add "Getting Started Guide" to Foam docs.

# Basic example

N/A

# Motivation

The motivation for doing this is to improve the UX around getting started with Foam. Here are a few snippets pulled from the Foam Discord in support of the need for this:

> Hi! I am trying to use foam for the first time. I already have visual studio code and I downloaded the foam-template as a zip. It then says open it in VS Code but I cannot find any workspace file to open it as a workspace. I am obviously doing something wrong can someone help me please

> Yeah I was confused by that too, the docs seem to imply opening it as a workspace not a folder

> Hi,is there a way to simplify the setup?
> I.e not create a github repo and just have a folder as the 'database'?

> I'm trying just to understand how it works for now. My idea is using it as a zettelkasten, but I don't know how to create the bubbles and the links. Also the visualization of the graph is not showing and I thought I had to run the code in Chrome or something, but just because VSCode has those 2 options: Run Chrome against localhost or preview with node.js

The goal is to expand beyond the [quickstart](https://foambubble.github.io/foam/#getting-started) we have and writing a guide that appeals to more users. The expected outcome is less questions around this topic in the Discord server.

# Detailed design

To implement this change, we change the "Getting Started" to "Quickstart" on the [homepage](https://foambubble.github.io/foam/#getting-started). Then, we would create a new page in the docs /getting-started, which would be this new "Getting Started Guide."

The "Getting Started Guide" would walk you through the entire process of using Foam from installing VS Code all the way to creating new wiki-links. By starting from the beginning, we would enable not only developers, but all types of users to get started with Foam. A great example of this is the [Gatsby tutorial](https://www.gatsbyjs.org/tutorial/), which has a step 0 for walking users through setting up their development environment.

### Example of some steps it will cover

This is non-exhaustive list of the steps it should cover:

- installing and setting up VS Code
- creating a GitHub account
- using the Foam template
- installing extensions
- writing your first bubble

### Other sections

It should also include:

- a frequently asked questions section
- glossary for the terms used (bubble, wiki-link, reference)

### A few key points to keep in mind

Last, here are a few key points to consider (based on conversations in the #questions and #help channels on the Discord server):

- don't call it a workspace file
- tell users to install all vscode extensions
- talk about using the graph
- talk about "assembling" foam (you don't install it)
- talk about creating references and wiki links
- talk about viewing wiki-links
- add a CTA at the end (next you should go try X, Y, Z) so they learn more about what's possible with Foam
- talk about not needing to upgrade from the template
- publishing (link to it)

# Drawbacks

Why should we _not_ do this? Please consider:

- implementation cost, both in term of code size, effort and complexity
- whether the proposed feature can be implemented in user space, a recipe, or a third party extension
- whether the proposed feature makes it more difficult for non-programmers to use Foam.
- integration of this feature with other existing and planned features
- cost of migrating existing Foam workspaces (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

# Alternatives

No other designs have been considered at the time of this writing.

The impact of not doing this is more confusion, more frustrated users and more questions on Discord.

# Adoption strategy

Exisiting Foam users won't need to change anything. There are no breaking changes. If it's implemented, the only change we'll need to make is how we onboard new users. We'll tell them to check out this getting started guide instead of the current approach.

# Unresolved questions

N/A
