---
sidebar_position: 2
---

# Important differences from the current model

This page mentions a number of major ways, in how this specification differs from the Opencast status quo.


## No snapshot system anymore

The old system of creating snapshots and using hardlinks on the file system is no more.
Whether and how want to version parts of an entity's data is still questionable (see [Open Questions](./open-questions)).


## No publications

There is no "engage", "external API", OAIMPH or any other internal _publication_ anymore.
There might still be a place for external publications in the sense of interacting with another system like YouTube.
These would require some async data synchronization and stuff.
But hardly anyone is using that, so while reading this specification just think: there are no publications at all.
The term does not exist anymore.

Instead, the DB, file system and all APIs have the same view of the world.
If an event with title "Banana" exists in Opencast, then it exists _everywhere_, i.e. in the DB, on the file system, and in all APIs¹.

This also includes modifications and deletions.
There is no staging area for changes anymore: all metadata and ACL changes to Opencast entities (event, series, ...) are instantly reflected in all APIs¹.
Changing metadata and ACLs does not require running a workflow anymore.
APIs for modifying this data promise that once they return 2xx, the change has been finalized to the database (the single source of truth).

A small number of Opencast users might like the two-stage metadata changing.
_If_ it is really desired, this "feature" can be implemented on top of the core Opencast, e.g. in the Admin UI (but disabled by default).

(¹) A small delay to update the search index is fine.

### Long running operations

Of course, there are some modifications or operations that cannot be done immediately, e.g. encoding a video or generating subtitles.
APIs starting these operations are _async_, i.e. they return 2xx to just state the operation has been started, but don't wait for the operation to finish.
But even with these operations, there is still only one view of the world.
For example, say a subtitle generation for an event was started: until the moment that operation finishes, the event has its previous subtitles (e.g. none) and that's reflected in all APIs.

An event is visible in APIs immediately after ingesting.
Of course, while the video is not encoded yet, there are no URLs to video tracks yet.
The API should represent that fact in a way that makes it easy for external apps to check if a video is still processing.

Sometimes, long running operations need to be run on metadata changes, e.g. to generate thumbnails with metadata in them (aside: this is usually not a great idea).
This can still be done, with the difference that the DB/API immediately reflects the changed metadata, while the thumbnail needs to catch up.
Again: the DB is the single source of truth.
Everything derived from it (e.g. search index, thumbnails, ...) needs to catch up.

As an aside, we should treat fewer operations as "long running" and thus offer synchronous APIs for them.
Cutting subtitles, generating thumbnails in different sizes, and more are things that can be easily done in tens of milliseconds.

## Storage format & API format

### Independence

How Opencast stores data should be independent of how Opencast exposes data in its API.
Just because the API format is JSON, does not mean that Opencast should store everything as JSON in the DB or on the file system.

Further, the structure of classes in Opencast code or the format in the search service should also not leak into the API.
The structure of the API response should be selected purely based on good API design and not on internals.
Avoiding to leak internals makes it easier to change these internals without breaking the API.
(The rewrite of the search service from Solr to ElasticSearch demonstrates how badly this can fail: the very widely used search API changed a lot.)

The implementation should do everything to ensure this separation.
For example, by having separate `record` definitions which are *only* used for API serialization.
This also makes it a lot harder to accidentally change the API.

### Unified response for all entities

An event in the API should always be represented with the same JSON response, regardless of whether it was fetched by ID, or returned from a full text search, or as the entry of a series.
Previously, this differed depending on whether it was loaded from the search index or the database or elsewhere.

Ideally, there shouldn't be a separate `search` endpoint anyway, but rather have the search feature be part of the external event API.
As an API user, I don't care what indices or data structures Opencast uses to give me the data.
And now that we use ElasticSearch/OpenSearch, there is no reason why there are nodes that couldn't perform that search.
