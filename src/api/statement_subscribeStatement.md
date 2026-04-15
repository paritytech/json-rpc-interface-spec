# statement_subscribeStatement

**Parameters**:

- `topicFilter`: A topic filter indicating which statements to subscribe to.

The `topicFilter` parameter can be one of:

- `"any"` — matches all statements regardless of their topics.

- `{"matchAll": ["0x...", ...]}` — matches only statements that include **all** of the provided topics. Each topic is a hexadecimal-encoded 32-byte value. Up to 4 topics can be provided.

- `{"matchAny": ["0x...", ...]}` — matches statements that include **any** of the provided topics. Each topic is a hexadecimal-encoded 32-byte value. Up to 128 topics can be provided.

**Return value**: String representing the subscription.

The string returned by this function is opaque and its meaning can't be interpreted by the JSON-RPC client. It is only meant to be matched with the `subscription` field of notifications and potentially passed to `statement_unsubscribeStatement`.

## Initial batch

When a subscription is initiated, the endpoint first returns all matching statements already in the store as one or more `newStatements` notifications. Each notification in this initial batch includes a `remaining` field indicating how many more statements are guaranteed to follow. The client will receive at least this many more statements, but may receive more if new statements are added to the store that match the filter during delivery.

If there are no matching statements in the store, an empty batch is sent (a `newStatements` notification with an empty `statements` array).

After the initial batch is delivered, subsequent `newStatements` notifications are generated whenever new statements matching the filter are added to the store.

## Notifications format

This function will generate one or more notifications in the following format:

```json
{
    "jsonrpc": "2.0",
    "method": "statement_statement",
    "params": {
        "subscription": "...",
        "result": ...
    }
}
```

Where `subscription` is the value returned by this function, and `result` can be:

### newStatements

```json
{
    "event": "newStatements",
    "data": {
        "statements": ["0x...", "0x..."],
        "remaining": 5
    }
}
```

Or, when no more statements are guaranteed to follow from the initial batch:

```json
{
    "event": "newStatements",
    "data": {
        "statements": ["0x...", "0x..."]
    }
}
```

Each entry in `statements` is a hexadecimal-encoded SCALE-encoded statement.

The optional `remaining` field is an integer indicating the minimum number of additional statements the client is guaranteed to receive from the initial batch. This field is only present during the initial delivery of existing statements and is absent for notifications about newly added statements.

## Possible errors

- A JSON-RPC error with error code `-32602` is generated if the `topicFilter` parameter is invalid (e.g., wrong format, topics are not 32 bytes, or exceeds the maximum number of topics).
