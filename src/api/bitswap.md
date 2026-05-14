# Introduction

The functions with `bitswap` prefix allow fetching data chunks from the storage given their CID,
normally downloaded using Bitswap protocol.

If these methods are called on a full node, the data is returned from a local database. If the
methods are called on a light client, it tries to download the data via Bitswap protocol from full
nodes.

The full node may also implement downloading the data from other full nodes via Bitswap protocol
if it doesn't have the data locally (e.g., due to the block pruning settings).

The namespace exposes three methods:

- [`bitswap_v1_get`](bitswap_v1_get.md) — fetch a single chunk by CID. Errors are reported at the
  JSON-RPC level.
- [`bitswap_unstable_stream`](bitswap_unstable_stream.md) — fetch a batch of chunks via a JSON-RPC
  subscription. Per-CID outcomes are delivered as events as chunks arrive, so the client can start
  processing before the whole batch resolves.
- [`bitswap_unstable_unstream`](bitswap_unstable_unstream.md) — cancel an active
  `bitswap_unstable_stream` subscription.

`bitswap_unstable_stream` reuses the per-CID error categories of `bitswap_v1_get` for its
`streamItemError` events, so a client that already handles `bitswap_v1_get` errors can route
per-item failures through the same retry logic.
