# Introduction

The functions with `bitswap` prefix allow fetching data chunks  given their CID, normally downloaded using Bitswap protocol.

Depending on type of node or implementation the data is returned from a local database or downloaded via Bitswap protocol from other peers.

The namespace exposes three methods:

- [`bitswap_unstable_get`](bitswap_unstable_get.md) — fetch a single chunk by CID. Errors are reported at the JSON-RPC level.
- [`bitswap_unstable_stream`](bitswap_unstable_stream.md) — fetch a batch of chunks via a JSON-RPC subscription. Per-CID outcomes are delivered as events as chunks arrive, so the client can start processing before the whole batch resolves.
- [`bitswap_unstable_unstream`](bitswap_unstable_unstream.md) — cancel an active `bitswap_unstable_stream` subscription.
