---
sidebar_position: 2
---

# Common types

These are types used throughout the rest of this specification and defined here once to avoid repetition.

- `string`: a valid UTF-8 string. While being processed in code, it might be in a different encoding temporarily, but in the public interface of Opencast, these are always valid UTF-8.
- `NonBlankString`: **N**on-**E**mpty string.
- `Neas`: **N**on-**E**mpty **A**SCII **S**tring.
- `Label`: non-empty, ASCII-only string that only consists of letters, numbers or `-._~!*:@,;`. This means a label is URL-safe except for use in the domain part.<sup>(2?)</sup>
- `ID`: a `Label` that cannot be changed after being created.
- `Username`: TODO define rules for usernames
- `LangCode`: specifies a language and optionally a region, e.g. `en` or `en-US`. Based on the [IETF BCP 47 language tag specification](https://www.rfc-editor.org/info/rfc5646): a two letter language code, optionally followed by a hyphen and a two letter region tag.
- `int8`, `int16`, `int32`, `int64`: signed integers of specific bit width.
- `uint8`, `uint16`, `uint32`, `uint64`: unsigned integers of specific bit width.<sup>(1?)</sup>
- `Milliseconds`: a `uint64` representing a duration or a video timestamp in milliseconds (ms). Impl note: whenever possible, in code, this should be a custom type and not just `int`.
- `DateTime`: a date + time with timezone, i.e. a specific moment in a specific timezone.
- `Timestamp`: a specific moment in time, without time zone (e.g. always stored as UTC).

Generally, this basically uses TypeScript syntax:

- `T?`: denotes an optional type, i.e. `bool?` means the field could be either `true`, `false` or undefined. All fields without `?` are _required_ / `non null`.
- `T[]`: array of type `T`.
- `[T, U, ...]`: a tuple of values.
- `"foo" | "bar"`: one of the listed constant values.

## JSON serialization

For most types, the JSON serialization is the obvious one, but there are some minor important details.
- `bool` as `bool`
- `string` and all "string with extra requirements" (e.g. `Label`, `ID`, `Neas`) as string
- Integers as number.
  - Note on 64 bit integers: In JavaScript, there is only one `number` type, which is a 64 bit floating point number (`double`, `f64`).
  Those can only exactly represent integers up to 2<sup>53</sup>.
  While JSON is closely related to JS, the format itself is allowed to exceed `f64` precision and may in fact encode arbitrary precision numbers.
  Opencast should serialize a 64 bit integer as exact integer into JSON and *not* rounded like an `f64`.
  Rounding might happen in the frontend, but the API should emit the exact integer value.
- Arrays as arrays
- Tuples as arrays
- `Map<string, string>` is serialized as object
- `DateTime`: as ISO 8601-compatible formatted string. The ISO standard actually allows a number of different formats by ommitting parts of the string. Opencast shall format all date times as either `YYYY-MM-DDTHH:mm:ss.sssZ` or `YYYY-MM-DDTHH:mm:ssZ`, i.e. only the sub-second part is optional. The parts on this format string are best described in [the ECMAScript specification](https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-date-time-string-format) (which again, is a subset of ISO 8601). Only thing of note: `Z` could either be literal `Z` or a timezone offset like `+02`.
- `Timestamp`: like `DateTime` but always in UTC, so always ending with literal `Z`.

---

:::danger[Open questions]

- (1?) Java famously has no/bad support for unsigned integers. Decide how to deal with that: do we just give up one bit or do we require proper unsigned usage via `Integer.*Unsigned` methods? Either way: these values must never be negative!
- (2?) Maybe disallow more of these special characters?

:::
