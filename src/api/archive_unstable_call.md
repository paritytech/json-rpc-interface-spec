# archive_unstable_call

**Parameters**:

- `hash`: String containing the hexadecimal-encoded hash of the header of the block to make the call against.
- `function`: Name of the runtime entry point to call as a string.
- `callParameters`: Hexadecimal-encoded SCALE-encoded value to pass as input to the runtime function.
- `networkConfig` (optional): Object containing the configuration of the networking part of the function. See [here](./api.md) for details. Ignored if the JSON-RPC server doesn't need to perform a network request. Sensible defaults are used if not provided.

**Return value**: Hexadecimal-encoded output of the runtime function call.

The JSON-RPC server must invoke the entry point of the runtime of the given block using the storage of the given block.

**Note**: The runtime is still allowed to call host functions with side effects, however these side effects must be discarded. For example, a runtime function call can try to modify the storage of the chain, but this modification must not be actually applied. The only motivation for performing a call is to obtain the return value.

Use `chainHead_unstable_follow` if you want to call the runtime of recent chain blocks.

**Note**: This can be used as a replacement for the legacy `state_getMetadata`, `system_accountNextIndex`, and `payment_queryInfo` functions.

## Possible errors

- A JSON-RPC error is generated if the networking part of the behaviour fails.
- A JSON-RPC error is generated if the block hash passed as parameter doesn't correspond to any block of the archive node.
- A JSON-RPC error is generated if the method to call doesn't exist in the Wasm runtime of the chain.
- A JSON-RPC error is generated if the runtime call fails (e.g. because it triggers a panic in the runtime, running out of memory, etc., or if the runtime call takes too much time)

## About `callParameters`

Runtime entry points typically accept multiple input parameters, but this JSON-RPC function accepts as parameter a single hexadecimal-encoded value. The reason is that all the parameters are concatenated together before performing the call anyway. There is no reason to force JSON-RPC clients to provide a proper division between the various parameters if they are all concatenated in the implementation anyway.
