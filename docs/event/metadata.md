---
sidebar_position: 2
---

# Metadata

(🟦) Metadata fields marked with this symbol are *Opencast-managed*: they are read-only for users/external applications. All other fields can be freely changed, as long as validity checks pass.

#### General
- `id: ID` 🟦: unique among all events.
- `title: NonBlankString`: a short title that is the main label associated with this event for users. Plain text.
- `description: string?`: user-specified, human-readable description, potentially quite long.
  - TODO: Decide whether this is plain text, markdown or anything else. External apps displaying this need to know that. Some basic formatting options might be nice?
- `creators: NonBlankString[]`: The people mainly responsible for creating this video and/or presenting the talk which this video is a recording of. Should contain human-readable names and not usernames. Plain text. This is the main "who?"-information shown in the UIs; other fields in `extraMetadata` (e.g. `dct:contributor`) might be shown too, but less prominently.
- `language: LangCode?`: describes the (main) language of this event and its metadata. For example, the audio language and (if applicable) language of video content is more important than the language of available subtitles. Generally, assets can have their own language specified.
- `series: SeriesID?`: optional ID of the series this event belongs to. Must be a valid series ID of an existing series at all time.
- `owner: Username`: TODO figure out details

#### Time-data
- `startTime: DateTime?`: Actual real life datetime when the video recording started or will start, with timezone. If this is not applicable, for example because it's a short movie, this should be undefined. UIs should use this as primary date to show for a video and if unset, fallback to `created`.
- `endTime: DateTime?`: Like `startTime`, but when the video recording stopped. Due to cutting, recording pauses and etc, the `duration` is not necessarily `end - start`.
- `duration: Milliseconds` 🟦: duration of the event. As specified in ["assets"](./assets), this needs to always match the duration of all non-internal tracks.
- `updated: Timestamp` 🟦: Timestamp of when anything about this event was last changed.
- `created: Timestamp` 🟦: Timestamp of when the event was created in Opencast. It is set once when the event is first stored in Opencast's DB, and never changed again. This also implies that scheduled event's `created` date is when the scheduling took place, _not_ the time it is scheduled for (that would be `startDate`)

#### Flags
- `explicitContent: bool`: specifies whether this event contains content that is considered "explicit", like swear words or whatnot. This is required for some integrations like iTunes.
- `isLive: bool` 🟦: TODO this is currently stored per track, figure out if that's useful
- `ingestUser: Username` 🟦: username of the user that created this event. Cannot be changed and is useful for tracking responsibility.
- `downloadable: bool`: a flag indicating whether users are allowed to download this video (i.e. tracks attached to this event). This can inform external apps whether to show a download button or to enable anti-download protection. The exact effects of this flag are deliberately unspecified, this merely states an *intend*.
- `listed: bool`: specifies whether this event should be considered "list", meaning that users can find it via search. If it is `false`, users have to know the ID of the event (e.g. via a series or playlist) in order to access it.

#### Extra metadata
- `extraMetadata: Map<Label, ???>`: additional metadata that Opencast never interprets, but just stores and passes along.<sup>(1?)</sup>
  - The keys of this map consists of a _namespace_ and a _field name_, separated by `:`, i.e. `ns:name`. Both parts must consist of only `a-z`, `A-Z`, `0-9`, `-` and `_`.
  - The namespace `dct` is special as it refers to the Dublin Core Terms specification, e.g. `dct:rightsHolder` refers to [the `rightsHolder` property](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/rightsHolder) of DC terms. Also see [the DC mapping section below](#dublin-core-mapping). It should be avoided to set fields that already have a mapping, like `dct:title`, which is mapped to the OC core metadata `title`.
  - Unlike the "extended metadata" before, using `extraMetadata` does work out of the box and does not incur any relevant performance overhead. Therefore, applications are encouraged to add useful data here, e.g. `studip:course-id`, `oc-studio:version` or `ethz:room-number`.
  - There should be a community resource for collecting used fields and best practices around `extraMetadata`. That way, common requirements are identified quickly and the community can converge towards a standard.



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
- [`modified`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/modified): `updated`
- [`extend`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/extent): `duration` as ISO 8601 duration
- `date` or `temporal`???: TODO combination of `startDate` and `endDate`
- `created` or `dateSubmitted` or `issued`???: TODO `created`
- Additionally, all fields in `extraMetadata` with `dct` namespace are mapped directly to the corresponding property, e.g. `dct:license` is mapped to [`license`](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#http://purl.org/dc/terms/license).

TODO: Should we also define a mapping for OAIMPH? Does that make sense?

---

:::danger[Open questions]

- (1?) What values do we want to allow? `string` or string arrays are required, but maybe allow numbers? bools? Arbitrary JSON?
:::
