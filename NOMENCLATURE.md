# Nomenclature

This document defines the common terminology and conventions to be used across all interoperability standards and related documents in this repository.

## Core Terminology

- **L1 (or Layer 1)**: A blockchain that is self-reliant on its validator set for its security and consensus properties.
- **L2 (or Layer 2)**: A blockchain aimed to scale the L1 in a trust minimized way  while maintaining a "safe bridge" between them. Blockchains in this category include Rollups and Plasma solutions, and may also include other less-secure solutions such as Validiums and Optimiums. For a detailed characterization of this category, please refer to [L2Beat](https://l2beat.com/).
- **L3 (or Layer 3)**: A blockchain that follow a similar structure to the L2 <> L1 structure but with an L2 underneath.
- **Origin Chain**: The chain where a cross-chain operation originates.
- **Destination Chain**: The chain where a cross-chain operation is received/executed.
- **Gateway/Bridge**: A system enabling communication between chains.
- **User**: The end-user initiating a cross-chain operation.
- **Sender**: The account/contract initiating a message on the origin chain.
- **Receiver**: The account/contract receiving a message on the destination chain.
- **Relayer**: An entity facilitating message transmission between chains.


## Addresses

- **Chain Identifier**: A unique identifier for a blockchain used as part of the address formated. Do not confuse it with the Chain ID introduced in [EIP-155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md), even though it may be used as a reference in some cases.
- **Chain Name**: A human-readable identifier for a blockchain.

## Assets

- **`Deposit`**:  The act of sending tokens to a contract that serves as escrow to execute a cross-chain operation, such as minting or releasing tokens on destination chains. Others may refer to this action as `lock`.
- **`Mint` and `Burn`**: These terms refer to the actions of minting (increasing) and burning (decreasing) tokens in two token contracts associated with the same cross-chain operation. Typically, both contracts represent the same token. These actions may also be referred to as `crosschainBurn`/`crosschainMint` or `bridgeBurn`/`bridgeMint`.

## Intents

- **`Fill`/Filling**: The act of executing a user's cross-chain intent on the destination chain. The participant who fulfils a user's intent on the destination chain is called a **filler** or **solver**.
- **`Leg`**: A portion of the user's intent that can be executed independently from the others. All legs must be executed for an intent to be considered fulfilled.
- **Settlement**: The process of finalizing an intent operation, for example by managing user deposits and paying fillers. The contract responsible for this is called the **Settler**.

## Messaging

- **Message**: Data transmitted between chains.
  - **`send`**: Used to denote the act of sending a message. Generally, the functions responsible for this are named `sendMessage`.
  - **`receive`**: Used to denote the act of receiving a message. Depending on the context, receiving a message may have its own flow or involve multiple steps. Functions might be named `receiveMessage`, `relayMessage`, `validateMessage`, `executeMessage` or similar.
- **`Metadata`**: Information about the message (sender, receiver, nonce, etc.)
  - **`ChainId`**: Referes the chain identifier. Others might refer to it as "domain" instead of "chain," or simply as the destination.
- **`Payload`**: The actual data orinstructions being transmitted.
- **Verification**: The process of validating cross-chain messages.
 


---



## Updates:

This document should be updated when:
1. New standards introduce novel concepts
2. Existing terminology requires clarification
3. Community feedback suggests improvements

## Acknowledgments

This document draws its terminology and nomenclature from established (or proposed) Ethereum/Blockchain standards—see the standard references throughout this repository—as well as widely accepted sources such as L2Beat, and aims to reflect the general consensus.
