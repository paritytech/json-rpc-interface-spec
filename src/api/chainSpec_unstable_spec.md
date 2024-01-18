# chainSpec_unstable_spec

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

    "properties": {
        ...
    },

    // Optional fields:

    "codeSubstitutes": {
        "...": "0x...",
    },

    "telemetryEndpoints": [
        {
            "address": "...",
            "verbosity_level": 0,
        }
    ],

    "networkProperties": {
        "protocolId": "...",
        "forkId": "...",
    },

    "parachain": {
        "id": 0,
        "relayChain": "..."
    },

    "checkpoint": {
        "trustedBlocks": [
            {
                "blockNumber": "...",
                "blockHash": "0x...",
            }
        ],
        "badBlocks": [
            "0x...",
        ],
    }
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

- `bootnodes` is an array of strings containing the [multi adresss format](https://github.com/multiformats/multiaddr) of the bootnodes of the chain.  
For example, the `"/dns/polkadot-bootnode-0.polkadot.io/tcp/30333/p2p/12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU"` is a valid p2p multiaddr, where `12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU` represents the peer ID of the bootnode.
To establish a connection to the chain at least one bootnode is needed.  
The servers are encouraged to provide at least one trusted bootnode.

- `genesis` is a JSON object containing the genesis storage of the chain. This field has the following formats, depending on the provided `rawGenesis` parameter.
  - If `rawGenesis` is `true`, then the `genesis` field is a JSON object containing the raw genesis storage of the chain. The object has the following format:

    ```json
    {
        "raw": {
            "top": {
                "0x..": "0x..",
                ...
            },
        }
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

- `properties` is a JSON object containing a key-value map of properties of the chain.  
  The following are examples of possible properties, although implementations are free to diverge from this list:  
  The `"ss58Format"` field is an unsigned integer indicating the designated SS58 prefix of the addresses of the chain. For more details see [Polkadot Accounts In-Depth](https://wiki.polkadot.network/docs/learn-account-advanced).  
  The `"tokenDecimals` field is an unsigned integer indicating the number of decimals of the native token of the chain.  
  The `"tokenSymbol"` field is a string containing the symbol of the native token of the chain.

- `codeSubstitutes` is an _optional_ JSON object containing a key-value map of code substitutes of the chain. The key is an unsigned integer represented as string indicating the block number at which the code substitute is applied. The value is a hexadecimal-encoded string representing the wasm runtime code, which is normally found under `b:code:` key.  
The given wasm runtime code is used to substitute the runtime code starting from the provided block number and until the spec-version of the chain changes.  
A substitute should be used when runtime upgrades cannot fix an underlying issue. For example, when the runtime upgrade panics.

- `telemetryEndpoints` is an _optional_ array of objects containing the telemetry endpoints of the chain.  
Each object has the following format:

    ```json
    {
        "address": "...",
        "verbosity_level": 0,
    }
    ```

  - `"address"` is a string containing the address of the telemetry server. The address can be specified in the URL format, or in the multi address format.  
  - `"verbosity_level"` is an unsigned integer indicating the verbosity level of the telemetry server. The verbosity ranges from 0 to 9, where 0 is the least verbose and 9 is the most verbose.

- `networkProperties` is an _optional_ JSON object containing the network properties of the chain. The object has the following format:

    ```json
    {
        "protocolId": "...",
        "forkId": "...",
    }
    ```

  - `protocolId` is an _optional_ string containing the network protocol id that identifies the chain.
  - `forkId` is an _optional_ string containing the fork id that identifies the chain. In the case where two chains have the same genesis, this field can be used to signal a fork on the networking level.

- `parachain` is an _optional_ JSON object containing the parachain information. This fields is present only if the client spec describes a parachain. The object has the following format:

    ```json
    {
        "id": 0,
        "relayChain": "..."
    }
    ```

  - `id` is an unsigned integer indicating the id of the parachain.
  - `relayChain` is a string containing the identifier of the relay chain.

- `checkpoint` is an _optional_ JSON object containing the checkpoint of the chain. This information could be used to synchronize the client with the head of the chain.  
The `"trustedBlocks"` field is an array of JSON objects representing the expected hashes of blocks at given heights. This can be used to set trusted checkpoints. The client should refuse to import blocks with a different hash at the given height.

    ```json
        "trustedBlocks": [
            {
                "blockNumber": "...",
                "blockHash": "0x...",
            }
        ]
    ```

  - `"blockNumber"` is an unsigned integer represented as string indicating the block number.
  - `"blockHash"` is a hexadecimal-encoded string representing the hash of the block at the given height.

  The `"badBlocks"` field is an array of hexadecimal-encoded strings representing the hashes of blocks that should be considered invalid. These hashes represent known and unwanted blocks on forks that have been pruned.
