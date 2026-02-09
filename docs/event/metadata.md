---
sidebar_position: 2
---

# Metadata


## Fields

(🟦) Metadata fields marked with this symbol are *Opencast-managed*: they are read-only for users/external applications. All other fields can be freely changed, as long as validity checks pass.

When *duplicating* an event, all fields are copied 1:1 unless specified otherwise.

### General
- `id: ID` 🟦: unique among all events.
  - Can be chosen when creating an event, but cannot be changed afterwards.
  - If no ID is specified while creating an event, Opencast generates a generated unguessable ID.
  - When duplicating an event, the new event gets a new unguessable ID.
- `title: NonBlankString`: a short title that is the main label associated with this event for users. Plain text.
- `description: string?`: user-specified, human-readable description, potentially quite long.
  - The text is *not* markdown, but plain text. However, there are some rules about how the text should be displayed:
    - Parts of the text separated by `\n\n` must each be treated as a paragraph (rendered with `<p>`). The space between paragraphs must be between 1.5 and 2 times the line height.
    - Within a paragraph, `\n` must be rendered as a visible line break.
    - *Optional recommendation*: valid HTTP(s) URLs in the text may be rendered as proper link element (`<a>`). Exactly specifying an allowed URL grammar is overkill for this specification: applications should use well-known library to perform link detection, with a conservative bias to avoid false positives.
    - Any other text must be displayed verbatim. For example, HTML tags inside the text need to be escaped.
- `creators: NonBlankString[]`: The people mainly responsible for creating this video and/or presenting the talk which this video is a recording of. Should contain human-readable names and not usernames. Plain text. This is the main "who?"-information shown in the UIs; other fields in `extraMetadata` (e.g. `dct:contributor`) might be shown too, but less prominently.
- `language: LangCode?`: describes the (main) language of this event and its metadata. For example, the audio language and (if applicable) language of video content is more important than the language of available subtitles. Generally, assets can have their own language specified.
- `series: SeriesID?`: optional ID of the series this event belongs to. Must be a valid series ID of an existing series at all time.
- `owner: Username`: TODO figure out details
- `createdBy: Username` 🟦: username of the user that created this event.
  - Like `created`, this refers to the moment when the event is first added to the database, not necessarily when it is ingested.
  - This refers to the user with which the API request is authorized, e.g. potentially an API user and not referring to an actual human person.
  - Technical field, not intended to be shown to normal users.
  - Set by Opencast and cannot be changed.
  - When duplicating an event, the new event has this field set to the duplicating user.

### Time-data
- `startTime: DateTime?`: Actual real life datetime when the video recording started or will start, with timezone. If this is not applicable, for example because it's a short movie, this should be undefined. UIs should use this as primary date to show for a video and if unset, fallback to `created`.
- `endTime: DateTime?`: Like `startTime`, but when the video recording stopped. Due to cutting, recording pauses and etc, the `duration` is not necessarily `end - start`.
- `duration: Milliseconds` 🟦: duration of the event. As specified in ["assets"](./assets), this needs to always match the duration of all non-internal tracks.
- `modified: Timestamp` 🟦: Timestamp of when anything about this event was last changed.
  - More precisely: at any point in time since `modified`, all fields, assets, ACL and any other part of the event data model need to have the exact same value as they have at the present moment.
  Whenever anything about an event described in this data model changes, `modified` has to be set to `now()`.
    - Noteworthy case: when a series is deleted and the event's `series` field is set to `null`, the event's `modified` needs to change.
    - Opencast should try its best to not update `modified` when it's not necessary (e.g. when the title is set to the current value), but it is not a bug if `modified` is set to `now()` unnecessarily.
  - When duplicating an event, the new event has `modified = now()`, i.e. it is not copied.
- `created: Timestamp` 🟦: Timestamp of when the event was created in Opencast. It is set once when the event is first stored in Opencast's DB, and never changed again.
  - This also implies that scheduled event's `created` date is when the scheduling took place, _not_ the time it is scheduled for (that would be `startDate`).
  - When duplicating an event, the new event has `created = now()`, i.e. it is not copied.


### Flags

- `isLive: bool` 🟦: TODO this is currently stored per track, figure out if that's useful


### Extra metadata
- `extraMetadata: Map<Label, Json>`: additional metadata that Opencast never interprets, but just stores and passes along.
  - The keys of this map consists of a _namespace_ and a _field name_, separated by `:`, i.e. `ns:name`. Both parts must consist of only `a-z`, `A-Z`, `0-9`, `-` and `_`.
  - The values of this map are arbitrary JSON values, i.e. Booleans, numbers, strings, arrays, maps or `null`. Items of arrays & maps can recursively be any JSON value as well.
  - Unlike the "extended metadata" before, using `extraMetadata` does work out of the box and does not incur any relevant performance overhead. Therefore, applications are encouraged to add useful data here, e.g. `studip:course-id`, `oc-studio:version` or `ethz:room-number`.

#### Community documentation

There should be a community document for specifying rules, as well as collecting used fields and best practices around `extraMetadata`.
That way, common requirements are identified quickly and the community can converge towards a standard.

> Official namespaces:
> - **`oc`**: reserved.
> - **`ocx`**: OC eXperimental for fields that might get promoted to core metadata field in the future.
>   - `ocx:downloadable: bool`: a flag indicating whether users are allowed to download this video (i.e. tracks attached to this event). This can inform external apps whether to show a download button or to enable anti-download protection. The exact effects of this flag are deliberately unspecified, this merely states an *intent*. *Note* (TODO): this is currently discussed [in this issue](https://github.com/opencast/data-model/issues/11).
>   - `ocx:listed: bool`: specifies whether this event should be considered "listed", meaning that users can find it via search. If it is `false`, users have to know the ID of the event (e.g. via a series or playlist) in order to access it.
>   - `ocx:explicitContent: bool`: specifies whether this event contains content that is considered "explicit", like swear words or whatnot. This is required for some integrations like iTunes.
> - **`dct`**: refers to the Dublin Core Terms specification, e.g. `dct:rightsHolder` refers to [the `rightsHolder` property](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/rightsHolder) of DC terms. It should be avoided to set fields that are already mapped from core fields, like `dct:title`, which is mapped to the OC core metadatum `title`.
>
> Community namespaces:
> - ...

Generally, anyone should be able to add to the "Community namespaces" section via pull request against this document:
its purpose is to document what is used, not for the OC committers to control what external apps do.
In other words: a PR against that section is *not* asking for permission.





## Dublin Core mapping

As you can see above, the event metadata is not literally a Dublin Core catalog anymore.
Dublin Core should be considered an *exchange format*, not a storage format!
Of course, standards make sense and therefore, many Opencast metadata fields correspond exactly to a Dublin Core field.
Opencast can offer an API that emits a DCC for an event.
That DCC XML is created on the fly in the API handler.

The mapping is as follows:

- [`identifier`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/identifier): `id`
- [`title`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/title): `title`
- [`description`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/description): `description`
- [`creator`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/creator): `creators`
- [`language`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/language): `language`
- [`isPartOf`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/isPartOf): `series` (i.e. the ID)
- [`modified`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/modified): `modified`
- [`extend`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/extent): `duration` as ISO 8601 duration
- `date` or `temporal`???: TODO combination of `startDate` and `endDate`
- `created` or `dateSubmitted` or `issued`???: TODO `created`
- Additionally, all fields in `extraMetadata` with `dct` namespace are mapped directly to the corresponding property, e.g. `dct:license` is mapped to [`license`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/license).

TODO: Should we also define a mapping for OAIMPH? Does that make sense?

---

:::danger[Open questions]

- ...
:::
