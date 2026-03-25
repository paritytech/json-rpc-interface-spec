# Introduction

The functions with `bitswap` prefix allow fetching data chunks from the storage given their CID,
normally downloaded using Bitswap protocol.

If these methods are called on a full node, the data is returned from a local database. If the
methods are called on a light client, it tries to download the data via Bitswap protocol from full
nodes.

The full node may also implement downloading the data from other full nodes via Bitswap protocol
if it doesn't have the data locally (e.g., due to the block pruning settings).
