# Minimal L2 Resolver Specification

## Overview

This document specifies a minimal L2 Resolver system for ENS. It focuses on resolving ENS names to their corresponding EIP-7930 formatted chain identifiers (as bytes) and resolving these EIP-7930 chain identifiers back to their primary ENS names. This system is implemented as a single contract that handles storage, authorization, and resolution logic.

The system provides bidirectional resolution between ENS names (e.g., `optimism.l2.eth`) and EIP-7930 formatted chain identifiers, operating on the `l2.eth` 2LD (second-level domain) using ENSIP-10 wildcard resolution. The single `L2Resolver` contract contains storage via a `records` mapping, authorization logic following the ENS ownership model, and resolution functions including `resolveChainId()` for forward resolution and `resolveChainName()` for reverse resolution. Chain identifiers must use EIP-7930 format with `AddressLength = 0` (chain-only, no address component), and reverse lookups utilize a dedicated `REVERSE_LOOKUP_NODE` with keccak256-hashed keys.

It is important to note that this resolver specification is primarily intended for consumption by off-chain clients, such as SDKs and wallet software, rather than for direct on-chain smart contract integrations.

---

## Architecture

The system comprises a single main smart contract:

- **`L2Resolver`**: A contract that contains all data storage mechanisms, authorization logic for write operations, and the specific ENS resolution logic.

### Design Considerations

This single-contract approach offers simplicity in initial deployment as all data and logic are co-located. The storage structure and the initial set of functionalities are fixed upon deployment. This specification focuses on the self-contained L2Resolver contract. If further resolution paths or record types are desired, this contract can be wrapped by a newer one that extends the original interface. 

---

## Core Requirements

The primary functionalities of this L2 resolver system are:

1. **Domain to Chain ID Resolution**: Given an ENS name (e.g., `optimism.l2.eth`), resolve its EIP-7930 formatted chain identifier as `bytes` (representing the chain only, with `AddressLength = 0`) via `resolveChainId`.
2. **Chain ID to Domain Resolution**: Given an EIP-7930 formatted chain identifier as `bytes` (with `AddressLength = 0`), resolve its primary associated ENS name (e.g., `optimism.l2.eth`) via `resolveName`.

The L2 resolver system will operate on the designated ENS 2LD `l2.eth` using ENSIP-10 wildcard resolution. ENSIP-10 compliant clients, when resolving a name like `sub.l2.eth`, will first attempt to find a resolver for `sub.l2.eth`. If none is found, they will try the parent (`l2.eth`). If a resolver (this `L2Resolver` contract) is found for `l2.eth`, the client will then call the `resolve(bytes calldata name, bytes calldata data)` function on this resolver, passing the DNS-encoded original full name (e.g., `dnsencode("sub.l2.eth")`) as the `name` parameter.

---

## Contract Specification: `L2Resolver`

This contract serves as the combined storage, authorization, and resolution layer.

**State Variables:**

- `mapping(bytes32 _node => mapping(string _key => bytes _data)) public records;`
    - Stores arbitrary data for each ENS node.
    - For domain-to-chain-identifier resolution, the `key` will be a pre-defined string (e.g., `CHAIN_IDENTIFIER_EIP7930_KEY`) and `data` will be the **raw EIP-7930 chain identifier `bytes`** (formatted with `AddressLength = 0`).
    - For EIP-7930-chain-identifier-to-domain resolution (reverse lookup), the `REVERSE_LOOKUP_NODE` is used. The `key` is constructed by concatenating a prefix (e.g., `CHAIN_IDENTIFIER_EIP7930_KEY`) with the hexadecimal string representation of the `keccak256` hash of the EIP-7930 chain identifier `bytes`. The `data` stored is the **ABI-encoded human-readable ENS name string** (e.g., `abi.encode("optimism.l2.eth")`).

**Constants:**

- `bytes32 constant REVERSE_LOOKUP_NODE = bytes32(keccak256("reverse.chain.id.eip7930"));`
    - A dedicated node hash for storing reverse lookup entries within this contract's `records` mapping.
- `string constant CHAIN_IDENTIFIER_EIP7930_KEY = "chain.id.eip7930";`
    - The key used in `records` to store the EIP-7930 chain identifier bytes for a forward resolution.

**Key Functions:**

- `function setRecord(bytes32 _node, string calldata _key, bytes calldata _value) external /* authorized */`
    - Sets raw bytes for a given `node` and `key` in the `records` mapping.
    - Authorization logic is implemented directly within this function (see below).
    - Emits: `RecordSet(bytes32 indexed _node, string indexed _key, bytes _value)`
    
- `function getRecord(bytes32 _node, string calldata _key) external view returns (bytes memory)`
    - Retrieves raw bytes for a given `node` and `key` from the `records` mapping. Returns empty `bytes` if the record is not found.
    
- `function resolveChainId(bytes32 _node) external view returns (bytes memory)`
    - Retrieves the EIP-7930 formatted chain identifier (as `bytes`, with `AddressLength = 0`) for a given `node`.
    - Implementation: Calls `this.getRecord(node, CHAIN_IDENTIFIER_EIP7930_KEY)`. Returns the raw `bytes` (which will be empty if the record is not found).
    
- `function resolveChainName(bytes calldata _chainIdBytes) external view returns (string memory)`
    - Retrieves the primary human-readable ENS name string associated with a given EIP-7930 chain identifier (`bytes`, with `AddressLength = 0`).
    - Assumes the input `chainIdBytes` is a correctly formatted EIP-7930 sequence.
    - Off-chain tooling used to set reverse records should ensure canonical name formatting (e.g., lowercase, UTS#46 normalization) of the ENS name string.
    - Implementation:
        1. Hash the input: `bytes32 identifierHash = keccak256(chainIdBytes);`
        2. Convert hash to hex string (conceptual step, e.g., using an internal `_bytes32ToHexString` helper). Let this be `hexIdentifierHash`.
        3. Construct key: `string memory key = string.concat(CHAIN_IDENTIFIER_EIP7930_KEY, hexIdentifierHash);`
        4. Retrieves data: `bytes memory data = this.getRecord(REVERSE_LOOKUP_NODE, key);`
        5. Handles not found: `if (data.length == 0) { return ""; }`
        6. Decodes and returns: `return abi.decode(data, (string));` (This will revert if `data` is not a valid ABI-encoded string).
        

**ENS Standard Functions:**

- `function resolve(bytes calldata _name, bytes calldata _data) external view returns (bytes memory)`
    - Implements the ENSIP-10 Extended Resolver interface. This function is called by ENS clients when this contract serves as a wildcard resolver (e.g., for `.l2.eth`).
    - The `name` parameter is the DNS-encoded full version of the name being resolved (e.g., `dnsencode("optimism.l2.eth")`). This allows the resolver to inspect individual labels if needed for custom resolution logic beyond simple node-based lookups.
    - The `data` parameter contains the ABI-encoded function selector and arguments for the actual data being requested, prepared by the client. For target functions that operate on an ENS node (like `resolveChainId(bytes32 node)`), the client calculates the `node` and includes it within this `data` payload.
    - This function routes calls to other specific view functions on this resolver based on the function selector found in the `data` parameter.
    - Currently supports routing to:
        - `this.resolveChainId(node)`: If `data` contains the selector and ABI-encoded arguments for `resolveChainId(bytes32)`. The `node` argument is decoded by the resolver from the `data` parameter.
        - `this.resolveName(chainIdentifierBytes)`: If `data` contains the selector and ABI-encoded arguments for `resolveName(bytes)`. The `chainIdentifierBytes` argument is decoded by the resolver from the `data` parameter.
    - Returns the ABI-encoded result of the target function, or reverts if the `data` does not correspond to a supported function or if resolution fails.
- `function supportsInterface(bytes4 interfaceID) external pure returns (bool)`
    - Implements ENSIP-165. Returns `true` for:
        - `0x01ffc9a7` (IERC165)
        - `0x9061b923` (ENSIP-10 ExtendedResolver interface)

---

## EIP-7930 Chain Identifier Format

Chain identifiers are stored and retrieved as EIP-7930 binary formatted `bytes`. For the purpose of this resolver (identifying a chain, not an address on a chain), these identifiers **MUST** be formatted with `AddressLength = 0`.

The basic structure as per EIP-7930 v1 is:
`Version (2 bytes) | ChainType (2 bytes) | ChainReferenceLength (1 byte) | ChainReference (variable) | AddressLength (1 byte, value 0x00)`

- **Version**: `0x0001` (for v1).
- **ChainType**: As defined in CAIP-350 (corresponds to a CAIP-2 namespace).
- **ChainReferenceLength**: Length of `ChainReference`.
- **ChainReference**: Binary representation of the chain ID within the `ChainType`'s namespace.
- **AddressLength**: `0x00` (1 byte, value zero), indicating no address part.

Clients (SDKs, wallets) interacting with this resolver are responsible for correctly constructing and parsing these EIP-7930 chain identifier `bytes`. The resolver contract treats this data as opaque byte sequences for storage and retrieval in forward lookups, and uses its hash for keying in reverse lookups.

---

## Authorization Model

Write operations via `setRecord(bytes32 node, string calldata key, bytes calldata value)` must be permissioned. The authorization logic is implemented directly within the `L2Resolver` contract. It should generally follow the ENS ownership model:

- To set a record for a specific `node` (e.g., `namehash("optimism.l2.eth")`), `msg.sender` must be the beneficial owner of that `node` (potentially resolved via `ENS.owner(node)` and checking against `INameWrapper.ownerOf(node)` if applicable) or an operator approved by the beneficial owner.
- For non-registered 3LDs (wildcard scenario where `node` corresponds to something like `namehash("unregistered.l2.eth")`), `msg.sender` must be the beneficial owner of the parent domain (e.g., `l2.eth`) or their approved operator.
- To set a reverse EIP-7930 chain identifier mapping (i.e., when `node == REVERSE_LOOKUP_NODE`), `msg.sender` must be a designated administrative entity (e.g., the owner of `l2.eth` or a specific authorized address). This control for `REVERSE_LOOKUP_NODE` writes is crucial for the integrity of reverse lookups.

The `L2Resolver` will require a reference to the ENS Registry (and potentially INameWrapper) to implement this authorization logic.