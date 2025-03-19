# Properties

This document outlines the required properties for existing interoperability tracks. For each one, properties are categorized as:

- **Must-have**: Core requirements that any solution MUST implement.
- **Should-have**: Important features that solutions SHOULD implement.
- **Nice to have**: Optional features that improves the solution.
- **Non-Goals**: Explicitly out of scope.

# Chain-Specific Addresses

L2s and interconnected chains complicate how we identify accounts and targets for transactions. This becomes clear when looking at common scenarios:
* The same address on different chains may represent completely different entities, or in some cases, may not _exist_ at all.
* Different addresses across chains representing the same logical entity.

From the integration perspective, we can separate cross-chain identification into two distinct concerns:

- **Cross-Chain Naming**: Human-readable names or identifiers that can be resolved to cross-chain addresses (e.g., alice@rollup.eth).
- **Cross-Chain Addressing**: Machine-readable addresses for uniquely identifying addresses across chains (e.g., eth:rollup:0x123...abc).

_For a comparison of current efforts, see [here](./docs/chain-specific-addresses/csa-current-efforts.md)._

The following properties outline the requirements for solutions in both spaces.

## Cross-Chain Addressing

### Must-have

1. **Address Uniqueness**
- Unique and therefore canonical identification of addresses per chain.
- Prevention of cross-chain ambiguity.

2. **Format Compatibility**
- Support for arbitrary "blockchain" address formats not constrained to EVM.
- Consistent encoding and decoding of its format.

3. **Implementation Requirements**
- Deterministic resolution from chain identifier + address.
- Guarantee there is a binary representation that can be passed, stored, and decoded unambiguously in smart contract environments.
- Clear error handling for invalid identifiers.
- Support for checksums and pre-validations.

### Should-have

1. **Extensibility**
- Support for future chain types and formats.
- Flexible implementation details.

2. **Integration Support**
- Alignment with existing standards like DIDs and CAIP-10.

### Nice to Have

- Resolution path using on-chain config registries.

### Non-Goals

- Human readability as primary concern.
- Enforcing specific name resolution services.
- Standardizing chain identifiers (covered by other standards).

## Cross-Chain Naming

### Must-have

1. **Name Resolution**
- Deterministic resolution to cross-chain addresses.
- Support for hierarchical naming patterns.
- Clear chain identification within names.

2. **User Interface**
- _Human_-readable format.
- Reasonable length for manual validation.
- Clear syntax for separating name and chain components.

3. **Wallet Implementation Requirements**
- Trusted verification method (e.g. on-chain registry).
- Safe handling of malformed or invalid names.

### Should-have

1. **Integration Support**
- Compatibility with name service resolution such as ENS.
- Alignment with identity standards like DIDs.

2. **Error Prevention**
- Support for checksums at the name level.

### Nice to have
- Reliance path in on-chain config registries.

### Non-Goals
- Enforcing specific name resolution services.

## Message Passing

[To be added]

## Cross-Chain Intents

[To be added]
