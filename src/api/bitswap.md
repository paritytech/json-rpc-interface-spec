# Introduction

The functions with `bitswap` prefix allow fetching data chunks from the storage given their CID,
normally downloaded using Bitswap protocol.

If these methods are called on a full node, the data is returned from a local database. If the
methods are called on a light client, it tries to download the data via Bitswap protocol from full
nodes.

The full node may also implement downloading the data from other full nodes via Bitswap protocol
if it doesn't have the data locally (e.g., due to the block pruning settings).

The namespace exposes four methods:

- [`bitswap_v1_get`](bitswap_v1_get.md) — fetch a single chunk by CID. Errors are reported at the
  JSON-RPC level.
- [`bitswap_v1_getMany`](bitswap_v1_getMany.md) — fetch a batch of chunks in a single call. The
  response is an array with one outcome per input CID, in input order. Per-CID errors are embedded
  in the response so that one failure does not abort the rest of the batch.
- [`bitswap_v1_stream`](bitswap_v1_stream.md) — like `bitswap_v1_getMany`, but exposes the per-CID
  outcomes via a JSON-RPC subscription so the client can start processing as chunks arrive.
- [`bitswap_v1_unstream`](bitswap_v1_unstream.md) — cancel an active `bitswap_v1_stream`
  subscription.

The three fetch methods share the same set of error categories. The per-CID error shape used by
`bitswap_v1_getMany` and `bitswap_v1_stream` is identical to the top-level error shape of
`bitswap_v1_get`.
