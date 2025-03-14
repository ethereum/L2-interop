# Cross-chain interoperable addresses specification
This document, in its current state, aims to be the starting point for a future address format called 'Interoperable Address' whose aims & properties are fully described in [a separate document](../PROPERTIES.md)

Requires: RFC-4648, ENSIP-11, CAIP-2

## Abstract
An extensible way to describe an address specific to one chain which includes the information to securely condense it into a human-readable name.

## Out of scope concerns
Similarly to CAIP-10, this specification is not concerned with the mapping from a chain id to a network name, which might not be surjective (eg: the case where if there are multiple EIP-155 chains with chain id 8453, which one should we call Base?), regarding that resolution a social-layer problem until a future ERC decides to tackle it. Efforts in that front are tracked in [the chain registires document](../docs/chain-registries.md)

## Definitions
Interoperable address
: A binary payload which includes both the information to unambiguosly point to a specific address on a specific chain and a _resolver specification_ which, together with this (and potentially future) ERCs, allows conversion to a human-readable name in a way that requires no further trust assumptions than those the wallet software executing said conversion already operates under.

Human-readable name
: A string representation of an interoperable address, condensed into something that is to be consumed by humans.

## Machine address format definition
The alternative above wastes a lot of space in order to be compliant with ABI spec. Another alternative is to define an address format with packing more similar to what is used for Solidity contract's storage layout. Addresses will be more compact in transit, but turning their components into Solidity types will not be trivial, requiring a library.

### Format definition
Word 0 Is the only word which will be common to all versions, and is defined as follows:
```
MSB                                                            LSB
0x0000000000000000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                \------------- 207-0   Reserved
```

No length restriction is placed on how many more words can an Interoperable Address span. Different versions of the standard can define uses for extra words and the 208 reserved bits.

### Restrictions for future versions
- The Interoperable Address MUST include an address on a particular chain and MUST also specify on which chain said address exists.
- Interoperable Address versions MAY only be able to represent a subset of the CAIP-2 namespaces
- Interoperable Address versions MAY define extra fields containing information on how to display the addresses.

## Human-readable name format definition

### Syntax
```bnf
<human readable name> ::= <account>@<chain>
<account>             ::= TODO
<chain>               ::= TODO
```

## Encoding considerations
- On-chain actors, such as smart contracts, MUST always receive and return the machine addresses as byte array
- Off-chain actors, such as wallets, MAY use the blockchain-native byte array representation, or, where practical MAY alternatively use base64 as defined in RFC-4648 to encode the former.
- Users SHOULD NOT be shown the base64 or binary representations, instead, they should be shown the intended human-readable name or, in cases where that is not possible, falling back to interpreting it as Interoperable Address version TODO.

## Interoperable Address version definitions

### `0x0000` : single-word 48-bit EVM chainids, no human-readable name resolution
This encoding is meant to be the shortest possible way to encode most EVM addresses

#### Supported chains
EVM chains with a chainid shorter than 48 bits (which potentially rules out chainids chosen to comply with ERC-7785)

#### Machine-address format

```
MSB                                                            LSB
0x0000000000000000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^
                \------------- 207-160 Chainid
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   Address
```

`Interoperable Address version`
: Always `0x0000`

`Checksum`
: Computed as described in [Appendix D](#Appendix-D)

`Chainid`
: `uint48` containing the chainId for the target chain

`Address`
: Native EVM address

#### Human-readable name resolution
The CAIP-2 namespace is to be rendered alongside the `0x`-prefixed EIP-55 address and the checksum

```
<address>@<CAIP-2 namespace>#<checksum>
```

#### Examples:
Machine address
: 
```
MSB                                                            LSB
0x0000618ad0d1000000000001D8DA6BF26964AF9D7EED9E03E53415D37AA96045
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^
                \------------- 207-160 Chainid
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   Address
```

Human-readable name
: `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#6164da10@eip155:1#618ad0d1`

### `0x0001` : Compact format, ENSIP-11+ERC-3770 name resolution 
The binary representation is identical to the format above, with the exception that the version number is different, indicating the intent for the chain to be displayed using the ERC-3770 shortname and the address to be displayed following mainnet's ENS deployment and ENSIP-11

#### Supported chains
Chain must be listed on ethereum-lists/chains on top of the restrictions as `0x0000`.

#### Machine-address format
Same as `0x0000`, with `0x0001` version

#### Examples:
Machine address
: 
```
MSB                                                            LSB
0x0001618ad0d1000000000001D8DA6BF26964AF9D7EED9E03E53415D37AA96045
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^
                \------------- 207-160 Chainid
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   Address
```

Human-readable name
: `vitalik.eth@eth#618ad0d1`

TODO: example on another chain

## Requirements for wallet software
- Wallet software MUST perform all relevant pre-validations, including verifying the checksum, and report any errors to the user, for every defined resolver. Wallets MAY reject an interoperable address solely on the basis of these checks failing.
- TODO: what to do when a machine address' resolver version is not supported by the wallet
    - option 1: show it as is -> easiest
    - option 2: convert it to resolver `1` -> guarantees not-that-bad-to-read address is shown to the user, might impose extra constraints on existing resolvers.

## Recommendations for rollups
TODO: probably coupled to ERC-7785, currently out of scope

## Rationale
- Checksum algorithm is independent of used resolution method to allow wallets to validate an interoperable address' checksum despite not being able to generate its human-readable name
- Ideally we'd want the resolution method to be fully abstracted away from the user, but that might not be achievable in every case. The next best thing is to prefix every human-readable name with a string denoting which method to use, which prevents the user ever being prompted to choose different addresses based on the resolution method used for each, which would be a more confusing and riskier UX 
- Base64 was chosen since it is a more widely implemented standard than base58, and since machine addresses are not to be directly displayed to the user, the collision-avoidance reasons in favor of base58 do not apply

## Compatibility with other public-key sharing standards

### W3C DID
TODO

## Security considerations

## Appendix A: Binary encoding of CAIP-2 blockchain id
First two bytes are the binary representation of CAIP-2 namespace (see table below), rest are the CAIP-2 reference.
In the special case of eip155 chains, the bare `chainid` encoded as a `uint` of the necessary amount of bytes will be used [^1]. This allows for the existing EVM chainids fit inside one EVM word.
Reference will be the ASCII-encoded value from CAIP-2 (from, and not including, the colon, up to the end of the string) in all other cases.

[^1]: This makes it possible to represent some chains using the full word as their chainid, which CAIP-2 does not support since the maximum value representable with 32 `a-zA-Z0-9` characters is less than `type(uint256).max`. This is done in an effort to support chains whose id is the output of `keccak256`, as proposed in ERC-7785.

### CAIP-2 namespaces' binary representation table
TODO

### Examples
TODO

## Appendix B: Binary encoding of addresses
TODO, will be similar to ENSIP-9 and/or CAIP-50

### Examples
TODO

## Appendix C: Short-encoding of byte arrays
This is a mix between [how bytes and strings are saved into storage](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#bytes-and-string) and how ABI encoding works

This arises out of:
- The need to have the shortest possible representation for EVM addresses.
- Not having a canonical place to delimit the _start_ of the data (in the case of storage strings, it's the keccak256 hash of the storage slot index).

If the data is at most 31 bytes long, the elements are stored in the higher-order bytes (left aligned) and the lowest-order byte stores the value `length * 2`.

If the data is 32 bytes or longer, the current word stores `length * 2 + 1`, and data follows in the next words, left-aligned and zero-padded.

This means access to an arbitrary field in the structure requires a worst-case linear amount of reads. Future encodings using it should take it into consideration to put less-frequently-accessed members last.

### Examples 

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

TODO: 
- Consider tradeoffs of following a head-tail encoding scheme a la ABI, instead of packing the data right after. It would be possible to pack the (relative to current position? absolute to start of payload?) offset into 16 bytes, and use the remaining 16 bytes for `length * 2 + 1`. (pro of this approach: not possible to add extra data between fields which is invisible to parsers like is possible with abi encoding)

## Appendix D: checksum computation
With the intent of:
- Maximizing the difficulty of mining checksum collisions
- Maximize the safety a user gets from knowing the checksum matches
- Allow maximum flexibility for human-readable names, leveraging the above to make collisions permissible

We propose a way to serialize the 'abstract' components of an address, ignoring the information on how it should be displayed, to achieve the above: 

```solidity
    function getChecksum(bytes2 caip2Namespace, bytes memory chainid, bytes memory address_)
        internal
        view
        returns (bytes4 checksum)
    {
        bytes memory fullChainid = bytes.concat(caip2Namespace, chainid);
        bytes memory serialized = abi.encode(fullChainid, address_);
        bytes32 hashed = keccak256(serialized);
        checksum = bytes4(hashed);
    }
```
TODO: describe this in a way which is not potentially coupled to solidity (or even a specific version of it)
