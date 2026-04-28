# statement_unstable_add

**Parameters**:

- `followSubscription`: An opaque string that was returned by `statement_unstable_follow`.
- `topicFilter`: A topic filter indicating which statements to track on this follow.

The `topicFilter` parameter can be one of:

- `"any"` — matches all statements regardless of their topics.

- `{"matchAll": ["0x...", ...]}` — matches only statements that include all of the provided topics. Each topic is a hexadecimal-encoded 32-byte value. Up to 4 topics can be provided.

**Return value**: Either a string containing an opaque value representing the filter, or:

```json
{
  "result": "limitReached"
}
```

The string returned when this function succeeds is opaque and its meaning can't be interpreted by the JSON-RPC client. It is only unique in the context of the `followSubscription` passed to this function, and is only meant to be passed later to `statement_unstable_remove` together with that same `followSubscription`.

When this function succeeds, the JSON-RPC server attaches the given filter to the follow, returns a `filterId`, and starts replaying on the corresponding `statement_unstable_follow` subscription the statements that are already present in the store and match this filter. These replayed statements are reported through `replayStatements` notifications carrying that `filterId`. If the follow remains alive and the filter isn't removed, the server later emits a `replayDone` notification carrying that same `filterId`, even if no statement matched.

If this function returns `{"result": "limitReached"}`, then the JSON-RPC server hasn't accepted that filter. No filter is attached, no replay starts, and no notification is generated because of that call.

This replay is a snapshot of the statements already present in the store when the JSON-RPC server accepts the call. Statements that enter the store later are not part of this replay. If such later statements match one or more filters whose replays are still in progress on this follow, they are delayed and are later reported through `newStatements` once all the matching in-progress replays have finished, have been removed, or have been interrupted by a `stop`.

If the same filter is added multiple times, each successful call creates a new replay pass and can therefore re-emit statements that were already emitted earlier on the same follow.

This specification doesn't define any specific limit to the number of active filters attached to a follow. If the JSON-RPC server can't accept the requested filter on the given follow, it returns `{"result": "limitReached"}`.

If, after accepting a filter, the JSON-RPC server can no longer preserve the guarantees of `statement_unstable_follow`, it must emit a `stop` event on that follow.

## Possible errors

- A JSON-RPC error with error code `-32801` is generated if the `followSubscription` doesn't correspond to any active `statement_unstable_follow` subscription.
- A JSON-RPC error with error code `-32602` is generated if `topicFilter` doesn't correspond to the expected format, if one of the topics isn't 32 bytes long, or if more than 4 topics are provided in `matchAll`.
- A JSON-RPC error with error code `-32603` is generated if the JSON-RPC server encounters an unexpected internal failure while creating the filter.
