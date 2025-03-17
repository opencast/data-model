---
sidebar_position: 2
---

# Metadata

## Data stored per event

TODO: think about good terms for this.
"Metadata" is a very overloaded term in the OC community.
As you see below, there are different kinds of data here.
Do we want to call all of it metadata? Or maybe just this first section?

### Fully user-editable, without behavior

These attributes are basically freely editable by users (except for basic validity checks).
Further, while these fields have are recognized by Opencast and will be displayed accordingly, they do not have any effect on behavior.
They are just stored, passed around and displayed.

- `title: NonBlankString`
- `description: string?`
- `language: LangCode?`: describes the (main) language of this event and its metadata. For example, the audio language and (if applicable) language of video content is more important than the language of available subtitles. Generally, assets can have their own language specified.
- `license: NonBlankString?`: identifier/name for an existing license (e.g. `CC-BY`)
- `source: NonBlankString?`: describes where an event originates (e.g. an URL or a citation)
- `rights: ???`: TODO what is that exactly? What is it used for?
- `subject: ???`: TODO what is that exactly? What is that used for?
- `location: NonBlankString?`: Physical location of where the video was created (e.g. room number or country name)
- `creators: NonBlankString[]`: The people mainly responsible for creating this video and/or presenting the talk which this video is a recording of. Should contain human-readable names and not usernames.
- `contributors: NonBlankString[]`: TODO describe semantic meaning of this
- `publisher: ???`: TODO describe semantic meaning of this
- `startTime: DateTime?`: Actual real life datetime when the video recording started, with timezone. If this is not really a recording (e.g. short movie), this should be undefined.
- `endTime: DateTime?`: Actual real life datetime when the video recording stopped, with timezone. Due to cutting, recording pauses and etc, the `duration` is not necessarily `end - start`.
- `explicitContent: bool`: specifies whether this event contains content that is considered "explicit", like swear words or whatnot. This is required for some integrations like iTunes.
- `extraMetadata: Map<Label, ???>`: user-specified extra metadata that Opencast ignores, i.e. just passes along.<sup>(2?) (3?)</sup>
- TODO: something like "ingest software"? A place where we can put "Opencast Studio" or "Tobira" or "Stud.IP"?
- TODO: legal owner? ETH is interested in something like this, see https://github.com/elan-ev/tobira/issues/870

### Partially user-editable, influence behavior
- `series: SeriesID?`: optional ID of the series this event belongs to. Must be a valid series ID of an existing series at all time.
- `downloadable: bool`: a flag indicating whether users are allowed to download this video (i.e. tracks attached to this event). This can inform external apps whether to show a download button or to enable anti-download protection. The exact effects of this flag are deliberately unspecified, this merely states an *intend*.
- `listed: bool`: specifies whether this event should be considered "list", meaning that users can find it via search. If it is `false`, users have to know the ID of the event (e.g. via a series or playlist) in order to access it.
- `owner: Username`: TODO figure out details

### Fully Opencast managed

These attributes cannot be directly changed by users, but are managed by Opencast.

- `id: ID`: unique among all events
- `duration: Milliseconds`: duration of the event. As specified in ["assets"](./assets), this needs to always match the duration of all non-internal tracks.
- `updated: Timestamp`: Timestamp of when anything about this event was last changed.
- `created: Timestamp`: Timestamp of when the event was created in Opencast.<sup>1?</sup>
- `isLive: bool`: TODO this is currently stored per track, figure out if that's useful
- `ingestUser: Username`: username of the user that created this event. Cannot be changed and is useful for tracking responsibility.


## Dublin Core mapping

As you can see above, the data we store is not a Dublin Core catalog anymore.
Opencast defines metadata fields and their semantics for itself.
Dublin Core should be considered an *exchange format*, not a storage format!
Of course, standards make sense and therefore, many Opencast metadata fields correspond exactly to a Dublin Core field.
Opencast can offer an API that emits a DCC for an event.
That DCC XML is created on the fly in the API handler.

The following describes how Opencast fields are mapped to Dublin Core:
- TODO

TODO: describe the same for OAIMPH

---

:::danger[Open questions]

- (1?) Should this ignore scheduling? Or should we always record the "created in Opencast", since scheduled events will have `startDate` set?
- (2?) We want some form of namespacing to make it easier to avoid collisions. How exactly do we do that? Just in the form of `namespace:key` as key in the map? E.g. `"pyca:cpu-usage": 0.45`
- (3?) What values do we want to allow? Just `string`s? String arrays? Arbitrary JSON?
:::
