# Message Passing

As the Ethereum landscape evolves, passing valid messages between different L2s, L3s, and LXs becomes fundamental for interconnecting user activity across chains, moving liquidity, and unlocking multi-domain applications, among other reasons. The desire for standardized APIs has motivated the creation of various proposals, from simple interfaces to those that aim to accommodate recent (thus common) rollup flows.

## Existing Messaging Standards

Standards are made to fit from any blockchain to potentially any blockchain. These standards are not concerned with the verification mechanism or off-chain methods of a message to verify and deliver.

### ERC-6170: Cross-Chain Messaging Interface

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

### ERC-7786: Cross-Chain Messaging Gateway

[ERC-7786](https://github.com/ethereum/ERCs/pull/673) also proposes a minimal interface to send (`sendMessage`) and receive (`executeMessage`) arbitrary messages. It containts extensible attributes that can be adapted to multiple bridging protocol models, as it is intented to be proof-agnostic. It leverages CAIP-10 for sender/receiver addresses, and introduces an optional post-proccessing step for any custom logic, as well as a explicit definitions of roles for sending and executing messages.

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

### ERC-7841: Cross-chain Message Format and Mailbox

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

### ERC-7854: Verification-independent Cross-Chain Messaging

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

## Rollup Messaging Protocols

Most rollups have built-in messaging interfaces between L1 and L2. 

### Linea

Linea deploys the corresponding messenger contracts on both L1 and L2. The relayers (called Postbots) listen for calls made on either side and deliver them to the destination. All cross-chain messages pass through this service, which provides replay protection.

The `sendMessage` function includes the value, recipient, fee to pay, and calldata, while `claimMessage` adds the fee recipient and nonce on top of those. Manual claiming is always available, especially used when no fee is set to be paid. For both flows, messages must first be verified against `MessageManager`. Additionally, for L2→L1 transfers, `claimMessageWithProof` is used, which includes a Merkle proof for final verification.

```mermaid
---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant OriginService as L1 or L2 MessageService (Chain A)
    participant Relayer as PostBots/Off-chain Relayer
    participant DestinationService as L2 or L1 MessageService (Chain B)
    participant Recipient

    User->>OriginService: sendMessage(to, fee, calldata)
    OriginService->>OriginService: emit MessageSent(from, to, fee, value, nonce, calldata, messageHash)
    Note over OriginService: Bridging logic <br/> (L1 deposit or L2 withdrawal initiated)
    OriginService->>Relayer: Off-chain bridging or PostBots flow
    Note over Relayer: Validation flow
    Relayer->>DestinationService: claimMessage(from, to, fee, value, feeRecipient, calldata, nonce) <br/> or claimMessageWithProof(...) 
    DestinationService->>DestinationService: emit MessageClaimed(messageHash)
    DestinationService->>Recipient: Deliver message contents
    DestinationService->>Relayer: receive fees (if it is allowed/seted)

```

### OP Stack

OP Stack deploys corresponding messenger contracts on both L1 and L2, as well as L2-to-L2 when interoperability is enabled at the protocol level. All of them include replay protection.

- **L1→L2 / L2→L1**: This follows the `CrossDomainMessenger` library. The `sendMessage` function includes the target, value (in ETH), message data, and gas limit. The `relayMessage` function, in turn, includes the same parameters while adding the sender and nocce.
    - L1→L2 operates on a push-based model, where sequencers process deposits when deemed safe, strictly following the order in which they were initiated.
    - L2→L1 follows a pull-based model, where withdrawals are in practice finalized asynchronously.

```mermaid
---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant OriginMessenger as L1 or L2 CrossDomainMessenger (Chain A)
    participant Relayer as Sequencer (L1 deposit) or Off-chain Relayer/User (L2 withdrawal)
    participant DestinationMessenger as L2 or L1 CrossDomainMessenger (Chain B)
    participant Recipient

    User->>OriginMessenger: sendMessage(to, minGasLimit, value, data)
    OriginMessenger->>OriginMessenger: emit SentMessage(target, sender, message, messageNonce, minGasLimit)
    OriginMessenger->>OriginMessenger: emit SentMessageExtension1(sender, value)
    Note over OriginMessenger: Bridging logic <br/> (L1 deposit or L2 withdrawal initiated)
    OriginMessenger->>Relayer: Off-chain proof of inclusion/finality reached
    Note over Relayer: Validation flow
    Relayer->>DestinationMessenger: relayMessage(nonce, sender, target, value, minGasLimit, message)
    DestinationMessenger->>DestinationMessenger: emit RelayedMessage(msgHash)
    DestinationMessenger->>Recipient: Execute message contents

```
- **L2→L2**: This has its own flow while still relying on the `sendMessage` and `relayMessage` concepts. Messages are sent directly from one L2 chain to another by specifying the destination chain ID, the target address, and the message payload. Once the message is validated via `CrossL2Inbox`, anyone can call `relayMessage` on the destination L2 by providing proof of the source event. This process remains asynchronous and can be finalized as soon as possible, depending on off-chain relayers and sequencers detecting and confirming initiated messages.

```mermaid
---
config:
  theme: dark
  fontSize: 48
---

sequenceDiagram
    participant User
    participant OriginMessenger as L2ToL2CrossDomainMessenger (Chain A)
    participant Relayer as Off-chain Relayer
    participant DestinationMessenger as L2ToL2CrossDomainMessenger (Chain B)
    participant Recipient

    User->>OriginMessenger: sendMessage(destination, target, message)
    OriginMessenger->>OriginMessenger: emit SentMessage(destination, target, nonce, sender, message)
    Note over OriginMessenger: Bridging logic on Chain A <br/> (writes event to be proven)
    OriginMessenger->>Relayer: Off-chain proof or inclusion flow
    Note over Relayer: Validation and cross-chain proof
    Relayer->>DestinationMessenger: relayMessage(id, sentMessage)
    DestinationMessenger->>DestinationMessenger: Validate & decode SentMessage Payload
    DestinationMessenger->>DestinationMessenger: emit RelayedMessage(source, nonce, messageHash)
    DestinationMessenger->>Recipient: Execute message contents

```

# Comparison

[WIP]
