# bitswap_v1_getMany

**Parameters**:

- `cids`: Array of CID strings, in [string format](https://github.com/multiformats/cid/blob/edb1c5294ad2d8257812d7ded4941c3e0fafccf3/README.md#variant---stringified-form).  
  Each entry must satisfy the same constraints as the `cid` parameter of [`bitswap_v1_get`](bitswap_v1_get.md): CIDv1 in `base32` multibase encoding (string starting from `b...`), with the `sha2-256` or `blake2b-256` hash function.  
  The maximum number of CIDs accepted in a single request is implementation-defined; an implementation must accept at least 16 CIDs and may accept more.

**Return value**: Array of `[cid, BlockResult]` tuples, in the same order as the input. The i-th entry
of the returned array corresponds to the i-th input CID.

`cid` is the input CID string, returned verbatim so that the caller can build a `Map<cid, BlockResult>`
without keeping the input array around.

`BlockResult` is either:

**Ok**:

A hex-encoded string starting with `0x...` containing the chunk data, e.g.

```json
"0x4869"
```

Same encoding as the return value of [`bitswap_v1_get`](bitswap_v1_get.md).

**Err**:

```json
{ "code": -32810, "message": "..." }
```

`code` carries the same [error categories](bitswap_v1_get.md#error-categories) as the top-level error
of [`bitswap_v1_get`](bitswap_v1_get.md). Clients that already know how to interpret `code` for
`bitswap_v1_get` can reuse the same retry logic per-CID.

`message` is a human-readable diagnostic string for logs and developer-facing tooling. Only `code` is
stable for programmatic dispatch. 

A missing or invalid CID at any index returns an `Err` at that index — it does **not** abort the rest
of the batch. The caller can re-issue the call with just the failed CIDs.

The maximum returned size per individual chunk is 4 MiB + 2 B (same per-chunk limit as
`bitswap_v1_get`); the total response size is bounded by the per-chunk limit times the implementation's
maximum CID count.

## Example response

For an input array `["<cidA>", "<cidB>", "<cidC>"]` where retrieval of `<cidB>` times out:

```json
[
  ["<cidA>", "0x4869"],
  ["<cidB>", { "code": -32811, "message": "request timeout" }],
  ["<cidC>", "0x6f6b"]
]
```

The response array preserves input order. The caller can re-issue the call with just `<cidB>`.

## Possible errors

See [error categories](bitswap_v1_get.md#error-categories).

## Empty input

An empty input array (`cids: []`) is allowed and returns an empty result array. No CID is fetched.

## Duplicate input

The implementation rejects calls whose input contains the same CID more than once. Detection is
two-stage:

1. Two literally-identical input strings (regardless of whether they are valid CIDs) → rejected.
2. Two cosmetically different CID strings that decode to the same 32-byte content digest →
   rejected.

In either case the call is rejected at the top level with `-32602 InvalidParams`. Callers must
deduplicate input before calling.

Duplicate inputs are rejected so that [`bitswap_v1_stream`](bitswap_v1_stream.md) can guarantee
exactly one event per input CID — the client uses the input array length to determine when the
stream is exhausted. `bitswap_v1_getMany` enforces the same rule for symmetry.
