# Chain Registry

## Context and Motivation

Historically, the Ethereum ecosystem has relied on off-chain registries, such as [`ethereum-lists/chains`](https://github.com/ethereum-lists/chains) or [`chainlist.org`](https://chainlist.org), to maintain the correspondence between chain names and their identifiers (chainId).

This approach, while practical, has two fundamental limitations:
1.  It requires trust in external repositories or servers.
2.  The information cannot be queried natively by smart contracts.

## Purpose of the Chain Registry

The Chain Registry addresses these limitations through a verifiable and standardized on-chain registry implemented in the `ChainResolver.sol` contract. It serves as the on-chain implementation for the human-readable chain name resolution proposed in **ERC-7828**.

This system:
-   Stores chain data directly on the blockchain.
-   Allows native queries from smart contracts.
-   Establishes a source of truth that is resistant to censorship and spoofing.
-   Is governed by an open model, initially controlled by a multisig and designed to evolve toward community consensus.

## Architecture

The `ChainResolver` contract is inspired by previous ENS resolver implementations, adapting the concept for a global, multichain registry aligned with the ERC-7930 standard.

The structure is based on a system of forward and reverse mappings for efficient name resolution:
-   **Forward mapping**: `labelhash` → `ChainData(chainId, chainName)`
-   **Reverse mapping**: `chainId` → `labelhash`
-   **Ownership mapping**: `labelhash` → `owner`

## Ownership and Permissions

The global owner of the contract (by default, a multisig address) can register new chains via:
`register(chainName, owner, chainId)`

Each registration has its own owner (`labelOwner`), who is responsible for managing the data for that chain. Owners can delegate permissions to operators (`setOperator`) to allow for authorized updates. Verifications are performed with `_authenticateCaller`, which ensures that only the owner or their operators can modify a record.

## Identifier Format: ERC-7930

The Chain Registry stores a standardized binary representation for chain identifiers, designed to be the on-chain counterpart to the text-based **CAIP-2** standard.

To achieve this, it uses a subset of the **ERC-7930** Interoperable Address format. Specifically, it stores an Interoperable Address where the `AddressLength` field is set to zero.

While ERC-7930 defines the *structure* (the "envelope"), **CAIP-350** provides the *content* (the "instructions"). Each CAIP-350 profile specifies how a chain's `namespace` and `reference` (from CAIP-2) are encoded into the binary `ChainType` and `ChainReference` fields.

This modular approach allows the registry to support identifiers of any length, overcoming the 32-character limitation of CAIP-2 and ensuring compatibility with both EVM and non-EVM chains.

## Core Operations

| Function                             | Description                                            |
| ------------------------------------ | ------------------------------------------------------ |
| `register(chainName, owner, chainId)` | Creates a new chain registration (only callable by the global owner). |
| `setRecord(labelHash, chainId, chainName)` | Updates a chain's data (owner or its operator).       |
| `chainId(labelHash)`                 | Returns a chain's identifier (direct lookup).          |
| `chainName(chainId)`                 | Returns the chain's name (reverse lookup).             |
| `setLabelOwner(labelHash, owner)`    | Transfers the ownership of a label.                    |
| `setOperator(operator, isOperator)`  | Manages delegated permissions.                         |

## A Naming Layer for Chains

The Chain Registry functions as a specialized ENS (Ethereum Name Service) for chains. Instead of resolving names to addresses, it resolves chain names to standardized identifiers (ERC-7930).

This offers:
-   Censorship resistance
-   On-chain verifiability
-   Multichain interoperability
-   Compatibility with existing ENS resolvers

## Benefits

-   Smart contracts can check the validity of a chain without relying on external sources.
-   Adopts CAIP-2 and ERC-7930, ensuring compatibility between EVM and non-EVM ecosystems.
-   Starts under a multisig and evolves toward community control.
-   Its design is optimized for simplicity and security.
