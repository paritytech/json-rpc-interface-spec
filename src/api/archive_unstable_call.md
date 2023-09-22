# archive_unstable_call

**Parameters**:

- `hash`: String containing the hexadecimal-encoded hash of the header of the block to make the call against.
- `function`: Name of the runtime entry point to call as a string.
- `callParameters`: Hexadecimal-encoded SCALE-encoded value to pass as input to the runtime function.

**Return value**:

- If no block with that `hash` exists, returns `null`.
- If the call was successful, the hexadecimal-encoded SCALE-encoded value returned by the runtime.

The JSON-RPC server must invoke the entry point of the runtime of the given block using the storage of the given block.

**Note**: The runtime is still allowed to call host functions with side effects, however these side effects must be discarded. For example, a runtime function call can try to modify the storage of the chain, but this modification must not be actually applied. The only motivation for performing a call is to obtain the return value.

If the height of the block hash provided is less than or equal to the current finalized block height (which can be obtained via `archive_unstable_finalizedHeight`), then calling this method multiple times is guaranteed to always return non-null and always the same result (except for the `error` message which is allowed to change).

If the height of the block hash provided is greater than the current finalized block height, then the block might be pruned at any time and calling this method may return null.

## Possible errors

- A JSON-RPC error if the provided parameters are invalid.
- A JSON-RPC error containing a human-readable message if the call wasn't successful, the provided runtime function doesn't exist, or the runtime crashes, or similar. The `error` isn't meant to be shown to end users, but is for developers to understand the problem.
