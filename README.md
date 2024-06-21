# Seismometer RFCs

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes though are "substantial", and we ask that these be put
through a bit of a design process and produce a consensus among the Seismometer
core team.

The "RFC" (request for comments) process is intended to provide a
consistent and controlled path for new features to enter the project.

## When you need to follow this process
[When you need to follow this process]: #when-you-need-to-follow-this-process
You need to follow this process if you intend to make "substantial"
changes to Seismometer libraries, dependencies, algorithms, or the RFC process itself.
What constitutes a "substantial" change is evolving based on community norms,
but may include the following:

- Significant updates to seismometer's internal implementation
- New analysis templates
- New dependencies
- Removal of existing features

Some changes do not require an RFC:

- Rephrasing, re-organizing, refactoring, or otherwise "changing shape
  does not change meaning"
- Additions that strictly improve objective, numerical quality
  criteria (warning removal, speedup, etc.)
- Additions only likely to be _noticed by_ other developers-of-seismometer,
  invisible to users-of-seismometer

## What the process is

In short, to get a major feature added to Seismometer, one usually first gets
the RFC merged into the RFC repo as a markdown file. At that point the RFC
is 'active' and may be implemented with the goal of eventual inclusion
into Seismometer.

* Fork the RFC repo http://github.com/epic-open-source/seismometer-rfcs
* Copy `0000-template.md` to `text/0000-my-feature.md` (where
'my-feature' is descriptive. Don't assign an RFC number yet).
* Fill in the RFC. Put care into the details: **RFCs that do not
present convincing motivation, demonstrate understanding of the
impact of the design, or are disingenuous about the drawbacks or
alternatives tend to be poorly-received**.
* Submit a pull request. As a pull request the RFC will receive design
feedback from the larger community, and the author should be prepared
to revise it in response.
* Now that your RFC has an open pull request, use the issue number of the PR to rename the file: update your `0000-` prefix to that number. Also update the "RFC PR" link at the top of the file. 
* Build consensus and integrate feedback. RFCs that have broad support
are much more likely to make progress than those that don't receive any
comments.
* Eventually, the team will decide whether the RFC is a candidate
for inclusion in seismometer.
* An RFC can be modified based upon feedback from the team and community.
Significant modifications may trigger a new comment period.
* An RFC may be rejected by the team after public discussion has settled
and comments have been made summarizing the rationale for rejection. A member of
the team should then close the RFCs associated pull request.
* An RFC may be accepted at the close of its comment period. A team
member will merge the RFCs associated pull request, at which point the RFC will
become 'active'.

## The RFC lifecycle

Once an RFC becomes active, then authors may implement it and submit the
feature as a pull request to the Seismometer repo. Becoming 'active' is not a rubber
stamp, and in particular still does not mean the feature will ultimately
be merged; it does mean that the core team has agreed to it in principle
and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and is
'active' implies nothing about what priority is assigned to its
implementation, nor whether anybody is currently working on it.

Modifications to active RFCs can be done in follow-up PRs. We strive
to write each RFC in a manner that it will reflect the final design of
the feature; but the nature of the process means that we cannot expect
every merged RFC to actually reflect what the end result will be at
the time of the next major release; therefore we try to keep each RFC
document somewhat in sync with the language feature as planned,
tracking such changes via follow-up pull requests to the document.

## Implementing an RFC

The author of an RFC is not obligated to implement it. Of course, the
RFC author (like any other developer) is welcome to post an
implementation for review after the RFC has been accepted.

If you are interested in working on the implementation for an 'active'
RFC, but cannot determine if someone else is already working on it,
feel free to ask (e.g. by leaving a comment on the associated issue).
