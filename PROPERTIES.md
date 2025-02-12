# Properties

This document outlines the required properties for existing interoperability tracks. For each one, properties are categorized as:

- **Must-have**: Core requirements that any solution MUST implement
- **Should-have**: Important features that solutions SHOULD implement
- **Nice to have**: Optional features that improves the solution
- **Non-Goals**: Explicitly out of scope

## Chain-Specific Addresses

L2s and interconnected chains complicate how we identify accounts and targets for transactions. This becomes clear when looking at common scenarios:
* Same address on different chains representing completely different entities (or some of them do not "exist")
* Different addresses across chains representing the same logical entity

The ecosystem needs a standardized way to specify both an address and its associated chain. The following properties outline the requirements for solutions in this space.

### Must-have

1. **Address Resolution**
- Unique identification of accounts per chain
- Resolution from chain identifier
- Prevention of cross-chain ambiguity

2. **Format Compatibility**
- Support for arbitrary "blockchain" address formats (not constrained to EVM)
- Support for chain hierarchies (L1/L2/L3)
- Consistent parsing mechanism

3. **User Interface**
- Clear chain identification
- Separation of concerns between human readability and machine addresses
- Error prevention (e.g. reasonable length for manual validation)

4. **Wallet Implementation Requirements**
- Deterministic resolution from human-readable name to machine address
- Safe handling of malformed or invalid identifiers
- Support for checksums and pre-validations

### Should-have

1. **Extensibility**
- Support for future chain types and formats
- Flexibility in implementation details

2. **Integration Support**
- Compatibility with name service resolution (ENS, etc)
- Alignment with existing standards (DIDs, CAIP-10)

### Nice to Have

- Name resolution services standard
- On-chain config registries
- Deterministic resolution from machine address to human-readable name

### Non-Goals

- Enforcing specific name resolution services
- Standardizing chain identifiers (covered by other standards)

For a comparison of current efforts, see [here](./docs/addresses-current-efforts.md).

## Message Passing

[To be added]

## Cross-Chain Intents

[To be added]
