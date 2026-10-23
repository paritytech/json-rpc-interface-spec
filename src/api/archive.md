# Introduction

Functions with the `archive` prefix allow obtaining the state of the chain at any point in the present or in the past.

These functions are meant to be used to inspect the history of a chain. They can be used to access recent information as well, but JSON-RPC clients should keep in mind that the `chainHead` functions could be more appropriate.

These functions are typically expensive for a JSON-RPC server, because they likely have to perform either disk accesses or network requests. Consequently, JSON-RPC servers are encouraged to put a global limit on the number of concurrent calls to `archive`-prefixed functions.

# Usage

The JSON-RPC server exposes a finalized block height, which can be retrieved by calling `archive_v1_finalizedHeight`.

Call `archive_v1_hashByHeight` in order to obtain the hash of a block by its height.

If the height passed to `archive_v1_hashByHeight` is inferior or equal to the value returned by `archive_v1_finalizedHeight`, then it is always guaranteed that there is exactly one block with this hash.
The JSON-RPC client can then call `archive_v1_header`, `archive_v1_body`, `archive_v1_storage`, and `archive_v1_call` in order to obtain details about the block with this hash. It is always guaranteed to return a value.

If the height passed to `archive_v1_hashByHeight` is strictly superior to the value returned by `archive_v1_finalizedHeight`, then `archive_v1_hashByHeight` might return zero, one, or more blocks. Furthermore, the list of blocks being returned can change at any point. It is also possible to call `archive_v1_header`, `archive_v1_body`, `archive_v1_storage`, and `archive_v1_call` on these blocks, but these functions might return `null` even if their hash was previously returned by `archive_v1_hashByHeight`.

## Transaction location queries

The `archive_unstable_findTransaction` function returns every retained block that contains a given encoded extrinsic, together with the extrinsic index and chain placement status of each occurrence.

The method is a query against the server's database, not a proof of inclusion. Clients that require trustless guarantees must verify a finality proof or re-execute the block themselves. The returned data is runtime-neutral and contains no events, fees, weight, or outcome flags; events for a returned occurrence can be obtained by querying `archive_v1_storage` against the returned block hash.

Implementing this method requires the server to maintain an index over the extrinsic bytes of every block it retains, which Substrate nodes do not do by default. A server that does not maintain such an index must not advertise this method in `rpc_methods`.
