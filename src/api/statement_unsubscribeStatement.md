# statement_unsubscribeStatement

**Parameters**:

- `subscription`: Opaque string equal to the value returned by `statement_subscribeStatement`.

**Return value**: *null*

## Possible errors

A JSON-RPC error is generated if the `subscription` doesn't correspond to any active subscription.
