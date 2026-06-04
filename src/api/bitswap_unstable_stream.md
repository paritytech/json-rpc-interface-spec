# bitswap_unstable_stream

**Parameters**:

- `cids`: Non-empty array of CID strings, in [string format](https://github.com/multiformats/cid/blob/edb1c5294ad2d8257812d7ded4941c3e0fafccf3/README.md#variant---stringified-form).  
  Each entry must satisfy the same constraints as the `cid` parameter of [`bitswap_unstable_get`](bitswap_unstable_get.md): CIDv1 in `base32` multibase encoding (string starting from `b...`), with the `sha2-256` or `blake2b-256` hash function.  
  The maximum number of CIDs accepted in a single subscription is implementation-defined; an implementation must accept at least 16 CIDs and may accept more.

**Return value**: String representing the subscription.

The string returned by this function is opaque and its meaning can't be interpreted by the JSON-RPC client. It is only meant to be matched with the `subscription` field of events and potentially passed to `bitswap_unstable_unstream`.

`bitswap_unstable_stream` emits exactly one outcome per input CID, in the order results become available — **not** in input order. This lets the client begin processing the first chunks while later chunks are still in flight. The client correlates each event with its request via the `cid` field embedded in the event payload.

If `bitswap_unstable_unstream` is called or the JSON-RPC client disconnects, the implementation cancels remaining work and stops emitting events. No further events are emitted after cancellation.

## Notifications format

Notifications are emitted using the following JSON-RPC envelope:

```json
{
    "jsonrpc": "2.0",
    "method": "bitswap_unstable_streamEvent",
    "params": {
        "subscription": "...",
        "result": { "event": "...", ... }
    }
}
```

Where `subscription` is the value returned by this function, and `result` is one of the three event variants below, distinguished by the `event` field.

### streamItem

Emitted once per successfully retrieved CID.

```json
{
    "event": "streamItem",
    "cid": "<cid>",
    "value": "0x..."
}
```

`cid` is the input CID string, returned verbatim. `value` is the chunk data, hex-encoded, always starting with `0x...`. The encoding is the same as the return value of [`bitswap_unstable_get`](bitswap_unstable_get.md); the per-chunk size limit is 4 MiB + 2 B.

### streamItemError

Emitted once per CID that could not be retrieved. Emitting `streamItemError` for one CID does **not** abort the stream — outcomes for the remaining CIDs still arrive.

```json
{
    "event": "streamItemError",
    "cid": "<cid>",
    "code": -32811,
    "message": "request timeout"
}
```

`cid` is the input CID string, returned verbatim.

`code` and `message` mirror the standard JSON-RPC error-object fields. The same per-CID [error categories](bitswap_unstable_get.md#error-categories) apply: `-32602 InvalidParams`, `-32810 Fail`, `-32811 FailRetry`, `-32812 FailRetryBackoff`. Clients that already know how to interpret `code` for `bitswap_unstable_get` can reuse the same retry logic per-CID.

Only `code` is stable for programmatic dispatch. `message` is a human-readable diagnostic string for logs and developer-facing tooling, is implementation-specific, and must not be relied upon in business logic.

A missing or invalid CID in the input array produces a `streamItemError` event for that CID with `code: -32602` and does not abort the stream. Because events are emitted in arrival order, a `streamItemError` event for one CID may be emitted before, after, or interleaved with `streamItem` events for other CIDs.

### streamDone

Emitted exactly once, after exactly one `streamItem` or `streamItemError` event has been emitted per input CID. It is the explicit end-of-stream marker.

```json
{
    "event": "streamDone"
}
```

`streamDone` is not emitted when the client cancels via [`bitswap_unstable_unstream`](bitswap_unstable_unstream.md) or disconnects — in those cases the stream simply stops with no further events.

If the implementation determines that it cannot serve any of the remaining CIDs (for example, all peers disconnected), it emits a `streamItemError` for each remaining CID with an appropriate `code`, then `streamDone`. There is no separate stream-wide abort event.

## Example notifications

For an input of three CIDs `["<cidA>", "<cidB>", "<cidC>"]`, the implementation might deliver `<cidC>` first, then `<cidA>`, then `<cidB>` as an error, then the end-of-stream marker. The `result` field of each notification, in arrival order:

```json
{ "event": "streamItem", "cid": "<cidC>", "value": "0x6f6b" }
```

```json
{ "event": "streamItem", "cid": "<cidA>", "value": "0x4869" }
```

```json
{
  "event": "streamItemError",
  "cid": "<cidB>",
  "code": -32811,
  "message": "request timeout"
}
```

```json
{ "event": "streamDone" }
```

## Possible errors

Top-level subscription rejection causes no events to be emitted. The following codes apply:

| Code   | Category          | Meaning                                                                  |
| ------ | ----------------- | ------------------------------------------------------------------------ |
| -32602 | (standard)        | An RPC parameter doesn't correspond to the expected type                 |
| -32801 | `TooManyCids`     | Input array exceeded the implementation-defined maximum                  |
| -32802 | `EmptyCids`       | Input array is empty                                                     |
| -32803 | `DuplicateCids`   | Input array contains the same CID more than once (see "Duplicate input") |

Per-CID failures (whether the CID is malformed, the chunk cannot be found, or retrieval failed transiently) are surfaced via `streamItemError` events, not top-level errors. See the [error categories](bitswap_unstable_get.md#error-categories) of `bitswap_unstable_get` for the codes used in those events.

## Empty input

An empty input array (`cids: []`) is rejected at the top level with `-32802 EmptyCids` (no subscription is opened, no events are emitted).

## Duplicate input

The implementation rejects subscriptions whose input contains the same CID more than once. Detection is two-stage:

1. Two literally-identical input strings (regardless of whether they are valid CIDs) → rejected.
2. Two cosmetically different CID strings that decode to the same 32-byte content digest → rejected.

In either case the subscription is rejected at the top level with `-32803 DuplicateCids` (no events are emitted). Callers must deduplicate input before subscribing.

Duplicates are rejected so that `bitswap_unstable_stream` can guarantee exactly one `streamItem`/`streamItemError` event per input CID. The `streamDone` event is the authoritative end-of-stream marker, but the duplicate-rejection rule keeps a 1-to-1 mapping between input CIDs and per-item events.
