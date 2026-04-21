# statement_unstable_remove

**Parameters**:

- `followSubscription`: An opaque string that was returned by `statement_unstable_follow`.
- `filterId`: Opaque string equal to the string returned by a successful call to `statement_unstable_add`.

**Return value**: _null_

Stops a filter started with `statement_unstable_add`.

Has no effect if the `followSubscription` is invalid, if the `filterId` is invalid, if the `filterId` doesn't correspond to a filter attached to the given `followSubscription`, if the filter has already been removed, or if the given follow has already emitted a `{"event": "stop"}` notification.

If this filter was still replaying statements, no further `replayStatements` or `replayDone` notification must be generated because of this filter after `statement_unstable_remove` has been processed.

If one or more newly inserted statements were waiting only because this filter's replay was still in progress, then once the removal has been processed they no longer need to wait for this filter.

However, due to the asynchronous nature of JSON-RPC client <-> server communication, notifications that were already in the process of being sent back by the JSON-RPC server might still arrive.
