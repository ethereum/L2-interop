# Nomenclature

This document defines the common terminology and conventions to be used across all interoperability standards and related documents in this repository.

## Core Terminology

- **L1 (or Layer 1)**: A blockchain that is self-reliant on its validator set for its security and consensus properties. While it may refer to any blockchain that fits this definition, it commonly refers to the Ethereum Mainnet.
- **L2 (or Layer 2)**: A blockchain aimed to scale the L1 in a trust minimized way  while maintaining a "safe bridge" between them. Blockchains in this category include Rollups and Plasma solutions, and may also include other less-secure solutions such as Validiums and Optimiums. For a detailed characterization of this category, please refer to [L2Beat](https://l2beat.com/).
- **L3 (or Layer 3)**: A blockchain that follow a similar structure to the L2 <> L1 structure but with an L2 underneath.
- **Origin Chain**: The chain where a user starts a cross-chain operation. In the context of assets, this could be where the user's funds or assets currently reside.
- **Destination Chain**: The chain where the desired side effects of a cross-chain operation are received or executed. In the context of assets, this could be where the user receives the funds.
- **Bridge**: A system enabling communication between chains. Others may use the term "Gateway".
- **User**: The end-user initiating a cross-chain operation.
- **Sender**: The account/contract initiating a message on the origin chain.
- **Receiver**: The account/contract receiving a message on the destination chain.
- **Relayer**: An entity facilitating message transmission between chains.


## Addresses

- **Chain Identifier**: A unique identifier for a blockchain used as part of the address formated. Do not confuse it with the Chain ID introduced in [EIP-155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md), even though it may be used as a reference in some cases.
- **Chain Name**: A human-readable identifier for a blockchain.

## Assets

- **`Deposit`**:  The act of sending tokens to a contract that serves as escrow to execute a cross-chain operation, such as minting or releasing tokens on destination chains. Others may refer to this action as `lock`.
- **`Mint` and `Burn`**: A pattern used in cross-chain operation where tokens are burned (supply decreased) on origin chain and minted (supply increased) on destination chain to simulate the effect of tokens "moving" between chains. Both token contracts should represent the same asset. These actions may also be referred to as `crosschainBurn`/`crosschainMint` or `bridgeBurn`/`bridgeMint`.

## Intents

- **`Fill`**: The act of a participant (called a `filler` or `solver`) executing a user's cross-chain intent on the destination chain(s) - for example, delivering tokens or executing a swap that the user requested.
- **`Leg`**: A portion of the user's intent that can be executed independently from the others. All legs must be executed for an intent to be considered fulfilled.
- **Settlement**: The process that happens after a successful fill where the system verifies the fill was executed correctly and handles the financial aspects —specifically, releasing the user's deposited funds to pay the filler for their service. The contract responsible for managing deposits and payments is called the **Settler** and may use a cross-chain verification method.

## Messaging

- **Message**: Information transmitted between chains.
  - **`send`**: The act of marking information on the origin chain to be transmitted to another chain(s). Functions in existing implementations are tipically named as `sendMessage` or others.
  - **`receive`**: Used to denote the act of receiving a message. Depending on the context, receiving a message may have its own flow or involve multiple steps. There are two ways a message can be received: through a _push_ model (where the underlying action is executed as soon as available to do so, enforced by some protocol rules and operators) or through a _pull_ model (where users or external operators self-complete the cross-chain operation). Functions in existing implementations are tipically named as `receiveMessage`, `relayMessage`, `validateMessage`, `executeMessage` or others.
- **`Metadata`**: Information about the message (sender, receiver, nonce, etc.)
  - **`ChainId`**: Referes the chain identifier. Existing implementations might refer to it as "domain" instead of "chain," or simply as the destination.
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
