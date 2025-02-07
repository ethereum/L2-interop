# Chain Registries

Historically, the ecosystem has relied on off-chain registries and social consensus through centralized databases like [ethereum-lists](https://github.com/ethereum-lists/chains) to maintain chain identifiers as unique per chain. However, this creates dependencies on external maintainers and does not provide a trust-minimized way to resolve chain names with identifiers.

## ERC-7785: Onchain Registration of Chain Identifiers

[ERC-7785](https://ethereum-magicians.org/t/erc-7785-onchain-registration-of-chain-identifiers/21299) proposes maintaining an on-chain registry using ENS. It aims to facilitate the scalability of chain lists as new chains are created, ensure censorship resistance, and protect against spoofing and replay attacks.

Rollups can be resolved as follows: for example, a given `rollup.eth` resolves to `{version, bridge, chain_id}`.
