# Interop Framework

This document tries to lay out a framework for understanding and discussing cross-chain interop protocols.

- **Data vs Value Transfer**
    - A data-transfer protocol allows information in general to be transmitted cross-chain.
    - A value-transfer protocol allows cross-chain transmission of fungible assets.
        - Cross-chain value transfer is a form of exchange.
- **Narrow vs Broad**
    - In a narrow protocol, specific actions on a source chain initiate cross-chain communication. For example, that a specific contract was invoked (gateway, outbox, bridge, etc.) or a specific kind of event was emitted.This subset of the chain state is transmitted cross-chain.
        - Each of these actions is a **message**, so narrow protocols are also known as **message-based** protocols.
    - In a broad protocol, an unrestricted set of source chain state can be the basis for cross-chain communication. For example, all logs or all storage from the source chain could be accessed on the destination chain.
- **Push vs Pull**
    - A push protocol allows for the entire cross-chain transmission lifecycle, up to and including delivery to a receiver, to be arranged on the source chain without a user transaction on the destination chain. The protocol may rely on **third party relayers** to complete the lifecycle.
    - A pull protocol requires a user (or application) to complete the lifecycle with a transaction on the destination chain. The protocol can be said to require **self-relaying**.

Within narrow message-based protocols:

- **Targetted vs Broadcast**
    - A targetted message inherently specifies a receiver on a destination chain.
        - Some protocols allow atomic bundles of messages, each specifying a different receiver.
    - A broadcast message doesn't specify a receiver. It may specify a destination chain.
- **Active vs Passive**
    - This is merely about the on-chain APIs and is independent of the push or pull nature of the protocol.
    - An active protocol concludes delivery of the message by executing a call (`CALL`) to the receiver on the destination chain from a protocol contract. This call can be done in one of two ways:
        - **Explicit vs Implicit**
            - An explicit delivery invokes a special-purpose function on the receiver, e.g., `receiveMessage(address sender, bytes payload)`.
            - An implicit delivery invokes the raw message payload on the receiver. Parameters like the sender of the message are made available by the protocol, e.g., as public getter on a protocol contract.
    - A passive protocol allows a receiver to check the validity of a message against a protocol contract after delivery.
