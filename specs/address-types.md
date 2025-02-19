# Cross-chain interoperable addresses specification
This document, in its current state, aims to be the starting point for a future address format called 'Interoperable Address' whose aims & properties are fully described in [a separate document](../PROPERTIES.md)

Requires: CAIP-10

## Abstract
An extensible way to describe an address specific to one chain which includes the information to securely condense it into a human-readable name.

## Out of scope concerns
Similarly to CAIP-10, this specification is not concerned with the mapping from a chain id to a network name, which might not be surjective (eg: the case where if there are multiple EIP-155 chains with chain id 8453, which one should we call Base?), regarding that resolution a social-layer problem until a future ERC decides to tackle it. Efforts in that front are tracked in [the chain registires document](../docs/chain-registries.md)

## Definitions
Account id
: A string which unambiguously corresponds to an address on a specific chain (EVM or not). 

Human-readable name
: A shorter representation of an account id

Interoperable address
: A string which includes both an account id and a _resolver specification_ which, together with this (and potentially future) ERCs, allows conversion to a human-readable name in a way that requires no further trust assumptions than those the wallet software executing said conversion already operates under.

## Machine address format definition

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
    - In CAIP-2 namespaces where there is a canonicalization scheme (such as EIP-55 in EIP-155 chains) it SHOULD be used.
    - In CAIP-2 namespaces where addresses have an *optional* checksum field, it MUST be omitted and the checksum defined in the standard be used without redundancy
    - In CAIP-2 namespaces where addresses are raw hexadecimal data without an EIP-55-equivalent capitalization scheme, the lowercase `a-f` characters MUST be used instead of uppercase ones.
    - In the case of EIP-155 chains, EIP-55 canonicalization MUST be used
- The checksum MUST be the hexadecimal representation first four bytes of the keccak-256 hash of UTF-8 representation of the interoperable address from it's beginning up to and including the hash `#` character separating the checksum from the rest of the address.

## Human-readable name format definition

### Syntax
```bnf
<human readable name>      ::= <resolver version>=<version-specific payload>
<resolver version>         ::= [0-9]{1,4}
<version-specific payload> ::= [@-.%a-zA-Z0-9]*
```

## Rationale
- In order to guarantee future resolvers don't share prefixes with existing resolvers in an ambiguous way, and therefore human-readable names are always mappable to a single machine address, this standard defines the human-readable names to start with the resolver version. (TODO: ideally we should be smarter about this and resolve it in a way that doesnt expose this implementation quirk to users)
- Similarly to CAIP-10, arbitrary characters can be url-encoded.

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
: `2:::eip155:1:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#bc797def`

Human-readable name
: `2::eth:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

### `3` : ENSIP-11 + ENSIP-19 for raw address resolution + ethereum-lists chain name resolution
This resolver relies on the centralized [SLIP44](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) list for ENS standards compliance and on the one used by ERC-3770 to define the short chain names.

The machine address, in turn, specifies what will the source of truth be for the `raw address -> ens-like name` mapping, expecting it to conform to ENSIP-11.

The chain/contract on which the ENSIP-11 contract used to resolve the human-readable name resides is abstracted away from the user, but implementing wallets MUST maintain a list of those considered trustworthy and warn the user or choose to display the machine address in full when the contract it instructs to use is outside the set.

Also, wallets MAY have a default registry they use for converting human-readable names into machine addresses. This means different wallets could potentially resolve the same human-readable name to different machine addresses which potentially also map to different raw addresses. This should be mitigated by wallets' choices of valid and default registries.

#### Human-readable name format

```bnf
<human readable name> ::= 3::<punycode-encoded name>@<chain-spec>
<chain-spec>          ::= <CAIP-2 chain id>|<ERC-3770 shortName>
```

#### Machine-address format

```bnf
<machine address> ::= 3::<CAIP-10 account id of ENSIP-11 contract>:<CAIP-10 account id>#<checksum>
```

#### `machine address -> human-readable name` resolution
1. Extract CAIP-2 `chainId` and raw address from `<CAIP-10 account id of ENSIP-11 contract>`.
    - Failure mode: wallet can't call the contract (eg: does not have access to an rpc of that chain)
    - Failure mode: `chainId` is not an EVM address
2. Look up in wallet's 'trusted registry' set (which MAY be locally updated by the user) and see if the contract is contained in it. If it is not, issue a warning to the user.
3. Extract `chainId` and raw address from `<CAIP-10 account id>`.
4. Convert `chainId` from step above into ENSIP-11 `coinType`
5. Compute the namehash for the ENSIP-19 reverse resolution from the results of step 3 and 4
    - TODO: check if ambiguities or other edge cases are possible when converting from CAIP-10 into the raw form required by ENSIP-11
6. Call `resolver(bytes32 node)`on the contract obtained in step 1, using the value from the step above for `node`.
7. Call `name(bytes32 node)` on the contract returned by the step above. Save the response as the `<punycode-encoded name>`.
8. Check direct resolution of name obtained in the step above, and fail the resolution if it does not match, as described in ENSIP-19
9. For the `<chain-spec>`, extract the CAIP-2 chain id from `<CAIP-10 account id>`. If said chain has an entry in ethereum-lists, display its shortname. Otherwise, display the raw CAIP-2 chain id.

#### `human-readable name -> machine address ` resolution
1. Let the user input a destination chain name, and convert it to a CAIP-2 chainId.
2. Convert the result from step 1 into an ENSIP-11 `coinType`.
3. Let the user input a unicode string instead of the punycode-encoded name.
4. Convert the string from step 3 into a punycode-encoded name.
5. Compute the punycode-encoded name's namehash.
6. Call `addr(bytes32 namehash, coinType)` on the wallet's default resolver, with the values from steps 5 and 2 respectively.
    - If it returns the zero address, prompt the user to pick another trusted resolver where the name is defined.
    - Failure mode: name cant be resolved with trusted resolvers.
7. Construct `<CAIP-10 account id of ENSIP-11 contract>` with the resolver used in the step above.
8. Construct `<CAIP-10 account id>` with the address returned in step 6 and the destination chain from step 1.

#### Pre-validations
- The CAIP-2 `chain_id` included within both CAIP-10 account ids can be mapped to a valid ENSIP-11 `coinType` which extends both ENSIP-9 and SLIP44

#### Semantics
- ENS uses [punycode](https://www.unicode.org/reports/tr46/) to encode non-ascii characters, which SHOULD be rendered by wallets. In cases where the wallet could infer the presence of non-ascii characters to be unlikely (eg: depending on locale), a warning SHOULD also be issued to the user.
- `<punycode-encoded name>` is to be the fully qualified name, including the TLD. 
- For chains listed in `ethereum-lists/chains`, the short name MUST be used.

#### Examples:

Machine address
: `3:eip155:1:0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e::eip155:10xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045#5966be0f`

Human-readable name
: `3::vitalik.eth@eth`

#### Security considerations
- If the resolver lives in the destination/source chains, then securing a trustworthy RPC is already in scope of the wallet's responsibilities to operate safely
- If the resolver lives in L1 or a name-specific rollup, then presumably it's feasible to run a light client of said network as part of the infrastructure trusted by the wallet

## Requirements for wallet software
- When parsing the CAIP-10 `account_address` for a CAIP-2 chain namespace where URL-escaping or the `%` character is not part of valid addresses, finding URL-encoded data MUST be treated as an error.
- Wallet software MUST perform all relevant pre-validations, including verifying the checksum, and report any errors to the user, for every defined resolver. Wallets MAY reject an interoperable address solely on the basis of these checks failing.
- TODO: what to do when a machine address' resolver version is not supported by the wallet
    - option 1: show it as is -> easiest
    - option 2: convert it to resolver `1` -> guarantees not-that-bad-to-read address is shown to the user, might impose extra constraints on existing resolvers.

## Recommendations for rollups
TODO: probably coupled to ERC-7785, might actually be out of scope

## Possible failure modes

### Pre-validation of machine addresses
TODO
### Computing human-readable name from machine address
TODO

## Rationale
- URL-escaping of characters might be necessary for future resolvers
- Checksum algorithm is independent of used resolution method to allow wallets to validate an interoperable address' checksum despite not being able to generate its human-readable name
- Ideally we'd want the resolution method to be fully abstracted away from the user, but that might not be achievable in every case. The next best thing is to prefix every human-readable name with a string denoting which method to use, which prevents the user ever being prompted to choose different addresses based on the resolution method used for each, which would be a more confusing and riskier UX 

## Compatibility with other public-key sharing standards

### W3C DID
TODO

## Security considerations

