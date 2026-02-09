# Opencast Data Model

This repository contains the in-progress specification for the *future* Opencast data model.
This can be understood as a change request, but is written as a specification and will eventually end up as part of the main Opencast documentation.
This repository is here to collaboratively work on this specification until we reach community consensus.

Discussions take place in three places:
- In [Pull Requests](https://github.com/opencast/data-model/pulls) on this repository, which is especially useful for specific discussions about parts of the docs (using the line comment feature of GitHub).
- In [the GitHub issues labelled with `discuss`](https://github.com/opencast/data-model/issues?q=is%3Aissue%20state%3Aopen%20label%3Adiscuss) on this repository, for broader topics.
- Face to face at our architecture meeting or at conferences. To participate, it is expected that you have read all previous discussion and relevant text related to the topic you want to discuss. For meeting dates, follow [the Opencast discussion](https://github.com/orgs/opencast/discussions).

The rendered docs can be read here: https://opencast.github.io/data-model

### Build

This section is in case you want to build the rendered docs yourself.
This is not necessary to participate in the discussion.

To start a local development server with hot reloading:

```
npm ci
npm start
```

To build a static deployable website once:
```
npm ci
npm run build
```
