# ERC-7786 Working Group Notes

## Call #1 (2025-04-24)

- Context
    - Group has consolidated around ERC-7786 but it still needs a few changes
    - Purpose of this series of calls will be to facilitate technical discussions to smooth out these rough edges so everyone feels comfortable backing ERC-7786 as the standard GMP interface we want protocols to implement
    - Potential future agenda items:
        - gas payment interface
        - some form of support for token bridging
        - payload delivery interface
        - interoperable addresses
- Agenda
    - Points raised in [review of unification proposal](https://github.com/ethereum/L2-interop/pull/39#pullrequestreview-2779281098)
        - Hooks and attributes
        - Message id as `bytes32` vs `bytes`
- Hooks
    - Inspired by Hyperlane, OP Interop, Uniswap V4
    - Goal: Peripheral business logic without requiring upgradeability or complex abstractions inside the main contract
    - Examples
        - Batching/bundling of multiple messages (transfer and swap, transfer and rebalance, multi transfer)
    - Currently underspecified, missing specifics about how gateways invoke hooks
- Attributes
    - Debate should not be attributes vs hooks, rather attributes vs extraData, ie. opaque vs structured
    - `extraData` is more clearly protocol-specific. No standardization built into ERC-7786 (`supportsAttribute`) but can be standardized in subsequent ERCs.
    - General agreement to go with `extraData` for the purpose of simplifying the interface and removing potential friction for implementers and users.
- Message id as `bytes32` vs `bytes`
    - `bytes` can be used to encode more complete identification data, eg. transaction hash and log index
    - The gateway can use larger identifiers internally and still deliver a short identifier to the receiver
    - `bytes32` seems favored for simplicity
- Next steps
    - Gather more examples of potential hooks to understand their value
    - Specify exactly how the gateway should invoke hooks

## Call #2 (2025-05-01)

- Glacis adapter compliance with ERC-7786
    - Link: https://www.notion.so/glacislabs/7786-Conversion-1e67a3452406806fa7d8fce81c07eb94
    - Option A: Make Glacis Router compliant
        - Higher level
        - Handles fees, quorum/multisend, retries
            - Would require use of attributes for these features
    - Option B: Make adapters compliant
        - Very close to an ERC-7786 Gateway
        - Includes gas payment
            - Can this be decoupled? Yes but more work
        - May need to finish gas payment spec before working on code changes
            - See Catalyst and Hyperlane for inspiration on different fee mechanisms
            - Can be done by taking fees implicitly
        - LayerZero, Wormhole, Axelar, CCIP, Hyperlane
            - Wormhole uses standard relayers
        - Adapters have some admin-gated functionality / configurability
            - RemoteCounterpartManager
                - configure source/destination adapter pairs
            - chainId mappings
        - ✔ Seems like the way to go
- Hooks
    - See [Hooks Explainer](https://github.com/ethereum/L2-interop/pull/39/files#diff-9dd6a6c891082f3fb3b42f751c1dac4f9bfafc6113fff1acf93aef2dfae40035)
