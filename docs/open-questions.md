# Open questions

- Should all data be versioned?
  - It adds complexity, but having access to old data is nice.
  - Storage wise, keeping old metadata does not cost much.
  - Via the `internal` asset system, we can already kind of version assets.
  - Get rid of the current asset manager/snapshot system to avoid hardlinks.
- What do we generally think about size limitations for various fields?
  - Abuse protection: this is just to prevent abuse, DOS, slow downs and stuff like that. Limit `description` to 2<sup>16</sup> bytes, limit `title`, `license`, ... to 1024 bytes. I think these limits make sense and should prevent OC suffering from bad payloads.
  - Semantic limits: for example, for `license`, we could say "it should just be a identifier for a license, so limit to 64 bytes". This is a lot more tricky as one has to really think of the intended use case and runs the risk of making use cases impossible.


## TODO

- Metadata can be changed when a workflow is running
  - Mhhh small problem: some workflows might depend on metadata, e.g. when creating images with metadata in them. So maybe workflows can declare dependencies to metadata?
  - So maybe we cannot do this now, this feature we can still add in a second step. When we rework the workflow system 😈
- Explain how snapshots are removed
