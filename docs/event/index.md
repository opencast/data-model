---
sidebar_position: 4
---

# Event

An event<sup>(1?)</sup> is the core entity of Opencast, representing a multimedia content.
An event consists of:
- [Metadata](./metadata)
- [ACL](./acl)
- [Assets](./assets)

As described [here](../common#data-storage), almost all of this data is stored in the DB.
Only the actual asset files are stored on the file system (the metadata about assets is still stored in the DB).

In terms of API response, it might look like this:


---

:::danger[Open questions]

- (1?) Potentially very controversial: rename "event"/"episode" to "video"?
  - Intuitively, most people call it "video"
  - "Event" is a very generic term and can mean many other things, "episode" implies being part of a series.
  - Yes, there can be two _video files_, but we already have a name for that: video stream. So Idon't see a confusion risk here. I don't see any problems with calling a thing a video even if it contains two video streams.
  - New name in API would make clear that data model has changed.
:::
