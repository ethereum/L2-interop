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

## Machine address format definition #1: easiest-calldata

In order to maximize ease of use with most existing infrastructure, the format is deliberately solidity-ABI-compatible. For environments where solidity-ABI is not a native encoding, userspace parsers could be easily written.

This standard defines the format at the binary level, but for better readability, solidity structs with solidity types will be used throughout this document.

The foundation for the Interoperable Address format is the following struct:

```solidity
struct InteroperableAddress {
    bytes32 version_checksum_reserved;
    bytes chainId;
    bytes address_;
}
```

where:

`version`
: Defines the Interoperable Address version, which will be useful to know if to expect any extra fields at runtime. `uint16` stored in the most significant bits (240-255) of the `version_checksum_reserved`

`checksum`
: A 4-byte value computed over the entirety of the Interoperable Address payload (details below). `bytes4` stored in bits (239-208) of `version_checksum_reserved`

`reserved`
: Bytes that ABI-encoding would otherwise pad to zero, which can have specific uses in future versions of the standard.

`chainid`
: Binary representation of the CAIP-2 blockchain id. See [Appendix A](#Appendix-A)

`address_`
: Raw binary representation of the address in the chain. For `eip155` chains, this is trivial. Other chains can have different types of addresses which include more than just a public key, and those will be serialized as described in [Appendix B](#Appendix-B).

## Machine address format definition #2: compact
The alternative above wastes a lot of space in order to be compliant with ABI spec. Another alternative is to define an address format with packing more similar to what is used for Solidity contract's storage layout. Addresses will be more compact in transit, but turning their components into Solidity types will not be trivial, requiring a library.

### Word-by-word definition of the format

#### Word 0:
```
MSB                                                            LSB
0x0000000000000000000000000000000000000000000000000000000000000000
  ^^^^------------------------ 255-240 Interoperable Address version
      ^^^^^^^^ --------------- 239-208 Checksum
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                \------------- 207-0   Reserved
```

#### Word 1:
Binary-encoded CAIP-2 blockchain id (following [Appendix A](#Appendix-A)) inside a short-packed bytes array, as described in [Appendix C](#Appendix-C).

#### Word 2:
Binary-encoded address (following [Appendix B](#Appendix-B)) inside a short-packed bytes array, as described in [Appendix C](#Appendix-C).

#### Word 3+:
Optionally used by the fields defined above, in the case chain ids or addresses are long enough to merit it.
Further words may be used by future versions of the standard

## Checksum definition & validation
The high-level description of how to compute the checksum is:
- Set checksum to zero.
- Serialize the machine address.
- Compute the `keccak256` hash of the serialized address.
- Insert the first four bytes of the hash where the checksum is supposed to be
- Serialize the machine address again

TODO: should the checksum go over the entire address, including the instructions on how to display it, or only the fields required for version zero? I'm specifying the former but leaning for the latter

### Example
Assuming machine address format #1, CAIP-2 eip155 namespace has binary id `0x0001`, network is eth mainnet, and address in that network `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`:

```
InteroperableAddress memory addy = InteroperableAddress({
    version_checksum_reserved: bytes32(uint256(uint16(2))) << 240,
    chainId: hex'000101',
    address_: hex'd8dA6BF26964aF9D7eEd9e03E53415D37aA96045'
});
```
Serlialization with checksum set to zero:
``` 
0x
0000000000000000000000000000000000000000000000000000000000000020
0002000000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000000000000000060
00000000000000000000000000000000000000000000000000000000000000a0
0000000000000000000000000000000000000000000000000000000000000003
0001010000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000000000000000014
d8da6bf26964af9d7eed9e03e53415d37aa96045000000000000000000000000
```
keccak256 of the above:
```
0xeb43550c612da0271a1f59bf6e9e4f3572139769ba88e9d05337d31fa1c2d2c2
```
Inserting the checksum into the address:
```
addy.version_checksum_reserved |= bytes6(keccak256(abi.encode(addy))) >> 16;
```
Serialized address including checksum:
```
0x
0000000000000000000000000000000000000000000000000000000000000020
0002eb43550c0000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000000000000000060
00000000000000000000000000000000000000000000000000000000000000a0
0000000000000000000000000000000000000000000000000000000000000003
0001010000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000000000000000014
d8da6bf26964af9d7eed9e03e53415d37aa96045000000000000000000000000
```

## Human-readable name format definition

### Syntax
```bnf
<human readable name>      ::= <resolver version>=<version-specific payload>
<resolver version>         ::= [0-9]{1,4}
<version-specific payload> ::= [@-.%a-zA-Z0-9]*
```

## Rationale
- In order to guarantee future resolvers don't share prefixes with existing resolvers in an ambiguous way, and therefore human-readable names are always mappable to a single machine address, this standard defines the human-readable names to start with the resolver version. (TODO: ideally we should be smarter about this and resolve it in a way that doesn't expose this implementation quirk to users)

## Encoding considerations
- on-chain actors, such as smart contracts, MUST always receive and return the machine addresses as byte array
- off-chain actors, such as wallets, MAY use the blockchain-native byte array representation, or, where practical MAY alternatively use base64 as defined in RFC-4648 to encode the former.
- users SHOULD NOT be shown the base64 or binary representations, instead, they should be shown the intended human-readable name or, in cases where that is not possible, falling back to interpreting it as Interoperable Address version zero.

## Interoperable Address version definitions

### `0` : No resolution
This is used to indicate no resolution, with the human-readable name being equal to the machine address. The machine address MUST be shown in full.

#### Examples:
Machine address
: TODO

Human-readable name
: `0:::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#6164da10`

### `1` : No resolution, compact format
This is used to indicate no resolution, but removing all redundant information that can be verified by the wallet. It is equivalent to the CAIP-10 account id, with the canonicalization caveat described above.

#### Examples:
Machine address
: TODO

Human-readable name
: `1::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

### `2` : ERC-3770 chain name resolution only
The human-readable name in this approach is consistent with the address format defined in ERC-3770, and similarly depends on the EF's mapping of chain ids to names maintained in github at: https://github.com/ethereum-lists/chains

#### Human-readable name format

```bnf
<human readable name> ::= 2::<short chain name>:<EIP-55 formatted address>
```
Since wallet software can always validate the checksum, it MUST be removed.

EIP-55 or applicable scheme is used for canonicalization only.

#### Examples:

Machine address
: TODO

Human-readable name
: `2::eth:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

### `3` : ENSIP-11 + ENSIP-19 for raw address resolution + ethereum-lists chain name resolution
This resolver relies on the centralized [SLIP44](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) list for ENS standards compliance and on the one used by ERC-3770 to define the short chain names.

The machine address, in turn, specifies what will the source of truth be for the `raw address -> ens-like name` mapping, expecting it to conform to ENSIP-11.

The chain/contract on which the ENSIP-11 contract used to resolve the human-readable name resides is abstracted away from the user, but implementing wallets MUST maintain a list of those considered trustworthy and warn the user or choose to display the machine address in full when the contract it instructs to use is outside the set.

Also, wallets MAY have a default registry they use for converting human-readable names into machine addresses. This means different wallets could potentially resolve the same human-readable name to different machine addresses which potentially also map to different raw addresses. This should be mitigated by wallets' choices of valid and default registries.

#### Human-readable name format
```bnf
<human readable name> ::= 3::<punycode-encoded name>@<chain-spec>#<checksum>
<chain-spec>          ::= <CAIP-2 chain id>|<ERC-3770 shortName>
```

#### Machine-address format
TODO

#### `machine address -> human-readable name` resolution
1. Extract CAIP-2 `chainId` and raw address from TODO
    - Failure mode: wallet can't call the contract (eg: does not have access to an rpc of that chain)
    - Failure mode: `chainId` is not an EVM address
2. Look up in wallet's 'trusted registry' set (which MAY be locally updated by the user) and see if the contract is contained in it. If it is not, issue a warning to the user.
3. Extract `chainId` and raw address from `TODO`.
4. Convert `chainId` from step above into ENSIP-11 `coinType`
5. Compute the namehash for the ENSIP-19 reverse resolution from the results of step 3 and 4
    - TODO: check if ambiguities or other edge cases are possible when converting from CAIP-10 into the raw form required by ENSIP-11
6. Call `resolver(bytes32 node)`on the contract obtained in step 1, using the value from the step above for `node`.
7. Call `name(bytes32 node)` on the contract returned by the step above. Save the response as the `<punycode-encoded name>`.
8. Check direct resolution of name obtained in the step above, and fail the resolution if it does not match, as described in ENSIP-19
9. For the `<chain-spec>`, extract the CAIP-2 chain id from `<CAIP-10 account id>`. If said chain has an entry in ethereum-lists, display its shortname. Otherwise, display the raw CAIP-2 chain id.
10. Append the checksum from the machine address.

#### `human-readable name -> machine address ` resolution
1. Let the user input a destination chain name, and convert it to a CAIP-2 chainId.
2. Convert the result from step 1 into an ENSIP-11 `coinType`.
3. Let the user input a unicode string instead of the punycode-encoded name.
4. Convert the string from step 3 into a punycode-encoded name.
5. Compute the punycode-encoded name's namehash.
6. Call `addr(namehash, coinType)` on the wallet's default resolver, with the values from steps 5 and 2 respectively.
    - If it returns the zero address, prompt the user to pick another trusted resolver where the name is defined.
    - Failure mode: name cant be resolved with trusted resolvers.
7. Construct `TODO` with the resolver used in the step above.
8. Construct `TODO` with the address returned in step 6 and the destination chain from step 1.
9. The user should obviously not be prompted for the checksum, but it MUST be displayed somewhere in the wallet UI for the user to optionally validate with the receiver that they are using the same resolver.

#### Pre-validations
- The CAIP-2 `chain_id` included within both CAIP-10 account ids can be mapped to a valid ENSIP-11 `coinType` which extends both ENSIP-9 and SLIP44

#### Semantics
- ENS uses [punycode](https://www.unicode.org/reports/tr46/) to encode non-ascii characters, which SHOULD be rendered by wallets. In cases where the wallet could infer the presence of non-ascii characters to be unlikely (eg: depending on locale), a warning MAY also be issued to the user.
- `<punycode-encoded name>` is to be the fully qualified name, including the TLD. 
- For chains listed in `ethereum-lists/chains`, the short name MUST be used, taking precedence over the CAIP-2 chain id.

#### Rationale
This is proposed as a way to have functional human readable names with existing tooling and infrastructure, while being flexible in not enshrining a particular contract to allow for future changes on where & how names are resolved. Future developments in chain configs and name registries should improve upon it's decentralization characteristics and render it obsolete in the long-term.

Since it's possible for users' wallets to use different name registries by default when computing a machine address from a human-readable name, the checksum for the machine address is provided as a layer of defense against losing funds either by mis-configuration or deliberate attacks taking advantage of the differences between the data contained in different naming registries.

#### Examples:

Machine address
: TODO

Human-readable name
: `3::vitalik.eth@eth`

#### Security considerations
- If the resolver lives in the destination/source chains, then securing a trustworthy RPC is already in scope of the wallet's responsibilities to operate safely
- If the resolver lives in L1 or a name-specific rollup, then presumably it's feasible to run a light client of said network as part of the infrastructure trusted by the wallet

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

This means access to 

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
- Consider tradeoffs of following a head-tail encoding scheme a la ABI, instead of packing the data right after. It would be possible to pack the (relative to current position? absolute to start of payload?) offset into 16 bytes, and use the remaining 16 bytes for `length * 2 + 1`.
