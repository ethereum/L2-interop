# Chain-specific addresses

A core challenge in Ethereum's multichain landscape is disambiguating addresses across L2s, L3s, Ln, and other EVM ecosystems. In this context, an address by itself does not provide enough information to determine which chain it belongs to and may even differ in its code content (in the case of smart contracts). For this reason, the introduction of chain identifiers and human-readable labels aims to simplify the cross-chain UX.

# Existing efforts

## ERC-3770: Chain-specific Addresses

[ERC-3770](https://eips.ethereum.org/EIPS/eip-3770) prepends a human-readable chain short name as a prefix to addresses (e.g. `eth:0xAbC...123`). A `chainID` could also be used but is not intended to be preferred over readable strings, as this proposal is since designed primarily for UX improvements.

## CAIP-10: Account ID Specification

[CAIP-10](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-10.md) is a chain-agnostic standard that defines a way to identify an account on any blockchain that accomplies with [CAIP-2](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-2.md) (e.g. `eip155:1:0xAbC...123` for Ethereum mainnet). It is not neccessarily intended to be human-friendly, but rather serves as an universal reference for multi-chain tooling (e.g. Wallet Connect).

## ERC-7828: Chain-specific Addresses using ENS

[ERC-7828](https://ethereum-magicians.org/t/erc-7828-chain-specific-addresses-using-ens/21930) builds on top of [ERC-7785](https://ethereum-magicians.org/t/erc-7785-onchain-registration-of-chain-identifiers/21299), to integrate with ENS, enabling the storage of chain names within an existing chain ID mapping and moving registrations away from centralized registries. Address formats could take the form of `alice@rollup` or `alice.rollup.eth`, which can be resolved on-chain through wallets.

## Comparison

The existing approaches to chain-specific addresses represent an evolution in thinking about cross-chain identification rather than competing solutions. Some standards builds upon lessons learned from previous implementations while other targets other aspects or they could built on top of.

| **Feature** | ERC-3770 | CAIP-10 | ERC-7828 |
| --- | --- | --- | --- |
| **Status** | Draft (Fully defined) | Final (Fully defined) | Draft (Incomplete) |
| **Format Example** | `chain:address` | `chain_namespace:chain_reference:address` | `address:chain.eth` or `address@chain.eth` |
| **Human Readability (_best-case scenario_)** | Medium | Medium | High |
| **Technical Compatibility** | EVM only (but extensible) | All chains | EVM only (potential non-EVM support) |
| **ENS Integration** | Optional | Optional | Required |
| **DID Compatibility** | No | Yes | Optional |
| **Checksum Support** | Incomplete (ERC-55 only) | No | Yes |
| **Usage of lists** | Yes (referencing github.com/ethereum-lists/chains) | No | Yes (requires ERC-7785 aka onchain registry) |

# Additional Considerations

Unique chain identifiers rely on social consensus to avoid collisions that could break integrations. The transition from off-chain registries to on-chain methods in the Ethereum ecosystem has been widely discussed and documented [here](chain-registries.md).
