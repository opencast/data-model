---
sidebar_position: 1
title: Introduction
---

# Opencast Data Model

This document specifies the _future_ data model of Opencast.
The data model describes everything that is stored, what types and requirements certain data has, how it is represented in the API, how data can be changed, and more.

:::warning
This specification does *not* describe the current state of Opencast!
Also, it is a work in progress and is currently being developed and discuss in the community.
:::

Readers familiar with Opencast should ignore their prior knowledge while reading this, and treat this as a specification for a completely new software.
Do not interpret any existing OC behavior into this specification, if it isn't explicitly mentioned.
Also read the special [Important Differences](./important-differences) page, which explains where this data model differs in significant ways from the current Opencast.


## Goals

There are multiple reasons we are proposing this new data model:
- Improve robustness of Opencast by having a stricter and well defined data model. Be clear about what is allowed and what isn't, and catch invalid data as early as possible.
- Simplify developement of external applications: currently, the API responses are grossly underspecified and it is unclear what properties apps can expect from Opencast (e.g. do I need to deal with duration = -1?).
- Improve robustness by clearly specifying the source of truth for data and reducing the number of places/APIs that store/return data.
- Enable immediate modification of metadata (e.g. changing a video's title) without running a workflow.
- Improve performance by changing how data is stored.

The goal behind this very specification is to allow for easy discussion in the community, and eventually to have a written specification.

This specification is written mainly as if it was talking to API users, i.e. developers of external apps who want to integrate with Opencast.
I think this is a useful choice to define the "public interface" of Opencast.
The document does contain quite a bit of implementation notes, too, which just define how things should be handled inside Opencast.

## Contributing to this specification

Discussing every single detail in the community beforehand is not viable and not necessary.
Instead, the idea is that there is one main person working on this spec, writing most of the text, therefore proposing parts of the model.
These proposals are discussed in regular meetings and on GitHub.
See [the `opencast/data-model`](https://github.com/opencast/data-model) repository, and in particular the pull requests and discussions tabs.

## Backwards compatibility and breaking changes

It is very clear that we need to be able to migrate existing data to the new model.
We also don't want to change every single piece without good reason, in order to keep the overall change managable.
The new model was designed with that in mind.
That said, this document (especially its initial version) does contain incompatibilities and breaking changes, and does not yet consider every single use case.
I expect these use cases to be discussed during the community review of this.
