# chainSpec_unstable_clientSpec

**Parameters**:

- `rawGenesis`: A boolean indicating whether the genesis block should be returned in raw format or not.

**Return value**: A JSON object.

The JSON object returned by this function has the following format:

```json
{
    "name": "...",

    "id": "...",

    "chainType": "Live" | "Local" | "Development" | { "Custom": "..." },

    "bootnodes": [
        "...",
    ],

    "genesis": {
        "stateRootHash": "0x...",
    },

    "telemetryEndpoints": [
        {
            "address": "...",
            "verbosity_level": 0,
        }
    ],
}
```

- `name` is a string containing the name of the chain, identical to the result of `chainSpec_v1_chainName`.
- `id` is a string containing the identifier of the chain.
- `chainType` is an object containing the type of the chain.  
This information can be used by tools to display additional information to the user.  
This fields can be one of the following values:
  - `"Live"`: A string indicating the chain is a live chain.
  - `"Local"`: A string indicating the chain is a local chain that runs on multiple nodes for testing purposes.
  - `"Development"`: A string indicating the chain is a development chain that runs mainly on one node.
  - `{ "Custom": "..." }`: A JSON object indicating the chain is a custom chain. The value of the `"Custom"` field is a string.
- `bootnodes` is an array of strings containing the [p2p multiaddr](https://github.com/libp2p/specs/blob/master/addressing/README.md#the-p2p-multiaddr) of the bootnodes of the chain.  
For example, the `"/dns/polkadot-bootnode-0.polkadot.io/tcp/30333/p2p/12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU"` is a valid p2p multiaddr, where `12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU` represents the peer ID of the bootnode.
To establish a connection to the chain at least one bootnode is needed.  
The servers are encouraged to provide at least one trusted bootnode.
- `genesis` is an object containing the genesis storage of the chain. This field has the following formats, depending on the provided `rawGenesis` parameter.
  - If `rawGenesis` is `true`, then the `genesis` field is a JSON object containing the raw genesis storage of the chain. The object has the following format:

    ```json
    {
        "raw": {
            "top": {
                "0x..": "0x..",
                ...
            },
        }
        ...
    }
    ```

    The `"top"` field contains a map of keys to values for the top-level storage entries of the genesis.  
    Both the keys and the values are hexadecimal-encoded strings.  
    The provided keys are guaranteed to be unique and to not be a default storage child key. A default storage child key is a key that is prefixed with `b":child_storage:default:"`.  
    For example, the wasm code of the runtime is stored under the key `b":code"`.

  - If `rawGenesis` is `false`, then the `genesis` field is a JSON object containing the Merkle value of the genesis block.
  The object has the following format:

    ```json
    {
        "stateRootHash": "0x...",
    }
    ```

    The `"stateRootHash"` contains a hexadecimal-encoded string representing the Merkle value of the genesis block.

- `telemetryEndpoints` is an array of objects containing the telemetry endpoints of the chain.  
Each object has the following format:

    ```json
    {
        "address": "...",
        "verbosity_level": 0,
    }
    ```

  The `"address"` is a string containing the address of the telemetry server. The address can be specified in the URL format, or in the multiaddr format.  
  The `"verbosity_level"` is an unsigned integer indicating the verbosity level of the telemetry server.

