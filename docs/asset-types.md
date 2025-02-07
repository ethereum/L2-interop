# Asset Types

Ethereum has a long history of enabling the creation of interchangeable assets, most of which follow an ERC standard. Typically, asset creation converge into one of these three well-established standards:

- [ERC-20](https://eips.ethereum.org/EIPS/eip-20): The most widely used standard for fungible tokens, defininf a set of functions that enable transfers, approvals and balance tracking.
- [ERC-721](https://eips.ethereum.org/EIPS/eip-721): The standard for non-fungible tokens, ensuring uniqueness and ownership tracking.
- [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155): A multi-token standard that supports both fungible and non-fungible tokens winthin a single contract, optimizing batch transgers and reducing gas costs.

These standards define how an asset is structured, including its fundamental properties and fungibility rules. However, they do not specify how assets should function in a multi-chain environment. To address this, various standards and frameworks have been proposed over time.

# Common Bridging Methods

- **Lock and Mint**: The original token representations are directly deposited into a smart contract, where the underlying cross-chain messaging protocol facilitates the minting of the same amount of tokens on the destination chain. Typically, this system is isolated between the chain where the tokens are locked and the chain where they are minted, as seen in canonical bridges.  

- **Burn and Mint**: The token representation is considered virtually native across multiple chains, allowing it to be burned on one chain and minted on another. This ensures that all deployed token representations trust each other and rely on the cross-chain messaging protocol used.

# ERC-20-Compatible Standards and Frameworks

There have been a large number of distinct efforts to enable tokens to acquire cross-chain capabilities. In some cases, standards have been formally proposed under ERC tracks, while others have been introduced more as products aligned with specific bridging designs.

## ERC-7802: Crosschain Token Interface

[ERC-7802](https://github.com/ethereum/ERCs/pull/692) introduces the most minimal interface for tokens to communicate cross-chain. It allows bridges with mint and burn rights to send and relay token transfers with a standardized API. The provided interface is bridge agnostic and fully extensible.

## ERC-7281: Sovereign Bridged Token

[ERC-7281](https://github.com/ethereum/ERCs/pull/89) (aka xERC20) enables token bridging across chains. It introduces the concept of a Lockbox to allow existing tokens to comply with the specification through a familiar wrapping mechanism and exposes new interfaces that allow token issuers to apply custom risk profiles (through "rate limits") at a per bridge per chain granularity.

## Token Frameworks

Historically, various Cross-Chain Messaging Protocols have been motivated to create their own standards that fit their systems and may have varying degrees of flexibility for external integrations. The list includes:

- [CCT](https://docs.chain.link/ccip/concepts/cross-chain-tokens): Built by Chainlink.
- [Interchain Tokens](https://docs.axelar.dev/dev/send-tokens/interchain-tokens/intro): Built by Axelar.
- [NTT](https://wormhole.com/docs/learn/messaging/native-token-transfers/overview/): Built by Wormhole.
- [OFT](https://docs.layerzero.network/v2/home/token-standards/oft-standard): Built by LayerZero.
- [Warp Tokens](https://docs.hyperlane.xyz/docs/guides/deploy-warp-route): Built by Hyperlane.


## Canonically Bridged Tokens

Layer 2s and sidechains commonly implement canonical bridges, which generally operate within the enshrined messaging layer and allow tokens to be deployed in a permissionless manner. These tokens can be deployed as "vanilla" implementations or with custom properties, as long as they comply with the required interface and grant mint/burn rights to the canonical bridge. The actual implementation and methods may vary, but some well-known examples include:

- OP Stacks's OptimismMintableERC20
- Arbitrum's StandardArbERC20
- Scroll's ScrollStandardERC20
- Polygon PoS's UChildERC20
- And more.

## Others

Other teams have chosen to implement fully independent token frameworks and bridging mechanisms. The most notable example is the Bridged USDC Standard and native USDC with CCTP.

# ERC-721-Compatible Standards and Frameworks
[WIP].

# ERC-1155-Compatible Standards and Frameworks
[WIP].

## Useful Links
1. [Comparing Token Frameworks](https://li.fi/knowledge-hub/comparing-token-frameworks/)
