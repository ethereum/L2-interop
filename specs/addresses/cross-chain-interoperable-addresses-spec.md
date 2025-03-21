# Cross-chain interoperable addresses specification
This document, in its current state, aims to be the starting point for a future address format called 'Interoperable Address' whose aims & properties are fully described in [a separate document](../PROPERTIES.md)

Requires: RFC-4648, ENSIP-11, CAIP-2

# Abstract
An extensible way to describe an address specific to one chain which also includes the information to securely condense it into a human-readable name.

# Motivation
The Ethereum ecosystem is rapidly expanding into a multi-chain environment encompassing L1s, L2s, sidechains, and rollups—some EVM‐compatible, others not. A simple address alone no longer identifies which chain the address belongs to, creating ambiguity for wallets, dApps, and users. At the same time, human‐readable naming (e.g. ENS) is important for usability, but must be deterministically tied to the correct network.

Interoperable Addresses build on insights from previous iterations of ERC-7828 and related discussions, offering a unified format which combines:
- Binding chain specificity (via explicit chain identifiers) to the raw address,
- Support for human‐readable naming (through optional resolvers, e.g., ENS), and
- Checksums for name collision mitigation.

All without adding new trust assumptions beyond what wallets already apply. By bundling chain references, addresses, and optional resolver info into a canonical binary format, wallets can deterministically parse and display addresses, while users benefit from intuitive names.
In short, this standard aims to:
- Erase ambiguity by enshrining a deterministic link between chain and address.
- Remain resolution‐agnostic, supporting various naming services and chain registries.
- Permit multiple “version” specifications, ensuring future extensibility for different resolution schemes.
- Provide a stable, machine‐readable binary representation digestible by smart contracts (e.g., via ABI).

These goals make Interoperable Addresses future-proof and user-friendly, enabling trustless, cross-chain workflows in a multi-chain world. The format also seeks to align with existing standards (e.g., CAIP-2, CAIP-10) and embrace diversity of identifier formats.

# Out of scope concerns
Similarly to CAIP-10, this specification is not concerned with the mapping from a chain ID to a network name, which might not be surjective (eg: the case where if there are multiple EIP-155 chains with chain ID 8453, which one should we call Base?), regarding that resolution a social-layer problem until a future ERC decides to tackle it. Efforts in that front are tracked in [the chain registries document](../docs/chain-registries.md)

# Definitions
Interoperable Address
: A binary payload which unambiguously identifies a target address and, optionally, a _resolver specification_ which, along with this (and potentially future) ERCs, allows conversion to a human-readable name without requiring further trust assumptions by wallet software.

Human-readable name
: A string representation of an interoperable address, meant to be used by humans.

Target address
: The (address, chain ID) pair a particular Interoperable Address points to.

Name-resolving contract
: A contract responsible for converting addresses and/or chain IDs from a machine-readable format into parts of a human-readable name.

# Guarantees provided by the standard
- Any Interoperable Address is trivially convertible to Interoperable Canonical Format, `0x8000`, making it a canonical representation for users who need to treat them opaquely.
- Checksums are short enough to be compared by humans, and effectively identify a _target address_ independently of any extra data the address may contain.

# Binary Format definition
Word 0 Is the only word which will be common to all versions, and is defined as follows:
```
MSB                                                            LSB
0xXXXXYYYYYYYYZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                \------------- 207-0   Reserved
```

No length restriction is placed on how many more words can an Interoperable Address span. Different versions of the standard can define uses for extra words and the 208 reserved bits.

# Human-readable name format definition

## Syntax
```bnf
<human readable name> ::= <account>@<chain>#<checksum>
<account>             ::= [-:_%a-zA-Z0-9]*
<chain>               ::= [-:_a-zA-Z0-9]*
<checksum>            ::= [0-9A-F]{8}
```

## Rationale
- Chain and account fields' syntax is deliberately chosen to be able to express CAIP-2 namespaces and CAIP-10 account IDs, with the caveat that no length restriction is placed, so chains with longer address formats or full 256-bit EVM chainids can be represented.
- Similarly, the account field includes `%` as a valid character to allow for url-encoding of any other characters.

# Interoperable Address version definitions

## `0x8000`: Interoperable Canonical Format: arbitrary length addresses, CAIP-10 compatible display format

### Machine-address format

#### First word
No reserved bits are used. They MUST be set to zero, to ensure canonicity of addresses.
```
MSB                                                            LSB
0x8000XXXXXXXX0000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              \--------------- 207-0   Reserved
```

#### Second field
Serializes the chain ID as described in [Appendix A](#appendix-a-binary-encoding-of-caip-2-blockchain-id) and encodes it into a byte array as described in [Appendix C](#appendix-c-short-encoding-of-byte-arrays). It is a _field_ and not a _word_, since it may take up more than one word.

#### Third field
Serializes the address as described in [Appendix A](#appendix-a-binary-encoding-of-caip-2-blockchain-id) and encodes it into a byte array as described in [Appendix C](#appendix-c-short-encoding-of-byte-arrays). It is a _field_ and not a _word_, since it may take up more than one word, and may not start at the third word boundary if the second field takes up more than one word.

### Human-readable name resolution
The CAIP-2 namespace is to be rendered alongside the CAIP-10 formatted address and the checksum:

```
<CAIP-10 formatted address>@<CAIP-2 namespace>#<checksum>
```

### Examples

Human-readable name:
`0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#6164DA10@eip155:1#618ad0d1`:

First word:
```
MSB                                                            LSB
0x80006164DA100000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              \--------------- 207-0   Not used
```

Second word (second field):
```
MSB                                                            LSB
0x0100000000000000000000000000000000000000000000000000000000000002
  ^^----------------------------------------------------------------- 255-249 bytes array payload
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ---- 9-249   padding
                                                                ^^ -- 0-8    2*1 (payload length)
```

Third word (third field):
```
MSB                                                            LSB
0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045000000000000000000000028
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ------------------------- 255-97 bytes array payload
                                          ^^^^^^^^^^^^^^^^^^^^^^ ---- 9-96   padding
                                                                ^^ -- 0-8    2*20 (payload length)
```

TODO: example with multi-word fields

## `0xC000`: Interoperable Canonical Format With Resolver Info: arbitrary length addresses, arbitrary length name-resolving contract specification

### Machine-address format

#### First word
Uses reserved bits as a registry of interfaces the provided resolver conforms to.
```
MSB                                                            LSB
0xC000XXXXXXXX0000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              \--------------- 207-0   Resolver Interface Key
```
The Resolver Interface Key defines how wallets should interact with the smart contract responsible for assigning names to addresses.
Said contract MAY also be responsible for naming the chain on which the target address resides as well as providing a human-readable name for the address, when ERC-7785 or similar standards reach production.

#### Second field
Serializes the chain ID of the target address  as described in [Appendix A](#appendix-a-binary-encoding-of-caip-2-blockchain-id) and encodes it into a byte array as described in [Appendix C](#appendix-c-short-encoding-of-byte-arrays).

#### Third field
Serializes the address of the target address as described in [Appendix A](#appendix-a-binary-encoding-of-caip-2-blockchain-id) and encodes it into a byte array as described in [Appendix C](#appendix-c-short-encoding-of-byte-arrays).

#### Fourth field
Serializes the chain ID where the name-resolving contract is located as described in [Appendix A](#appendix-a-binary-encoding-of-caip-2-blockchain-id) and encodes it into a byte array as described in [Appendix C](#appendix-c-short-encoding-of-byte-arrays).

#### Fifth field
Serializes the address of the name-resolving contract as described in [Appendix A](#appendix-a-binary-encoding-of-caip-2-blockchain-id) and encodes it into a byte array as described in [Appendix C](#appendix-c-short-encoding-of-byte-arrays).

### Human-readable name resolution
Specific to every Resolver Interface Key

### Resolver Interface Keys

#### `0x0000000000000000000000000000000000000000000000000000`
Responsibility over discovering the interface of the naming smart contract is delegated to the contract itself, via mechanisms comparable to `ERC165`, out of scope for this definition.

#### `0x0000000000000000000000000000000000000000000000000001`
Naming contract is expected to conform to ENSIP-11, and used to display the address. ENSIP-11 does not name chains, so the CAIP-2 name should be used instead.

### Examples

Human-readable name:
`vitalik.eth@eip155:1#618AD0D1`:

First word: metadata
```
MSB                                                            LSB
0xC0006164DA100000000000000000000000000000000000000000000000000001
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              \--------------- 207-0   Resolver Interface Key
```

Second word (second field): target address' chain ID
```
MSB                                                            LSB
0x000001000000000000000000000000000000000000000000000000000000000C
  ^^^^^^------------------------------------------------------------- 255-208 bytes array payload
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ---- 9-207   padding
                                                                ^^ -- 0-8     2*6 (payload length)
```

Third word (third field): target address
```
MSB                                                            LSB
0xD8DA6BF26964AF9D7EED9E03E53415D37AA96045000000000000000000000028
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ------------------------- 255-97 bytes array payload
                                          ^^^^^^^^^^^^^^^^^^^^^^ ---- 9-96   padding
                                                                ^^ -- 0-8    2*20 (payload length)
```

Fourth word (fourth field): name resolving contract's chain ID
```
MSB                                                            LSB
0x000001000000000000000000000000000000000000000000000000000000000C
  ^^^^^^------------------------------------------------------------- 255-208 bytes array payload
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ---- 9-207   padding
                                                                ^^ -- 0-8     2*6 (payload length)
```

Fifth word (fifth field): name resolving contract's address
```
MSB                                                            LSB
0x00000000000C2E074EC69A0DFB2997BA6C7D2E1E000000000000000000000028
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ------------------------- 255-97 bytes array payload
                                          ^^^^^^^^^^^^^^^^^^^^^^ ---- 9-96   padding
                                                                ^^ -- 0-8    2*20 (payload length)
```

TODO: example with other ENSIP-11 contract

### Security considerations
- Wallets will have to maintain a list of Resolver Interface Keys they support, and which name-resolving contracts they consider trustworthy, and display addresses as described in `0x8000` when the provided name-resolving contract is not contained in the list. Said list MAY be updateable by the user.

## `0x0000` : single-word 48-bit EVM chain IDs, no human-readable name resolution
This encoding is meant to be the shortest possible way to encode most EVM addresses

### Supported chains
EVM chains with a chain ID shorter than 48 bits (which potentially rules out chain IDs chosen to comply with ERC-7785)

### Machine-address format

```
MSB                                                            LSB
0x0000000000000000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^
                \------------- 207-160 Chain ID
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   Address
```

`Interoperable Address version`
: Always `0x0000`

`Checksum`
: Computed as described in [Appendix D](#appendix-d-checksum-computation)

`ChainID`
: `uint48` containing the chain ID for the target chain

`Address`
: Native EVM address

### Human-readable name resolution
The CAIP-2 namespace is to be rendered alongside the `0x`-prefixed EIP-55 address and the checksum

```
<address>@<CAIP-2 namespace>#<checksum>
```

### Examples:
Machine address
: 
```
MSB                                                            LSB
0x0000618ad0d1000000000001D8DA6BF26964AF9D7EED9E03E53415D37AA96045
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^
                \------------- 207-160 chain ID
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   Address
```

Human-readable name
: `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#6164da10@eip155:1#618AD0D1`

## `0x0001` : Compact format, ENSIP-11+ERC-3770 name resolution 
The binary representation is identical to the format above, with the exception that the version number is different, indicating the intent for the chain to be displayed using the ERC-3770 shortname and the address to be displayed following mainnet's ENS deployment and ENSIP-11

### Supported chains
Chain must be listed on ethereum-lists/chains on top of the restrictions of `0x0000`.

### Machine-address format
Same as `0x0000`, with `0x0001` version

### Examples:
Machine address
: 
```
MSB                                                            LSB
0x0001618ad0d1000000000001D8DA6BF26964AF9D7EED9E03E53415D37AA96045
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^
                \------------- 207-160 chain ID
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   Address
```

Human-readable name
: `vitalik.eth@eth#618AD0D1`

TODO: example on another chain

# Restrictions for future versions
- The Interoperable Address MUST include an address and specify the chain it belongs to, fully defining a target address. Any version that adds an indirection layer between payload data and these two fields would violate this restriction.
- Interoperable Address versions MAY only be able to represent a subset of the CAIP-2 namespaces.
- Interoperable Address versions MAY define extra fields containing information on how to display the addresses.
- The most significant version bit is reserved to mean an address complies with the Interoperable Canonical Format, described in Interoperable Address version `0x8000`.
- The second-most significant version bit is reserved to mean the payload includes information on how to display the address to users, but is not normative about how that information is to be encoded. An example of this is provided in Interoperable Address version `0xC000`.

# Requirements for wallet software
- Wallet software MUST perform all relevant pre-validations, including verifying the checksum, and report any errors to the user.
- Wallets MAY reject an interoperable address solely on the basis of the checks above failing.

# Encoding considerations
- On-chain actors, such as smart contracts, MUST always receive and return the machine addresses as byte array.
- On-chain actors, such as smart contracts, MAY only accept canonical `0x8000` address versions, with off-chain actors being responsible over converting them to said format.
- Off-chain actors, such as wallets, MAY use the blockchain-native byte array representation, or, where practical MAY alternatively use base64 [^2] as defined in RFC-4648 to encode the former.
- Users SHOULD NOT be shown the base64 or binary representations. Instead, they should be presented with the intended human-readable name or, in cases where that is not possible, wallet software MUST fall back to interpreting it as an Interoperable Address version `0x8000`.

[^2]: Base64 was chosen since it is a more widely implemented standard than base58. Additionally, since machine addresses are not to be directly displayed to the user, the collision-avoidance reasons in favor of base58 do not apply

# Appendix A: Binary encoding of CAIP-2 blockchain ID
The first two bytes are the binary representation of CAIP-2 namespace (see table below), while the remaining bytes correspond to the CAIP-2 reference, whose encoding is namespace-specific and defined below.

## CAIP-2 namespaces' binary representation table

| Namespace | binary key |
| ---       | ---        |
| eip155    | `0x0000`   |
| bip122    | `0x0001`   |

### eip155
The bare `chainid` encoded as a `uint` of the necessary amount of bytes will be used [^1]. This allows for most of existing EVM chain IDs to fit inside one EVM word.

[^1]: This makes it possible to represent some chains using the full word as their chainid, which CAIP-2 does not support since the set of values representable with 32 `a-zA-Z0-9` characters has less than `type(uint256).max` elements. This is done in an effort to support chains whose ID is the output of `keccak256`, as proposed in ERC-7785.

#### Examples
Ethereum Mainnet: `0x000001` (1, encoded as uint8)

Optimism Mainnet: `0x00000A` (10, encoded as uint8)

Ethereum Sepolia: `0x0000aa36a7` (11155111, encoded as uint24)

### bip122
The genesis/fork blockhash is to be stored raw, without encoding/decoding from/to base58btc, and without removing any leading zeroes:

Note: CAIP-2 limits chain references to 32 characters, so converting to it will require truncating the reference, so converting _from_ CAIP-2 to this standard is potentially ambiguous (but converting from actual bip122 to this standard will never be).

#### Examples
Bitcoin Mainnet
: `0x0001000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f`

Litecoin
: `0x000112a765e31ffd4059bada1e25190f6e98c99d9714d334efa41a195a7e7e04bfe2`

# Appendix B: Binary encoding of addresses

## eip155
20 bytes of evm addresses trivially stored as the payload.

### Examples
`0xD8DA6BF26964AF9D7EED9E03E53415D37AA96045` -> `0xD8DA6BF26964AF9D7EED9E03E53415D37AA96045`

## solana
base-58 encoded public keys should be decoded and stored as a 32 byte payload

### Examples

`7S3P4HxJpyyigGzodYwHtCxZyUQe9JiBMHyRWXArAaKv` -> `0x5F90554BB3D8C2FC82B6EE59C49AAA143E77F7D49A83E956CE1DBEF17A43F805`

`DYw8jCTfwHNRJhhmFcbXvVDTqWMEVFBX6ZKUmG5CNSKK` -> `0xBA7A74F374AB05B70D114A78112EF0D3F0695A819572C79710B5372000D81AE2`

# Appendix C: Short-encoding of byte arrays
This is a mix between [how bytes and strings are saved into storage](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#bytes-and-string) and how ABI encoding works

This arises out of:
- The need for the shortest possible representation of EVM addresses.
- The absence of a canonical place to delimit the _start_ of the data (in the case of storage strings, this is the keccak256 hash of the storage slot index).

If the data is at most 31 bytes long, the elements are stored in the higher-order bytes (left-aligned) and the lowest-order byte stores the value `length * 2`.

If the data is 32 bytes or longer, the current word stores `length * 2 + 1`, and data follows in the next words, left-aligned and zero-padded.

This means access to an arbitrary field in the structure requires a worst-case linear amount of reads. Future encodings using it should take it into consideration to put less-frequently-accessed members last.

## Examples 

`0x1234`: `0x1234000000000000000000000000000000000000000000000000000000000005`

`0x01FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF1234`: `0x01FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF12343E`

`0x00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF`:
```
0x
0000000000000000000000000000000000000000000000000000000000000041
00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
```

`0x00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF0011`:
```
0x
0000000000000000000000000000000000000000000000000000000000000045
00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
0011000000000000000000000000000000000000000000000000000000000000
```

## Rationale
With this approach, packing the data right after the word saving its length instead of storing the offset to the data, we don't allow for extra information to be inserted in a way that is not visible to parsers (as is possible with ABI encoding), which allows for the canonicity of encoded addresses.

# Appendix D: checksum computation
With the intent of:
- Maximizing the difficulty of mining checksum collisions
- Maximize the safety a user gets from knowing the checksum matches
- Allow maximum flexibility for human-readable names, leveraging the above to make human-readable name collisions permissible.

And considering:
- Binary payloads will often have their own error-correction mechanisms when in transit (eg: if encoded into a QR code, said QR code will have information redundancy & its own checksum ensuring only valid data can be decoded), so that can safely be deemed out of scope.
TODO: Considering the above, does it make sense to store the checksum inside the address?

We propose a way to serialize the 'abstract' components of an address, ignoring the information on how it should be displayed, to achieve the above: 

```solidity
    function getChecksum(bytes2 caip2Namespace, bytes memory chainId, bytes memory address_)
        internal
        view
        returns (bytes4 checksum)
    {
        bytes memory fullChainId = bytes.concat(caip2Namespace, chainId);
        // Deliberately abi-encode instead of encode-packed to avoid preimage collisions
        bytes memory serialized = abi.encode(fullChainId, address_);
        bytes32 hashed = keccak256(serialized);
        checksum = bytes4(hashed);
    }
```
TODO: describe this in a way which is not potentially coupled to solidity (or even a specific version of it)
