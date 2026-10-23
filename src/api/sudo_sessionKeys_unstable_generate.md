# sudo_sessionKeys_unstable_generate

**Parameters**:

- `owner`: SCALE encoded account id of the account that will register the session keys on chain. This is used by the runtime to create a proof. This proof is send alongside the generated keys to the chain. On chain
           this will be used to ensure that `owner` actually is in posession of the private keys. 

**Return value**: 

- If the runtime supports the function call (see below), an object of the form `{ "success": true, "version", ..., "value": ... }` where `value` contains a string containing the hexadecimal-encoded output of the runtime
  function call and version is the version of the `SessionKeys` API on chain that was used to generate `value`.
- Otherwise, an object of the form `{ "success": false, "error": ... }` where `error` is a human-readable error message indicating the problem. This string isn't meant to be shown to end users, but is for developers to
  understand the problem.

The JSON-RPC server must check that the runtime supports the `SessionKeys` API (64bits blake2 hash: `0xab3c0572291feb8b`) at version 1 or version 2, and call the `SessionKeys_generate_session_keys` runtime function.
The runtime call is done against the current best block of the chain.

If there is no `SessionKeys` API being supported, or if it is not at version 1, the JSON-RPC server is allowed to call an alternative version of this function if it is sensible to do so. For example, if the `SessionKeys`
API is updated to version 2 without a substantial change in the logic of `SessionKeys_generate_session_keys`, then the JSON-RPC server is allowed to call it as well. This specification should be updated if that happens.

Contrary to most other JSON-RPC functions that perform runtime function calls where side-effects are forbidden, this runtime must be allowed to call host functions that access the keys of the node (e.g. `ext_crypto_sr25519_generate_version_1`, `ext_crypto_ed25519_public_keys_version_1`, etc.).

**Note**: This can be used as a replacement for the legacy `author_rotateKeys` function.

## Possible errors

- `{ "success": false, "error": ... }` is returned if the runtime doesn't support the given API.
- `{ "success": false, "error": ... }` is returned if a problem happens during the call, such as a Wasm trap.
- `{ "success": false, "error": ... }` is returned if the runtime attempts to modify the storage of the block.

## About the behavior of `SessionKeys_generate_session_keys`

The objective of this JSON-RPC function is to call the `SessionKeys_generate_session_keys` runtime function. This paragraph describes how this runtime function behaves.

The `SessionKeys_generate_session_keys` runtime function generates a serie of keys, inserts these keys in the so-called keystore, and returns all the public keys concatenated together.

Because the newly-generated keys are inserted in the keystore of the JSON-RPC server, it will automatically start performing duties such as authoring blocks or emitting Grandpa votes if one of the generated public keys corresponds to a key that is given the rights by the blockchain to do so. Most of the time, the keystore is configured to write the keys on disk, meaning that these newly-generated keys remain in the keystore even after the JSON-RPC server has been restarted.

The value returned by this function, which is the concatenation of all newly-generated public keys, is called the session keys. The session keys are meant to be submitted to the blockchain via a transaction by the JSON-RPC client or its user. Before this is done, the newly-generated keys normally don't automatically obtain the right to, for example, generate blocks. Submitting the session keys to the blockchain is out of scope of this function.
