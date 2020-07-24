# Foam RFCs

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put
through a bit of a design process and produce a consensus among the Foam
maintainers and the sub-teams.

The "RFC" (request for comments) process is intended to provide a
consistent and controlled path for new features to enter the project.

[Active RFC List](https://github.com/foambubble/rfcs/pulls)

Foam is still **actively developing** this process, and it will still change as
more features are implemented and the community settles on specific approaches
to feature development.

## When to follow this process

You should consider using this process if you intend to make "substantial"
changes to Foam or its documentation. Some examples that would benefit
from an RFC are:

   - A new feature that creates new API surface area or UI functionality
   - The removal of features that already shipped.
   - The introduction of new idiomatic usage or conventions, even if they
     do not include code changes to Foam itself.

The RFC process is a great opportunity to get more eyeballs on your proposal
before it becomes a part of a released version of Foam. Quite often, even
proposals that seem "obvious" can be significantly improved once a wider
group of interested people have a chance to weigh in.

The RFC process can also be helpful to encourage discussions about a proposed
feature as it is being designed, and incorporate important constraints into
the design while it's easier to change, before the design has been fully
implemented.

Some changes do not require an RFC:

  - Rephrasing, reorganizing or refactoring
  - New documentation sections
  - Bugfixes, minor improvements to existing functionality
  - Additions only likely to be _noticed by_ other maintainers-of-Foam,
  invisible to users-of-Foam.

## What the process is

In short, to get a major feature added to Foam, one usually first gets
the RFC merged into the RFC repo as a markdown file. At that point the RFC
is 'active' and may be implemented with the goal of eventual inclusion
into Foam.

* Fork the RFC repo http://github.com/foambubble/rfcs
* Copy `0000-template.md` to `docs/0000-my-feature.md` (where
'my-feature' is descriptive. Don't assign an RFC number yet).
* Change the title of the file to match the file name (e.g. `# 0000 Template` to `# 0000 My Feature`).
* Add a Foam-style wiki link to `docs/index.md` and ensure link reference definitions are appropriately generated.
* Fill in the RFC. Put care into the details: **RFCs that do not
present convincing motivation, demonstrate understanding of the
impact of the design, or are disingenuous about the drawbacks or
alternatives tend to be poorly-received**.
* Submit a pull request. As a pull request the RFC will receive design
feedback from the larger community, and the author should be prepared
to revise it in response.
* Build consensus and integrate feedback. RFCs that have broad support
are much more likely to make progress than those that don't receive any
comments.
* Eventually, the team will decide whether the RFC is a candidate
for inclusion in Foam.
* An RFC can be modified based upon feedback from the team and community.
Significant modifications may trigger a new final comment period.
* An RFC may be rejected by the team after public discussion has settled
and comments have been made summarizing the rationale for rejection. A member of
the team should then close the RFCs associated pull request.
* An RFC may be accepted at the close of its final comment period. A team
member will merge the RFCs associated pull request, at which point the RFC will
become 'active'.

## The RFC life-cycle

Once an RFC becomes active, then authors may implement it and submit the
feature as a pull request to the Foam repo. Becoming 'active' is not a rubber
stamp, and in particular still does not mean the feature will ultimately
be merged; it does mean that the core team has agreed to it in principle
and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and is
'active' implies nothing about what priority is assigned to its
implementation, nor whether anybody is currently working on it. The team or 
sub-team taking responsibility of the feature will choose how to prioritize it. 

Modifications to active RFCs can be done in followup PRs. We strive
to write each RFC in a manner that it will reflect the final design of
the feature; but the nature of the process means that we cannot expect
every merged RFC to actually reflect what the end result will be at
the time of the next major release; therefore we try to keep each RFC
document somewhat in sync with the language feature as planned,
tracking such changes via followup pull requests to the document.

## Implementing an RFC

The author of an RFC is not obligated to implement it. Of course, the
RFC author (like any other developer) is welcome to post an
implementation for review after the RFC has been accepted.

The likelihood of a feature getting shipped is significantly improved if you are
able to participate in implementing yourself, or if you are able to rally other
contributors to your cause and motivate them to work on the feature. 
A feature may be worked on by the same person(s) who created the RFC, but
this is not a requirement. It is perfectly valid to open a RFC without intending
(or being able) to implement it yourself.

If you are interested in working on the implementation for an 'active'
RFC, but cannot determine if someone else is already working on it,
feel free to ask (e.g. by leaving a comment on the associated issue).

**This RFC process is subject to change. If it's not working for us, we'll improve it, or replace it with a more suitable process.**

**Foams's RFC process owes its inspiration to the [React RFC process], which in turn takes inspiration from [Yarn RFC process], [Rust RFC process], and [Ember RFC process]**

[React RFC process]: https://github.com/reactjs/rfcs
[Yarn RFC process]: https://github.com/yarnpkg/rfcs
[Rust RFC process]: https://github.com/rust-lang/rfcs
[Ember RFC process]: https://github.com/emberjs/rfcs
