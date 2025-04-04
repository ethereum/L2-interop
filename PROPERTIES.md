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

### Must-have

1. Safety - Assuming the mode of message verification is honest, a message is delivered at the destination iff it was sent at the source chain. 

2. Liveness - Assuming liveness and censorship-resistance of the source and destination chains, a sent message is eventually delivered at the destination chain. 

3. VM-Agnosticism - Assuming the VM allows arbitrary logic, the interface can be implemented on any VM.  The interface works between many different VM implementations. 
* Balance this goal with goal 6; the interface shouldn't be overly complex in order to achieve VM-agnosticism. 

4. Proof-Agnosticism - The interface makes no assumption about how messages are proved to be valid. Many unique proof systems will be used in the underlying messaging protocols, and the interface works the same across all of them. 

5. Protocol-agnosticism - The interface makes no assumptions about how the underlying messaging protocol works. 

6. Optimized for most use cases - The interface optimizes to provide the best developer experience for bridging and cross-chain function calls across L2s, L3s, and Ethereum. 
* Supporting all use cases results in a more complex interface.  This may lead developers to use wrapper contracts to simplify the interface or to not adopt the interface at all, defeating the purpose of the interface.  
* Edge cases can be handled by the messaging protocol itself. Some edge use cases may depend on the specific messaging protocol used. 

### Should-have

1. Timely Delivery - Assuming liveness and censorship resistance of the source and destination chains, messages should arrive within a known, bounded time delay. 

2. Allows arbitrary data transfer - Interface allows applications to send arbitrary data between chains, vs. forcing cross chain messages to represent tokens exclusively. 

### Nice to Have

1. L1 <-> L1 messaging - The interface should work to interoperate across L1s in the same way it interoperates within the Ethereum ecosystem. (This property is implied by the other properties). 

2. Gas abstraction - The interface should support the ability for users to pay gas on the destination chain from the source chain, since this is required in most use cases. 

### Non-Goals

1. Message broadcasting - While the underlying protocols may implement a message broadcast mechanism, the interface does not need to optimize for this use case. 

2. Ordering - The interface does not enforce the order in which messages arrive. 



## Cross-Chain Intents

[To be added]
