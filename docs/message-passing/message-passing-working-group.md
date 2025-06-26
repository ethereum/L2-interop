# Message Passing Working Group

## Introduction

Interoperability among chains requires data from the source chain to be made available to the destination chain. This is often described as passing messages between chains.  Cross-chain communication protocols that handle passing these messages are **message-passing protocols**. A message-passing protocol defines how a messages is transmitted, routed, verified, etc. These protocols are the foundation for token bridging, cross-chain function calls, and more.

There exist many message-passing protocols today.  These protocols are deployed on different chains and each one has its own tradeoffs. Each protocol defines its own messaging interface for applications, and these interfaces are not compatible with each other, which restricts the interoperability they can provide.

This group's focus is to evaluate a standard messaging interface so applications interact with each message-passing protocol in the same way.

## Discussion Summaries

### What is the value of creating a new standard vs. adopting an existing one?

If our goal is maximum interoperability between chains in the most pragmatic way, then it could make sense to canonicalize an existing message-passing protocol's interface. Applications already using this protocol would automatically adopt the standard, and such an interface benefits from having time in production.

The interface standard must be credibly neutral, meaning: 
* The interface can function without the support of any core developement team. 
* The interface does not favor any specific project. 
* The interface does not favor any particular ecosystem or L2. 
* Projects are naturally incentivized to adopt the interface. 

 Canonicalizing an existing message-passing protocol interface will not move us towards towards the goal of seemless interoperabilty between chains because existing messaging protocols realistically will not adopt the protocol of another project.  And thus we will continue to have many different interfaces that fragment cross-chain applications. A new standard allows us to create a credibly neutral interface that doesn't favor any particular party.

Secondly, while current interfaces are powerful, they are not necessarily the *right* interface for applications long term.

### What is the value of a message passing interface standard?

Unified, simple developer experience - A standard allows all applications to interact with messaging protocols in the same way.  Many different messaging protocols will continue to exist, and applications may need to make use of multiple protocols to serve certain cross-chain routes. A standard interface allows applications to use whichever message protocol they need to without restructuring their contracts.

No vendor lock-in - Since applications interact with all messaging protocols through the same interface, applications can easily switch which messaging protocols they use without major contract changes.  

### Why will applications adopt this standard?

Adoption is always difficult with efforts like this. \<Insert relevant XKCD\>  However, we still feel it is the best way forward to achieve long term interoperability. Furthermore, the group has prioritized creating adapter contracts for major message passing protocols to ease adoption.

Adoption will take time.

### Is a message passing interface standard the most impactful work this group can do?

Interoperability solutions are needed now, and this standard doesn't have much benefit in the short term since it only gains benefit once a critical mass of applications adopt it.

This is why the L2 Interop group overall is primarily focused on intents and cross-chain addresses, since those can provide end users benefit in the very near term.  We recognize that this interface standard is not the highest priority of the group.

### Is message passing the right abstraction for developers?

There are several ways to model cross-chain communication: message passing, reading from another chain's state, etc. Ultimately all these abstractions represent the exact same idea, and message passing is the most common and intuitive abstraction for developers.

### Why not standardize the entire message-passing protocol instead?

There was discussion of standardizing an entire protocol, or at least standardizing the message verification mechanisms.  This may be something to tackle in the future, but it is a contentious topic. We can provide more benefit right now by standardizing the interface first.

### Group Tasks

- [ ] Agree on the properties a messaging interface should have.
- [x] Discuss the value of a new interface versus an existing one.
- [x] Discuss whether an interface is the most beneficial work for the group to do.
- [ ] Agree on a final standard.
- [ ] Implement adapters for major messaging protocols.
- [ ] Update ERC-7786 based on group discussions, if needed.

## Open Discussions

1. Should we allow a tx hash as unique id of a message?  Current protocols today use this hash as an id, but it adds complexity to the interface since the tx hash is not known at runtime.

## Resources

* [Comparison with current efforts](docs/message-passing/messaging-current-efforts.md)

## Help Wanted

* Adding more detail to the [Comparison with current efforts](docs/message-passing/messaging-current-efforts.md) file.
