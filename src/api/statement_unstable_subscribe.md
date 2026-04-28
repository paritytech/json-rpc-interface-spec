# statement_unstable_subscribe

**Parameters**: _none_

**Return value**: String containing an opaque value representing the subscription.

This function lets the JSON-RPC client subscribe to statements known to the statement store.

This function works as follows:

- When called, returns an opaque `subscription` that is later used to match notifications and to call `statement_unstable_add_filter`, `statement_unstable_remove_filter`, or `statement_unstable_unsubscribe`.

- The subscription starts with no filters attached. As such, this function doesn't immediately generate notifications.

- Later, each successful call to `statement_unstable_add_filter` on this `subscription` causes the subscription to generate `replayStatements` notifications for statements already present in the store and matching the newly-added filter, followed by a `replayDone` notification once that replay is complete. Calls to `statement_unstable_add_filter` that return `{"result": "limitReached"}` mean that the requested filter wasn't accepted, and they don't generate notifications.

- Afterwards, whenever a new statement enters the store and matches at least one active filter on this subscription, the subscription eventually generates a `newStatements` notification once doing so wouldn't violate the replay guarantees below.

- If the JSON-RPC server can no longer preserve the guarantees of this page, it generates a `stop` notification indicating that the subscription is now dead and must be re-created. No more notifications will be sent out on this subscription.

The subscription can later be stopped by calling `statement_unstable_unsubscribe`.

## Notifications format

This function will later generate one or more notifications in the following format:

```json
{
    "jsonrpc": "2.0",
    "method": "statement_unstable_subscribeEvent",
    "params": {
        "subscription": "...",
        "result": ...
    }
}
```

Where `subscription` is the value returned by this function, and `result` can be one of:

### replayStatements

```json
{
  "event": "replayStatements",
  "filterId": "...",
  "statements": ["0x...", "0x..."]
}
```

The `replayStatements` event indicates statements emitted because of a specific successful call to `statement_unstable_add_filter`.

`filterId` is the opaque string returned by that call to `statement_unstable_add_filter`. It is only unique in the context of this `subscription`.

Each entry in `statements` is a hexadecimal-encoded SCALE-encoded statement.

The `statements` array always contains at least one entry.

No guarantee is offered as to the order of the statements within the array.

### replayDone

```json
{
  "event": "replayDone",
  "filterId": "..."
}
```

The `replayDone` event indicates that the replay caused by a specific successful call to `statement_unstable_add_filter` is complete.

`filterId` is the opaque string returned by that call to `statement_unstable_add_filter`. It is only unique in the context of this `subscription`.

If the subscription remains alive and the filter isn't removed before replay completion, then exactly one `replayDone` event is generated for each successful `statement_unstable_add_filter`, even if no `replayStatements` event was generated because no statement matched the filter.

### newStatements

```json
{
  "event": "newStatements",
  "statements": [
    {
      "statement": "0x...",
      "filterIds": ["...", "..."]
    },
    {
      "statement": "0x...",
      "filterIds": ["..."]
    }
  ]
}
```

Each entry in `statements` is an object with the following fields:

- `statement`: hexadecimal-encoded SCALE-encoded statement.
- `filterIds`: array containing the ids of the filters that are active on this subscription when the notification is emitted and that match this statement.

Each `filterId` is opaque and is only unique in the context of this `subscription`.

Each `statements` array always contains at least one entry.

Each `filterIds` array always contains at least one entry.

No guarantee is offered as to the order of the `statements` array.

No guarantee is offered as to the order of the `filterIds` array.

The same `filterId` must not appear more than once in the `filterIds` array of a given statement item.

The `newStatements` event indicates statements that have newly entered the store.

### stop

```json
{
  "event": "stop"
}
```

The `stop` event indicates that the JSON-RPC server can no longer preserve the guarantees of this page. This can happen for example because the subscription consumes an unreasonable amount of memory, because the JSON-RPC client is too slow to pull notifications, or because the server is otherwise overloaded.

No more event will be generated with this `subscription`.

Calling `statement_unstable_unsubscribe` on a subscription that has produced a `stop` event is optional.

## Delivery guarantees

A newly inserted statement is emitted at most once per subscription through `newStatements`, even if multiple active filters match it.

Each successful call to `statement_unstable_add_filter` creates a new replay pass over the statements currently known to the store and matching that filter. Re-adding the same filter creates a new replay pass and can therefore re-emit statements that were already emitted earlier on the same subscription. Statements emitted because of such a replay pass are reported through `replayStatements` with the corresponding `filterId`, and the replay completion is reported through `replayDone` with that same `filterId`.

For a given successful `statement_unstable_add_filter`, all statements that were already present in the store when the server accepted that call and that match its filter must be emitted before the corresponding `replayDone`.

Statements that enter the store after a given successful `statement_unstable_add_filter` has been accepted must never be emitted through the `replayStatements` of that `statement_unstable_add_filter`, even if they match its filter.

If a newly inserted statement matches one or more filters whose replays are still in progress on the same subscription, then this statement must not be emitted through `newStatements` until all these matching in-progress replays have emitted their corresponding `replayDone`, have been removed with `statement_unstable_remove_filter`, or have been interrupted by a `stop`.

When such a waiting statement is finally emitted through `newStatements`, its `filterIds` must contain the ids of all the filters that are active on the subscription at emission time and that match that statement, including filters that were added after the statement entered the store.

As a consequence, a statement that is emitted through `replayStatements` for a given `filterId` must not later be emitted through `newStatements` with that same `filterId`.

If a subscription emits `stop` before replay completion, then no `replayDone` is required for the filters whose replay hasn't completed.

JSON-RPC servers are allowed to delay live notifications in order to preserve the guarantees above.

## Multiple subscriptions

The JSON-RPC server must accept at least 2 `statement_unstable_subscribe` subscriptions per JSON-RPC client. Trying to open more might lead to a JSON-RPC error when calling `statement_unstable_subscribe`. In other words, as long as a JSON-RPC client starts 2 or fewer `statement_unstable_subscribe` subscriptions, it is guaranteed that this return value will never happen.

If multiple `statement_unstable_subscribe` subscriptions exist at the same time, then each of them has its own filters, its own replay state, and its own deduplication state.

## Possible errors

- A JSON-RPC error with error code `-32800` can be generated if the JSON-RPC client has already opened 2 or more `statement_unstable_subscribe` subscriptions.
