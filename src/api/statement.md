# Introduction

The `statement` functions expose the statement store of the node associated to the JSON-RPC server.

Statement stores are weakly coherent. Different JSON-RPC servers can know different subsets of statements at any moment, and no guarantee is offered as to the time it takes for a statement to become visible everywhere.

The `statement_unstable_subscribe` function creates the main source of notifications. Filters are later attached with successful calls to `statement_unstable_add_filter`, and all notifications for all filters attached to the same subscription are generated on that one subscription. If the server can't accept a filter on a subscription, `statement_unstable_add_filter` returns `{"result": "limitReached"}`. The `statement_unstable_submit` function is independent from subscriptions and can be called at any time.

## Concepts

A _statement_ is an opaque SCALE-encoded value from the point of view of a JSON-RPC client. The methods in this group don't require understanding its internal layout in order to submit it, store it, or receive it through notifications.

A _topic_ is a 32-byte tag attached to a statement. Topics are used for filtering: `"any"` matches all statements, while `matchAll` matches statements that contain all the requested topics.

A _channel_ is a logical slot scoped to the author of a statement. The store keeps at most one current statement per `(account, channel)` pair.

An _expiry_ is the ordering value used by the store when statements compete for retention. If multiple statements exist on the same channel, the one with the highest expiry wins. More generally, statements with lower expiry are more likely to be discarded first when the store needs to make room.

An _allowance_ is the budget that a given account has in the store. It limits how many statements and how much total data that account can occupy at the same time.
