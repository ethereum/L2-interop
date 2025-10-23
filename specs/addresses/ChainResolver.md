# Chain Resolver Specification

## Overview

This document specifies a unified Chain Resolver system for ENS, based on the `ChainResolver.sol` contract. It provides a single, on-chain source of truth for mapping human-readable chain labels (e.g., "optimism") to EIP-7930 chain identifiers and vice-versa.

### Chain Registry
Historically, the Ethereum ecosystem has relied on off-chain registries like the `ethereum-lists/chains` repository and services such as Chainlist.org to map chain names to their identifiers. This approach has two main limitations: it does not provide a trust-minimized source of truth, and the information cannot be queried natively by smart contracts.

This specification introduces an on-chain registry implemented in `ChainResolver.sol` to address these limitations. By storing chain data directly on-chain, it establishes a natively queryable source of truth that enhances censorship resistance and protects against spoofing.

Previously, proposals like ERC-7785 explored similar solutions. However, because ERC-7785 was deprecated, the current system is not compatible with that proposal and uses a different architecture centered on the ENSIP-10 resolver interface.

The system is designed to be agnostic to the top-level ENS namespace (e.g., `on.eth`, `l2.eth`) by keying all data to a `labelhash` (the `keccak256` hash of the chain's label). Read operations are exposed through the standard ENSIP-10: Wildcard Resolution interface (`resolve()`), making it compatible with existing ENS client libraries. Write operations for registering chains are restricted to the contract owner, while updates to a chain's records are restricted to the designated owner of that chain's label.

## Architecture

The system is implemented as a single, self-contained smart contract:

-   **`ChainResolver.sol`**: A contract that consolidates the logic for a chain registry, data storage, authorization, and ENS resolution.

### Design Considerations

-   **Namespace Agnostic**: Since all records are keyed by `labelhash` instead of the full ENS `node`, the top-level domain can be managed and changed externally without requiring data migration.
-   **Single Source of Truth**: Ownership, EIP-7930 chain identifiers, standard ENS records (`addr`, `text`, etc.), and reverse lookups are all managed within one contract, simplifying data management and retrieval.
-   **Standard Compliance**: It fully implements the `IExtendedResolver` interface, allowing any standard ENS client to resolve chain metadata.

---

## Contract Specification: `ChainResolver.sol`

### State Variables

-   **Chain Data Storage**:
    -   `mapping(bytes32 => bytes) internal chainIds`: Maps a `labelhash` to its EIP-7930 chain identifier (`bytes`).
    -   `mapping(bytes => string) internal chainNames`: Maps an EIP-7930 chain identifier (`bytes`) back to its human-readable chain name (`string`) for reverse resolution.
    -   `mapping(bytes32 => address) internal labelOwners`: Maps a `labelhash` to its designated owner address.
    -   `mapping(address => mapping(address => bool)) internal operators`: Per-owner mapping to approve operator addresses.
-   **ENS Record Storage**:
    -   `mapping(bytes32 => mapping(uint256 => bytes)) private addressRecords`: Stores multi-coin addresses per `labelhash` and `coinType`.
    -   `mapping(bytes32 => bytes) private contenthashRecords`: Stores the `contenthash` for each `labelhash`.
    -   `mapping(bytes32 => mapping(string => string)) private textRecords`: Stores text records for each `labelhash`.
    -   `mapping(bytes32 => mapping(string => bytes)) private dataRecords`: Stores raw data records for each `labelhash`.

### Events
- `event RecordSet(bytes32 indexed _labelhash, bytes _chainId, string _chainName)`: Emitted when a new chain is registered.
- `event LabelOwnerSet(bytes32 indexed _labelhash, address _owner)`: Emitted when the owner of a label changes.
- `event OperatorSet(address indexed _owner, address indexed _operator, bool _isOperator)`: Emitted when an operator is approved or revoked for an owner.
- `event AddrChanged(bytes32 indexed _labelhash, address newAddress)`: Emitted when the primary ETH address (coin type 60) for a label is set.
- `event AddressChanged(bytes32 indexed node, uint256 coinType, bytes newAddress)`: Emitted when a multi-coin address is set for a label.
- `event DataChanged(bytes32 node, string indexed indexedKey, string key, bytes data)`: Emitted when a data record is set.

### Custom Errors
- `error InvalidDataLength()`: Thrown by `batchRegister` if the input arrays have mismatched lengths.
- `error NotAuthorized(address _caller, bytes32 _labelhash)`: Thrown if an address that is not a label owner or authorized operator attempts a restricted action.

### Functions

#### Registry Administration (Contract Owner-Only)
-   `function register(string calldata _chainName, address _owner, bytes calldata _chainId) external onlyOwner`:
    -   Registers a new chain. Computes `labelhash` from `_chainName`.
    -   Stores the `_chainId`, maps it back to `_chainName`, and assigns the label `_owner`.
-   `function batchRegister(string[] calldata _chainNames, address[] calldata _owners, bytes[] calldata _chainIds) external onlyOwner`:
    -   Allows the contract owner to register multiple chains in a single transaction. Reverts if input array lengths do not match.

#### Label & Operator Management
-   `function setLabelOwner(bytes32 _labelhash, address _owner) external onlyAuthorized(_labelhash)`:
    -   Transfers ownership of a specific chain label. Requires the caller to be the current label owner or an approved operator.
-   `function setOperator(address _operator, bool _isOperator) external`:
    -   Allows a label owner (`msg.sender`) to grant or revoke operator permissions for their own account. An operator can then act on behalf of the owner for any labels they own.

#### Public Read Functions
-   `function chainId(bytes32 _labelhash) external view returns (bytes memory)`:
    -   Performs a forward lookup, returning the EIP-7930 chain identifier for a given `labelhash`.
-   `function chainName(bytes calldata _chainIdBytes) external view returns (string memory)`:
    -   Performs a reverse lookup, returning the chain name for a given EIP-7930 chain identifier.
-   `function isAuthorized(bytes32 _labelhash, address _address) external view returns (bool)`:
    -   Checks if an address is either the owner of a label or an authorized operator for that owner.

#### ENS Record Management (Label Owner or Operator)
The following functions are restricted by the `onlyAuthorized(_labelhash)` modifier:
-   `function setAddr(bytes32 _labelhash, address _addr)`: Sets the ETH address (coin type 60).
-   `function setAddr(bytes32 _labelhash, uint256 _coinType, bytes calldata _value)`: Sets a multi-coin address for any coin type.
-   `function setContenthash(bytes32 _labelhash, bytes calldata _hash)`
-   `function setText(bytes32 _labelhash, string calldata _key, string calldata _value)`
-   `function setData(bytes32 _labelhash, string calldata _key, bytes calldata _data)`

### ENS Resolution (`resolve` entrypoint)

The contract implements `resolve(bytes calldata name, bytes calldata data)` per ENSIP-10. It extracts the `labelhash` from the leftmost label of the DNS-encoded `name` and routes the call based on the function selector embedded in `data`. The resolver then uses the `labelhash` to retrieve the correct records.

This implementation supports all standard ENS record types:
- **Addresses**: Per ENSIP-1
- **Multichain Addresses**: Per ENSIP-9
- **Contenthash**: Per ENSIP-7
- **Text Records**: Per ENSIP-5

The resolution of chain identifiers is enabled by two new proposed standards: ENSIP-TBD-18, which introduces the global `"chain-id"` text record, and ENSIP-TBD-19, which defines the `data()` interface for resolving arbitrary bytes.

#### Forward Resolution (`label` → `chainId`)
Clients retrieve a chain's EIP-7930 identifier via standard ENS records. The `node` parameter in the calldata should be the `labelhash` of the chain name.

1.  **As raw bytes (Recommended)**:
    *   Call `resolve(name, data)`, where `name` is the DNS-encoded name (e.g., `optimism.cid.eth`) and `data` is the ABI-encoded calldata for `data(labelhash, "chain-id")`.
    *   The resolver intercepts this key and returns the raw `bytes` of the identifier from its `chainIds` mapping.
2.  **As a hex string (Compatibility)**:
    *   Call `resolve(name, data)` with calldata for `text(labelhash, "chain-id")`.
    *   The resolver intercepts this key, retrieves the raw bytes, and returns them as a hex-encoded string.

#### Reverse Resolution (`chainId` → `name`)
Reverse lookups use the `text` record interface with a specially formatted key. For reverse lookups, the resolver ignores the `name` parameter passed to `resolve()` and the `node` parameter within the `text()` calldata.

1.  The client constructs a text record key with the format `chain-name:<7930-hex-string>`.
2.  The client calls `resolve(name, data)`.
    - `name` can be any valid DNS-encoded string (e.g., `reverse.cid.eth`), as it is ignored.
    - `data` is the ABI-encoded calldata for `text(node, key)`, where `node` can be any `bytes32` value (e.g., `0x0`) and `key` is the string from step 1.
3.  The resolver detects the `chain-name:` prefix, extracts the hex identifier, converts it to bytes, and looks up the name in its `chainNames` mapping. This reverse resolution mechanism is based on the service key parameter format proposed in ENSIP-TBD-17.

---

## EIP-7930 Chain Identifier Format

The resolver stores a standardized binary representation for chain identifiers, designed to be the on-chain counterpart to the text-based **CAIP-2** standard. To achieve this, it uses a subset of the **ERC-7930** Interoperable Address format where the `AddressLength` field is set to zero.

While ERC-7930 defines the *structure* (the "envelope"), a related standard, **CAIP-350**, provides the *content* rules. Each CAIP-350 profile specifies how a chain's `namespace` and `reference` from its CAIP-2 identifier are encoded into the binary `ChainType` and `ChainReference` fields.

### Structure

The binary format is composed of the following fields:
```
┌─────────┬───────────┬──────────────────────┬────────────────┬───────────────┐
│ Version │ ChainType │ ChainReferenceLength │ ChainReference │ AddressLength │
└─────────┴───────────┴──────────────────────┴────────────────┴───────────────┘
```
- **Version** (2 bytes): `0x0001` for this version of the standard.
- **ChainType** (2 bytes): The CAIP-2 namespace code (defined by CAIP-350). For `eip155`, this is `0x0000`.
- **ChainReferenceLength** (1 byte): The length of the `ChainReference` in bytes.
- **ChainReference** (variable): The chain's identifier within its namespace (e.g., the chain ID for `eip155`).
- **AddressLength** (1 byte): Always `0x00`, as this is a chain-only identifier.

### Example: Optimism (Chain ID 10)
The identifier for Optimism is `0x00010000010a00`. Here’s the breakdown:

| Field                  | Value      | Bytes | Description                                |
| ---------------------- | ---------- | ----- | ------------------------------------------ |
| `Version`              | `1`        | 2     | `0x0001`                                   |
| `ChainType`            | `eip155`   | 2     | `0x0000` (Namespace for EVM chains)        |
| `ChainReferenceLength` | `1`        | 1     | `0x01` (The chain ID is 1 byte long)       |
| `ChainReference`       | `10`       | 1     | `0x0a` (The chain ID for Optimism)         |
| `AddressLength`        | `0`        | 1     | `0x00` (No address component)              |

Another example is **Arbitrum (chain 42161)**, which has a 2-byte `ChainReference` (`0xa4b1`) and thus a `ChainReferenceLength` of `0x02`. Its full identifier is `0x0001000002a4b100`.

---

## Authorization Model

-   The `owner` of `ChainResolver.sol` is the sole address that can register new chains via `register` and `batchRegister`. This role is intended for a DAO or a multisig, with the possibility of changing ownership subject to community consensus.
-   For each registered chain, an `owner` is assigned. This owner, or an operator they approve via `setOperator`, has exclusive rights to manage the label's ENS records and transfer its ownership.
