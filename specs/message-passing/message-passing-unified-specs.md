# Cross Messaging Unification

*This document aims to foster collaboration on existing standard specifications, without prescribing whether the best path forward involves evolving current ERCs or considering complementary directions.*

## Context

We propose a “unified” cross‑chain‑messaging interface that starts from [ERC‑7786](https://github.com/ethereum/ERCs/blob/448902e83117f175b4f682fb2d60545709df8cef/ERCS/erc-7786.md) and borrows useful ideas from [ERC‑6170](https://github.com/ethereum/ERCs/blob/448902e83117f175b4f682fb2d60545709df8cef/ERCS/erc-6170.md), [ERC‑7854](https://github.com/ethereum/ERCs/pull/817), and the [Interop Working Group discussions](https://github.com/ethereum/L2-interop/pull/13).

The primary audience is application developers, who need a predictable, ergonomic API rather than a patchwork of bridge‑specific contracts.

This draft keeps 7786’s proven core but adds just enough opinionated structure—notably binary interoperable addresses and optional hooks—to make implementations consistent and developer‑friendly.

👉 **This unified version is open for discussion and collaboration, and we welcome feedback and contributions from the community.**

---

## Specification

A complete specification primarily consists of four logical building blocks:

1. **Gateway:** The canonical entry point API for the cross‑chain message flow. On the origin chain, it lets contracts send messages, emits a tracking event, and (optionally) invokes hook calls. On the destination chain, each implementation provides its own Destination Gateway, which might be responsible for verifying the proof and delivering the message to the recipient.
2. **Recipient Interface**: A minimal contract API that must be implemented by any contract wishing to receive cross-chain messages.
3. **Message**: The cross-chain data structure containing sender, recipient, payload, and implementation-specific extra data required by the underlying bridge to handle delivery or verification.
4. **Hooks**: Opt‑in extension points that let developers run arbitrary logic before or after the sending, without polluting the Gateway surface.

### Message Field Encoding

A cross-chain message includes sender and recipient addresses, a payload, extra data, and optional hooks.

### Sender and Recipient

Binary interoperable addresses ([ERC‑XXXX](https://github.com/ethereum/ERCs/pull/1002)), containing the account address and chain identifier (from CAIP‑2).

### Payload and Extra Data

The payload is an opaque `bytes` value. The Extra Data encodes optional logic or metadata (such as low-level calls) that the underlying messaging protocol should process during delivery.

### Hooks

A hook is any contract logic that the application developer wants to execute after a message is sent from the origin chain. They make it possible for developers to include any external logic apart from the underlying message protocol entry point when sending a message. Hooks are encapsulated under a struct that contains the hook payload, and the local address of the hook, and value. Hooks are never part of the message delivered to the recipient; they only affect the execution environment on the origin chain.

### Interfaces

Smart contracts interact with a `Gateway` to send and receive messages. Using hooks (`HookData`) is optional.

```solidity
interface IGateway {
  // Optional Hook data struct
  struct HookData {
    bytes hookPayload;   // Low-level call data: selector and parameters for the hook
     ILocalAddress hook;  // Local origin chain address
    uint256 value;       // Optional, amount of native token forwarded to the hook 
  }
 
  // MessageSent event
  event MessageSent(
    bytes32 outboxId,  // Unique outbox ID (may be 0 if the gateway doesn’t track)
    bytes sender,      // Binary Interoperable Address
    bytes recipient,   // Binary Interoperable Address
    bytes payload,     // Message content
    bytes extraData,   // Optional encoded gateway metadata (leave empty if unused)
    HookData hookData  // Optional Hook data 
  );
  
  // sendMessage function
  function sendMessage(
      bytes calldata recipient,                   // Binary Interoperable Address 
      bytes calldata payload,                     // Message content
      bytes calldata extraData,                   // Optional encoded gateway metadata (leave empty if unused)
      HookData calldata hookData                  // Optional Hook data
  ) external payable returns (bytes32 outboxId);  // Unique ID (may be 0 if the gateway doesn’t track)
}
```

The destination `IGateway` is out of scope in this specification. Its responsibility is to deliver the message to the recipient, which must implement the `IMessageRecipient` interface.

```solidity
interface IMessageRecipient {
    function receiveMessage(
        bytes32 messageId,  // Unique ID supplied by the destination gateway
        bytes sender,       // Binary Interoperable Address representation
        bytes payload       // Message content
    ) external payable;
}
```

# Flows

A first glance on the existing and proposed flow.

## (As Reference) ERC-7786 flow:

```mermaid
sequenceDiagram
    participant User
    participant GatewaySource as IERC7786GatewaySource (Chain A)
    participant Relayer as Relayer / Post‑Processing
    participant GatewayDestination as Destination Gateway (Chain B)
    participant Receiver as IERC7786Receiver

    User->>GatewaySource: sendMessage(destinationChain, receiver, payload, attributes)
    GatewaySource->>GatewaySource: emit MessagePosted(outboxId, sender, receiver, payload, value, attributes)
    Note over Relayer: The post-processing or <br/> relaying step triggers eventually
    Relayer->>GatewayDestination: preReceive(parameters) "This is ONLY illustrative"
    GatewayDestination->>GatewayDestination: verify proof & parse message
    GatewayDestination->>Receiver: executeMessage(messageId, sourceChain, sender, payload, attributes)
    Receiver-->>GatewayDestination: returns IERC7786Receiver.executeMessage.selector
```

## (As Reference) ERC-7854 flow:

```mermaid
sequenceDiagram
    participant User
    participant MailboxA as Mailbox (Chain A)
    participant Relayer as Off-Chain Relayer
    participant MailboxB as Mailbox (Chain B)
    participant Recipient as IMessageRecipient

    User->>MailboxA: dispatch(destinationDomain, recipientAddress, body, customHookMetadata...)
    MailboxA->>MailboxA: emit Dispatch (sender, destination, recipient, message)
    MailboxA->>MailboxA: emit DispatchId (messageId)
    MailboxA->>Relayer: Off-chain bridging flow

    Relayer->>MailboxB: process(_metadata, _message)
    MailboxB->>MailboxB: verify() with ISM
    MailboxB->>Recipient: handle(origin, sender, _message) 
    MailboxB->>MailboxB: emit Process (origin, sender, recipient)
    MailboxB->>MailboxB: emit ProcessId (messageId)

```

## Proposed flow:

As well as above, this flow shows how it could work on top of the OP Stack message passing protocol.

```mermaid
sequenceDiagram
    participant User
    participant GatewayA as Gateway (Chain A)
    participant HookA as Hook (Chain A)
    participant Relayer as Off-Chain Relayer
    participant GatewayB as Gateway (Chain B)
    participant Recipient as IMessageRecipient

    User->>GatewayA: sendMessage(recipient, payload, extraData, hookData)
    GatewayA->>HookA: hookData
    GatewayA->>GatewayA: MessageSent (messageId, sender, recipient, payload)
    GatewayA-->>Relayer: Off-chain bridging flow

    Note over GatewayB: using deliverMessage as example method, destination Gateway is out of scope
    Relayer-->>GatewayB: deliverMessage(metadata, message)
    GatewayB->>GatewayB: runVerifications()
    GatewayB->>Recipient: receiveMessage(messageId, sender, payload)

```

```mermaid
graph TD
 Relayer
 
 subgraph Origin Chain 
	 GatewayE[Gateway]
	 Sender --"sendMessage(recipient, payload, extraData, hookData)"-->GatewayE
	 GatewayE --"run HookData()"-->HookContract
 end
 

 subgraph Destination Chain
	 _GatewayR_[Gateway]
	 Recipient
	 	 
	 _GatewayR_ --"_run verifications(☁️)_"--> _GatewayR_
	 _GatewayR_ --"receiveMessage(messageId, sender, payload)"--> Recipient
 end
 Relayer --"_deliverMessage(messageId, sender, payload)_"--> _GatewayR_

```

## Rationale and Discussions

This design is an incremental evolution based on ERC-7786 and other related standards, aligning progressively with what is considered best practice.

**What stays the same**

- **Unchanged core API:** send and receive messages behave exactly as in ERC-7786. Adapters would remain the same way.
- **Safety / Liveness expectations**: All guarantees listed in ERC-7786 are inherited unchanged.
- **Event-driven tracking**: an `outboxId` is still emitted on the origin chain, and a `messageId` is provided by the relaying/deliver step on the destination chain.

**What is added**

- **Hooks (optional):** A  `HookData` struct lets a project plug in fee payment, logging, batched calls, callbacks or any other external logic without bloating the base call.
- **Future‑proof addressing:** Binary interoperable addresses (ERC-XXXX); 7786 plans to adopt the same format, so this is forward‑compatible.
- **Clear separation of concerns**: `extraData` tweaks behavior inside the gateway, while `HookData` runs code outside the gateway.

**Hooks vs. Attributes**

ERC-7786 introduces attributes (key/value blobs interpreted by the gateway). Hooks fill the same niche but with two advantages:

1. Arbitrary Logic: A hook is external contract logic that the message sender wants to execute. Hooks remove the limitation of relying solely on the selectors implemented by the Gateway, enabling developers to execute arbitrary functions the Gateway didn’t originally anticipate.
2. No bloat for minimal gateways: A bare-bones gateway doesn’t need to parse or store attribute blobs. If a project wants richer behavior, it can deploy its own hook contract.
3. Isolate risk: if a hook misbehaves, only that hook’s callers are affected, the core gateway remains simple and auditable.

`ExtraData` and `HookData`

- **`extraData`**: small blobs consumed *inside* the gateway (relevant for adapters e.g., define the gas limit or a caller).
- **`hookData`**: full contract calls executed *outside* the gateway (e.g., pay a relayer, emit custom analytics, trigger another bridge).

# Appendix

## High-level Delta

| **Change** | ERC-7786 | ERC-7854 | Unification | **Why?** |
| --- | --- | --- | --- | --- |
| **Renames** | `Gateway`, `sendMessage`, `executeMessage` | `Mailbox`, `dispatch`, `process`, `handle` | `Gateway`, `sendMessage`, `receiveMessage` | Fresh terminology around more common conventions. |
| **Message Struct** | No explicit struct. Parameters are passed directly: `destinationChain` (or `sourceChain`), `receiver` (or `sender`), `payload`, `attributes`. | Contains: `version`,  `nonce`,  `origin`,  `sender`,  `destination`, `recipient`, `body`. | No explicit struct. `Parameters` are passed directly: `sender` (or  `recipient`) `payload`, `extraData`, `HookData`, and `messageId` in destination. | Simpler surface; extra inputs stays in `extraData` or `HookData` instead of bloating the envelope. |
| **Chain identifier and addresses** | `string` based; follows CAIP rules. There is an intention to support binary representations and interoperable addresses. | Uses `uint32` and `bytes32` without defined standards. | Under development binary interoperable address, in `bytes`. | Chain-agnostic and accommodate the address efforts in the interop group. |
| Message identifiers | Gateway generates an `outboxId` in origin and `messageId` to be used in the destination. | Defined in the struct with `nonce`. | `outboxId` (origin event) and `messageId` (destination callback). | Leaves identifier format to the transport layer while giving explorers two stable anchors. |
| **Events** | Only `MessagePosted`. | Not defined in the standard. It's worth to add Hyperlane’s implementation introduces `Dispatch`, `DispatchId`, `ProcessId`, and `Process` events. | Defined as `MessageSent`  in the origin Gateway. | One standardised event makes indexing uniform and keeps the destination side stateless. |
| **Post sending messages / Hooks Declaration** | Under `attributes`, as an optional feature. | Part of `dispatch` as `calldata` with appointed interface. | Encapsulated under `Hookdata` struct, (`hook`, `hookPayload` and `value`). Executed outside the gateway. | Hooks give full flexibility without bloating the core API; gateways that don’t need them can ignore the field. |
| Verification step | Abstracted from the interface (out of scope) | Defined as an ISM. | Abstracted from the interface (out of scope) | Keeps this ERC small; verification frameworks evolve quickly and deserve their own track. |
