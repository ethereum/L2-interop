# Shared Bridges
There have been ideas for sharing liquidity between multiple Layer 2s by utilizing common L1 escrow contracts, enabling assets to move freely across different chains. The primary objectives are:

1. Provide transfers for assets between rollup domains without requiring independent settlement transactions for each cross-chain transaction.
2. Ensuring withdrawal guarantees across all participating chains.
3. Use of standardized bridge and asset contracts to maintain consistency (nice to have).

Shared bridges may anchor to the existing rollup contracts or operate a minimal infrastructure that rollups rely on. In any case, shared bridges may require trust in the state transitions of each chain or integrate the entire infrastructure into a common proof system.

## Useful Links
1. [Embedded Rollups, Part 2: Shared Bridging](https://ethresear.ch/t/embedded-rollups-part-2-shared-bridging/21461).
2. [Trustless Interoperability between Rollups: Landscape, Constructions, and Challenges](https://medium.com/1kxnetwork/trustless-interoperability-between-rollups-landscape-constructions-and-challenges-8ff195ea92cc) (see Shared Settlement).
