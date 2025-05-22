---
sidebar_position: 3
---

# Common specifications

## Data storage

The single source of truth for everything is the database (DB) plus files on the file system¹ referenced by the DB.
Every piece of information is only stored in one place in the DB.

Only a handful of files are stored on the file system:
- Binary and/or large files like video, audio, images, ...
- Files that need to be delivered in a specific format anyway (VTT subtitles, ...)

:::info[Differences from current OC model]
In particular, textual metadata, ACLs, cutting information or anything like that is _not_ stored on the file system!
(Some APIs might still accept or produce these information in non-JSON exchange formats.)
:::

The database never references files by absolut path or URL.
At most, it stores a path relative to the configured `storage.dir`, but potentially in an even more implicit way.


(¹) File system = local file system, or NFS, or S3 storage or potentially others.

### Derived data storage

For different purposes, it might be useful to store the same data again in a different form.
For example, using a search index for full text search.
(Note however: whenever possible and useful, use DB indices built into the DBMS.)

These derived data sources can be slightly behind the DB (e.g. due to indexing times), which is acceptable.
However, it is crucially important that data only ever flows from the DB into other data stores, _never_ the other way around.
Deleting all derived data stores must never result in data loss as they can always be regenerated from the DB.
Rebuilding derived data stores must always results in the same result, regardless of what the derived store previously contained.
Opencast should do its best to keep the derived data stores in sync in a timely manner.


## Promised properties

This data model promises certain properties about certain fields/data, for example: "there is a non-empty title", "this is an array of strings" or "the duration matches the duration all tracks".

- It's Opencast's responsibility to ensure these properties. Whenever an entity is added or changed, these properties need to be maintained, usually by rejecting the change request (e.g. 4xx response in API).
- If an entity does not have these properties, this should be considered a bug in Opencast and should be fixed ASAP.
  - We should never find us in the situation where external apps (e.g. LMS plugins) need to work around a broken property of Opencast.
- The same goes for legacy events, which might be broken in the new model. They cannot be kept as is, they need to be changed/migrated to exhibit these properties.
- The implementation should try, wherever possible, to make broken events impossible to represent. As a simple example, the title field in the DB should be `non null`.


## Multi-tenancy

The rest of these specs pretend like there is only one tenant.
Multi-tenancy will be implemented on a "higher level of abstraction" than this data model.
As a simple example: each tenant has its own database and folder on the file system.
This not only makes proper isolation easier and improves the code by not mixing this complexity into other logic, it also makes writing this specification easier.


## Well defined API response

While not technically part of the data model, the possible responses of APIs should be well defined and documented.
The documentation should automatically be derived from the code in order to keep it up to date (which otherwise will absolutely fail).
The implementation details need to be figured out, but the idea is that the same "code" (e.g. a Java `record` definition with attributes) that leads to the serialized API response is also used as source for the documentation.

Users of the API should never need to look at an actual response to know what to expect.
An actual response is always something *specific* and does not communicate what fields are optional and what possible values to expect for each field.
