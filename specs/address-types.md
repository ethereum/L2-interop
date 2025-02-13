# Cross-chain interoperable addresses specification

This document, in its current state, aims to be the starting point for a future address format called 'Interoperable Address' whose aims & properties are fully described in [a separate document](../PROPERTIES.md)

Requires: CAIP-10

## Abstract
An extensible way to describe an address specific to one chain which includes the information to condense it into a human-readable name in a secure way.

## Out of scope concerns
Similarly to CAIP-10, this specification is not concerned with the mapping from a chain id to a network name, which might not be surjective (eg: the case where if there are multiple EIP-155 chains with chain id 8453, which one should we call Base?), regarding that resolution a social-layer problem until a future ERC decides to tackle it.

## Definitions
Account id
: A string which unambiguously corresponds to an address on a specific chain (EVM or not). 

Human-readable name
: A shorter representation of an account id

Interoperable address
: A string which includes both a account id and a _resolver specification_ which, together with this (and potentially future) ERCs, allows conversion to a human-readable name in a way that requires no further trust assumptions than those the wallet software executing said conversion already operates under.

## Format definition

### Syntax

```bnf
<interoperable address>             ::= <resolver specification>::<CAIP-10 account id>#<checksum>
<resolver specification>            ::= <resolver version>:<version-specific resolver payload>
<resolver version>                  ::= [0-9]{1,4}
<version-specific resolver payload> ::= [-.%a-zA-Z0-9]* // same as CAIP-10 account address format
<checksum>                          ::= [0-9a-f]{8}
```

### Semantics
- Similarly to CAIP-10, the resolver payload has `%` as a valid character to enable URL-escaping of arbitrary characters
    - The rules for wether to parse URL escaping in a resolver payload are specific to the resolver version
    - The rules for wether to parse URL escaping in a CAIP-10 account address are specific to its CAIP-2 chain namespace
- To maintain the bijectivity of account ids and therefore interoperable addresses to on-chain addresses:
    - In CAIP-2 namespaces where there is a canonicalization scheme (such as EIP-55 in EIP-155 chains) it SHOULD be used (using SHOULD since it would be a potentially unsolvable MUST in the case where there are competing canonicalization schemes)
    - In CAIP-2 namespaces where addresses have an *optional* checksum field, it MUST be omitted and the checksum defined in the standard be used without redundancy
    - In CAIP-2 namespaces where addresses are raw hexadecimal data without an EIP-55-equivalent capitalization scheme, the lowercase `a-f` characters MUST be used instead of uppercase ones.
    - In the case of EIP-155 chains, EIP-55 canonicalization MUST be used
- The checksum MUST be the hexadecimal representation first four bytes of the keccak-256 hash of UTF-8 representation of the interoperable address from it's beginning up to and including the hash `#` character separating the checksum from the rest of the address (rationale: none)

## Rules for current & future resolvers
- Future resolvers MUST NOT share prefixes with existing resolvers in an ambiguous way that, when a user types a human-readable name, they have to be prompted by the wallet to choose which resolver to use. While the `human-readable name -> machine address` resolution might not always yield a single result, ensuring the `human-readable name -> resolving algorithm` does is in scope of this standard.

## Resolver version definitions

### `0` : No resolution
This is used to indicate no resolution, with the human-readable name being equal to the machine address. The machine address MUST be shown in full.

#### Examples:
Machine address
: `0:::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#6164da10`

Human-readable name
: `0:::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#6164da10`

### `1` : No resolution, compact format
This is used to indicate no resolution, but removing all redundant information that can be verified by the wallet. It is equivalent to the CAIP-10 account id, with the canonicalization caveat described above.

#### Examples:
Machine address
: `1:::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#b50c58f9`

Human-readable name
: `eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

### `2` : ERC-3770 chain name resolution only
The human-readable name in this approach is consistent with the address format defined in ERC-3770, and similarly depends on the EF's mapping of chain ids to names maintained in github at: https://github.com/ethereum-lists/chains

#### Human-readable name format

```bnf
<human readable name> ::= <short chain name>:<EIP-55 formatted address>
```
Since wallet software can always validate the checksum, it MUST be removed
EIP-55 or applicable scheme is used for canonicalization only.

#### Examples:

Machine address
: `2:::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#bc797def`

Human-readable name
: `eth:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

#### Considerations
- This resolver does not maintain surjectivity from machine addresses to blockchain accounts, since it only supports a (potentially very small) subset of the `eip155` CAIP-2 namespace

## Requirements for wallet software
- When parsing the CAIP-10 `account_address` for a CAIP-2 chain namespace where URL-escaping or the `%` character is not part of valid addresses, finding URL-encoded data MUST be treated as an error.
- Wallet software MUST perform all relevant pre-validations, including verifying the checksum, and report any errors to the user, for every defined resolver. Wallets MAY reject an interoperable address solely on the basis of these checks failing.

## Recommendations for rollups
TODO: probably coupled to ERC-7785

## Possible failure modes

### Pre-validation of machine addresses
TODO
### Computing human-readable name from machine address
TODO

## Rationale
- URL-escaping of characters might be necessary for future resolvers
- Checksum algorithm is independent of used resolution method to allow wallets to validate an interoperable address' checksum despite not being able to generate its human-readable name

## Compatibility with other public-key sharing standards

### W3C DID
TODO

## Security considerations


