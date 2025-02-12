# Intents

It's called an intent to any user request that can be fulfilled by an agent in a fast manner. The user commonly "declares" what outcome they desire (e.g., fund transfer, token swaps) and it is allowed to be completed by solvers under given pricing, possibly in a competitive environment.

In interoperability, intents are key to enabling "cross-chain swaps," in which the user and solver (or underlying system) exchange assets in both origin and destination domains. This is particularly relevant in cases where users expect to initiate an action in one chain to be completed in another in a matter of seconds, in contrast to typical settlement wait times.

Intents are also considered a proper solution to minimize the consequences of liquidity fragmentation in a world where token representations are not homogeneous nor interconnected across chains.

## ERC-7683: Cross Chain Intents Standard

[ERC-7683](https://github.com/ethereum/ERCs/blob/master/ERCS/erc-7683.md) proposes a standard API for cross-chain value-transfer systems. A participant called a filler fulfills intents in the destination initiated by users and is paid out through a settlement system. Users are protected since their funds are escrowed in the origin chain and are released when the action is verified, commonly through a message passed and validated.

A CrossChainOrder is initiated (gasless or not gasless) and is resolved even when partitioned into several "legs." Note that the standard does not impose an opinion on the validation mechanism used to verify when an intent is completed.

For a gasless cross-chain Flow:

```mermaid
---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant Filler
    participant OriginSettler as IOriginSettler (Chain A)
    participant DestinationSettler as IDestinationSettler (Chain B)

    User-->Filler: sign GaslessCrossChainOrder (off-chain)
    Filler->>OriginSettler: openFor(order, signature, originFillerData)
    OriginSettler->>OriginSettler: emit Open(orderId, resolvedOrder)
    Note over OriginSettler: Emits Open with fill instructions
    Note over Filler: Bridging or messaging flow to <br/> relay fill data cross-chain

    Filler->>DestinationSettler: fill(orderId, originData, fillerData)
    DestinationSettler->>DestinationSettler: Validates & finalizes fill
```

For an on-chain cross-chain flow:

```mermaid
---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant OriginSettler as IOriginSettler (Chain A)
    participant Filler
    participant DestinationSettler as IDestinationSettler (Chain B)

    User->>OriginSettler: open(order)
    OriginSettler->>OriginSettler: emit Open(orderId, resolvedOrder)
    Note over OriginSettler: Emits Open with fill instructions

    Filler->>DestinationSettler: fill(orderId, originData, fillerData)
    DestinationSettler->>DestinationSettler: Validates & finalizes fill
```

This standard intentionally does not prescribe the specifics of final settlement logic or how cancellations (e.g., revoking an unfilled order) should be handled. Implementations may choose to use any cross-chain messaging system.
