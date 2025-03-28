# Message Passing

As the Ethereum landscape evolves, passing valid messages between different L2s, L3s, and LXs becomes fundamental for interconnecting user activity across chains, moving liquidity, and unlocking multi-domain applications, among other reasons. The desire for standardized APIs has motivated the creation of various proposals, from simple interfaces to those that aim to accommodate recent (thus common) rollup flows.

## ERC-6170: Cross-Chain Messaging Interface

[ERC-6170](https://github.com/ethereum/ERCs/blob/master/ERCS/erc-6170.md/) defines a basic, minimal interface to send (`sendMessage`) and receive (`receiveMessage`) arbitrary messages. This is considered one of the most simple standards.

```mermaid
---
config:
  theme: dark
  fontSize: 48 
---

sequenceDiagram
    participant User
    participant ContractA as IERC6170 (Chain A)
    participant Relayer as Off-Chain Relayer
    participant ContractB as IERC6170 (Chain B)
    participant Recipient

    User->>ContractA: sendMessage(chainId_, receiver_, message_, data_)
    ContractA->>ContractA: emit MessageSent(to, toChainId, message, extraData)
    Note over ContractA: Handling bridging <br/> logic off-chain
    ContractA->>Relayer: Off-chain bridging flow
    Relayer->>ContractB: receiveMessage(chainId_, sender_, message_, data_)
    Note over Relayer: chainId_ MUST be "A"
    ContractB->>ContractB: emit MessageReceived(from, fromChainId, message)
    ContractB->>Recipient: Deliver message contents
```

## ERC-7786: Cross-Chain Messaging Gateway

[ERC-7786](https://github.com/ethereum/ERCs/pull/673) also proposes a minimal interface to send (`sendMessage`) and receive (`executeMessage`) arbitrary messages. It contains extensible attributes that can be adapted to multiple bridging protocol models, as it is intented to be proof-agnostic. It leverages CAIP-10 for sender/receiver addresses, and introduces an optional post-proccessing step for any custom logic, as well as a explicit definitions of roles for sending and executing messages.

```mermaid
---
config:
  theme: dark
  fontSize: 48 
---

sequenceDiagram
    participant User
    participant GatewaySource as IERC7786GatewaySource (Chain A)
    participant Outbox as "Outbox Tracking"
    participant RelayProcess as Off-Chain Relayer
    participant GatewayDestination as IERC7786Receiver (Chain B)
    participant Receiver as Recipient

    User->>GatewaySource: sendMessage(destinationChain, receiver, payload, attributes)
    Note over User: MUST accomplish CAIP-10 and CAIP-2
    GatewaySource->>GatewaySource: emit MessagePosted(outboxId, sender, receiver, payload, value, attributes)
    GatewaySource->>Outbox: Update outbox for msg tracking

    Note over RelayProcess: The post-processing or <br/> relaying step triggers eventually

    RelayProcess->>GatewayDestination: executeMessage(sourceChain, sender, payload, attributes)
    GatewayDestination->>GatewayDestination: Validation + parse message
    GatewayDestination->>Receiver: Calls executeMessage(...) <br/> on receiver contract
    Receiver-->GatewayDestination: returns IERC7786Receiver.executeMessage.selector
```

## ERC-7841: Cross-chain Message Format and Mailbox

[ERC-7841](https://github.com/ethereum/ERCs/pull/766), similarly to ERC-7786, defines a standard message format (metadata + payload). Also conceives the existence of the Mailbox contract for storing/retrieving messages, allowing either push- or pull-based bridging (see `execute` implementation example), so the message might sit "in the mailbox" until bridging is proven or invoked.

```mermaid

---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant MailboxA as Mailbox (Chain A)
    participant Relayer as Off-Chain Relayer
    participant MailboxB as Mailbox (Chain B)
    participant Recipient

    User->>MailboxA: send(Metadata, payload)
    Note over User: "e.g. send(destChainId, destAddress...)"
    Note over MailboxA: Handling bridging <br/> logic off-chain
    MailboxA->>Relayer: Off-chain bridging flow

    Relayer->>MailboxB: populateInbox([Message], aux)
    MailboxB->>Recipient: recv(Metadata) → returns payload
```

## ERC-7854: Verification-independent Cross-Chain Messaging

[ERC-7854](https://github.com/ethereum/ERCs/pull/817) defines a minimal interface that decouples the messaging functions from the underlying verification method. Introduces a "Interchain Security Modules" API as a way to isolate the verification of messages.

```mermaid
---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant MailboxA as Mailbox (Chain A)
    participant Relayer as Off-Chain Relayer
    participant MailboxB as Mailbox (Chain B)
    participant Recipient as IMessageRecipient

    User->>MailboxA: dispatch(destinationDomain, recipientAddress, body, customHookMetadata...)
    Note over MailboxA: Handling bridging <br/> logic off-chain
    MailboxA->>Relayer: Off-chain bridging flow

    Relayer->>MailboxB: process(_metadata, _message)
    MailboxB->>MailboxB: verify() with ISM
    MailboxB->>Recipient: handle(origin, sender, _message)
```

# Comparison

[WIP]
