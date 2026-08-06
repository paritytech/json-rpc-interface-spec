# statement_unstable_add_filter

**Parameters**:

- `subscription`: An opaque string that was returned by `statement_unstable_subscribe`.
- `topicFilter`: A topic filter indicating which statements to track on this subscription.

The `topicFilter` parameter can be one of:

- `"any"` — matches all statements regardless of their topics.

- `{"matchAll": ["0x...", ...]}` — matches only statements that include all of the provided topics. Each topic is a hexadecimal-encoded 32-byte value. Between 1 and 4 topics must be provided.

**Return value**: Either a string containing an opaque value representing the filter, or:

```json
{
  "result": "limitReached"
}
```

The string returned when this function succeeds is opaque and its meaning can't be interpreted by the JSON-RPC client. It is only unique in the context of the `subscription` passed to this function, and is only meant to be passed later to `statement_unstable_remove_filter` together with that same `subscription`.

When this function succeeds, the JSON-RPC server attaches the given filter to the subscription, returns a `filterId`, and starts replaying on the corresponding `statement_unstable_subscribe` subscription the statements that are already present in the store and match this filter. These replayed statements are reported through `replayStatements` notifications carrying that `filterId`. If the subscription remains alive and the filter isn't removed, the server later emits a `replayDone` notification carrying that same `filterId`, even if no statement matched.

If this function returns `{"result": "limitReached"}`, then the JSON-RPC server hasn't accepted that filter. No filter is attached, no replay starts, and no notification is generated because of that call.

This replay is a snapshot of the statements already present in the store when the JSON-RPC server accepts the call. Statements that enter the store later are not part of this replay. If such later statements match one or more filters whose replays are still in progress on this subscription, they are delayed and are later reported through `newStatements` once all the matching in-progress replays have finished, have been removed, or have been interrupted by a `stop`.

If the same filter is added multiple times, each successful call creates a new replay pass and can therefore re-emit statements that were already emitted earlier on the same subscription.

This specification doesn't define any specific limit to the number of active filters attached to a subscription. If the JSON-RPC server can't accept the requested filter on the given subscription, it returns `{"result": "limitReached"}`.

If, after accepting a filter, the JSON-RPC server can no longer preserve the guarantees of `statement_unstable_subscribe`, it must emit a `stop` event on that subscription.

## Possible errors

- A JSON-RPC error with error code `-32801` is generated if the `subscription` doesn't correspond to any active `statement_unstable_subscribe` subscription.
- A JSON-RPC error with error code `-32602` is generated if `topicFilter` doesn't correspond to the expected format, if `matchAll` contains fewer than 1 or more than 4 topics, or if one of the topics isn't 32 bytes long.
- A JSON-RPC error with error code `-32603` is generated if the JSON-RPC server encounters an unexpected internal failure while creating the filter.
