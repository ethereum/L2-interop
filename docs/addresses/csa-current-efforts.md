# Chain-specific addresses

A core challenge in Ethereum's multichain landscape is disambiguating addresses across L2s, L3s, Ln, and other EVM ecosystems. In this context, an address by itself does not provide enough information to determine which chain it belongs to, and in the case of smart contracts, its code content may even differ. For this reason, the introduction of chain identifiers and human-readable labels aims to simplify the cross-chain UX.

# Existing efforts

## ERC-3770: Chain-specific Addresses

[ERC-3770](https://eips.ethereum.org/EIPS/eip-3770) prepends a human-readable chain short name as a prefix to addresses (e.g. `eth:0xAbC...123`). A `chainID` could also be used but is not intended to be preferred over readable strings, as this proposal is since designed primarily for UX improvements.

## CAIP-10: Account ID Specification

[CAIP-10](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-10.md) is a chain-agnostic standard that defines a way to identify an account on any blockchain that accomplies with [CAIP-2](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-2.md), such as `eip155:1:0xAbC...123` for Ethereum mainnet. It is not neccessarily intended to be human-friendly, but rather serves as an universal reference for multi-chain tooling like Wallet Connect.

## ERC-7828: Chain-specific Addresses using ENS

[ERC-7828](https://ethereum-magicians.org/t/erc-7828-chain-specific-addresses-using-ens/21930) builds on top of [ERC-7785](https://ethereum-magicians.org/t/erc-7785-onchain-registration-of-chain-identifiers/21299), to integrate with ENS, enabling the storage of chain names within an existing chain ID mapping and moving registrations away from centralized registries. Address formats could take the form of `alice@rollup` or `alice.rollup.eth`, which can be resolved on-chain through wallets.

## ENSIP-9: Multichain Address Resolution

[ENSIP-9](https://github.com/ensdomains/ensips/blob/master/ensips/9.md) introduces a unified way for ENS resolvers to store and return addresses for different blockchains by overloading the `addr` function. Rather than proposing a textual format like `chain:address`, ENSIP-9 leverage from coin types following [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md). This allows resolvers to identify each blockchain by its unique coin type and store the address in its native binary form such as 20-byte hex for Ethereum, base58-decoded bytes for Bitcoin.

### ENSIP-11: EVM compatible Chain Address Resolution

[ENSIP-11](https://github.com/ensdomains/ensips/blob/master/ensips/11.md) extends ENSIP-9 by introducing a dedicated range of coin types for EVM chains. This range prevents collision with existing [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) coin types.

## Comparison

The existing approaches to chain-specific addresses represent an evolution in thinking about cross-chain identification rather than competing solutions. Some standards build upon lessons learned from previous implementations, while other target other aspects or build on top of them.

| **Feature** | ERC-3770 | CAIP-10 | ERC-7828 | ENSIP-9/ENSIP-11 |
| --- | --- | --- | --- | --- |
| **Scope** | Primarily a UI/UX layer standard for human-readable prefixes | A universal account identifier format (machine-readable) for all blockchains | On-chain naming integration with ENS for chain names and addresses, EVM-focused | ENS resolver-level standards for storing/retrieving multi-chain addresses |
| **Status** | Draft (Fully defined) | Final (Fully defined) | Draft (Incomplete) | Final/Draft |
| **Format Example** | `chain:address` | `chain_namespace:chain_reference:address` | `address:chain.eth` or `address@chain.eth` | Still uses typical `.eth` format |
| **Human Readability (_best-case scenario_)** | Medium | Medium | High | High (from typical ENS format) | 
| **Technical Compatibility** | EVM only (but extensible) | All chains | EVM only (potential non-EVM support) | Blockchain that are part of [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) or follows EVM `chainId` specs |
| **ENS Integration** | Not Required | Not Required | Required | Required |
| **DID Compatibility** |  |  |  |  |
| **Checksum Support** | Incomplete (ERC-55 only) | No | Yes | Yes |
| **Usage of lists** | Yes (referencing github.com/ethereum-lists/chains) | No | Yes (requires ERC-7785 aka onchain registry) | Base in [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) |

# Additional Considerations

Unique chain identifiers rely on social consensus to avoid collisions that could break integrations. The transition from off-chain registries to on-chain methods in the Ethereum ecosystem has been widely discussed and documented [here](chain-registries.md).
