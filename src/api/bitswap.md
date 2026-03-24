# Introduction

The functions with `bitswap` prefix allow fetching data chunks from the storage given their CID,
normally downloaded using Bitswap protocol.

If these methods are called on a full node, the data is returned from a local database. If the
methods are called on a light client, it tries to download the data via Bitswap protocol from full
nodes.
