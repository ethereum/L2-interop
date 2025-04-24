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
