# Message Passing Working Group

## L2 Interop Overall Goal: 
A user can bridge assets between any two Lns within 3 slots or less of the slower chain.

Things required to achieve this goal: 
* Message passing protocols to physically relay information from one chain to another. Different protocols have different tradeoffs and security properties, and different L2s may use different protocols. 
* Common, abstracted interface for devs to interact with messaging protocols.  Devs should not have to worry about the underlying protocol to bridge between chains.  Cross-chain applications should be able to be built on top of any messaging protocol that supports the interface. See [ERC-7841's](https://github.com/elliedavidson/ERCs/blob/d7c16c21f0a012ef391783322ae9b6f0eb0b56bc/ERCS/erc-7841.md) *Motivation* section for details about this goal. 

This working group focuses on the second point. 

## Current Working Group Status
We are currently working on 3 tasks: 

1. Agreeing that such an interface is useful. 
2. Agreeing whether goal 1 listed below is in scope for this working group. We will iterate through each goal below until we reach rough consensus among core contributors. 
3. Identifying core contributors to move this consensus forward quickly.  We will ask for feedback from the broader group periodically. 
4. Establishing commitments and timelines from core projects to implement the interface once it is finalized. 

### Timeline: 
* 2/17/2025: Core contibutors identified. 
* 2/24/2025: Agreement on goals has been reached among core contributors, broader group feedback welcome. 
* 3/2/2025: Agreement on properties of interface has been reached among core contributors, broader group feedback welcome. 
* 3/9/2025: Agreement on interface has been reached among all core contributors, broader group feedback welcome. 
* 3/16/2025: Publish updated or new ERC, reach out to devs explicitly to get feedback.  
* 3/30/2025: All changes to interface are finalized. Implementation begins


## Goals: 
### 1. Interface supports only the most common use cases: bridging and cross-chain function calls. --> We'll start with 90% of uses cases, aiming to make those APIs simple, and then we can add more complex use cases from there. 

Reasoning:  
* Supporting all use cases results in a more complex interface.  This may lead developers to use wrapper contracts to simplify the interface or to not adopt the interface at all, defeating the purpose of the interface.  
* Edge cases can be handled by the messaging protocol itself. Some edge use cases may depend on the specific messaging protocol used. 

#### 1a. Developers should not need to use wrapper contracts to interact with the interface. 
 
### 2. Interface should be VM-agnostic.  --> Agree

Reasoning: 
* Our goal is to connect the entire ecosystem of Lns, including the many Lns using different VMs. 
* We will need to balance this goal with goal 1; we shouldn't make the interface overly complex to achieve this goal. 

### 3. Interface should be proof-agnostic. --> Agree

Reasoning:  
* Many proof systems will be used in the underlying messaging protocols.  This interface should be compatible with any of those protocols. Otherwise, the interface does not achieve its primary goal abstracting the underlying messaging protocol from developers. 

### 4. Interface should be protocol-agnostic. --> Agree

Reasoning: 
* Many messaging protocols will be used.  This interface should be compatible with any of those protocols. 

### 5. Interface is compatible with any Ln↔Ln messaging. --> Agree

Reasoning: 
* Both L2 <> L2 messaging and L1 <> L2 messaging are core use cases. If we want to create a unified developer experience, the interface needs to support both using the same abstractions.  
* L3 <> L2 messaging is a less common use case, but should be supported. 
* Key Question: Should it be an explicit goal of this interface to support L1 <> L1 messaging? 

### 6. Making it happen soon 

Glacis --> TODO investigate
OZ adapts - 2-4 weeks / adapter --> 3 months for at least 3 adapters
 Axelar
 Wormhole
 Hyperlane
 LZ
 Chainlink

 Parallel track - public support from major protocols to advertise this

### 7. Support m / n messsage aggregation

### 8. Data transfer

### 9. Only narrow casting vs broadcasting to begin with
3 ways: push, pull, read

push - 
pull - 

Push vs Pull
A push protocol allows for the entire cross-chain transmission lifecycle, up to and including delivery to a receiver, to be arranged on the source chain without a user transaction on the destination chain. The protocol may rely on third party relayers to complete the lifecycle.

A pull protocol requires a user (or application) to complete the lifecycle with a transaction on the destination chain. The protocol can be said to require self-relaying.

push + local pull
push + execute 
don't need to worry about broadcast (for now

### 10. Gas is out of scope (for now) 

### 11. Interface does not enforce ordering 

### 12. Liveness
The protocol MUST guarantee Safety: A message is delivered at the destination if and only if it was sent at the source. The delivery process must ensure a message is only delivered once the sending transaction is finalized, and not delivered more than once. Note that there can be multiple messages with identical parameters that must be delivered separately.
The protocol MUST guarantee Liveness: A sent message is delivered at the destination eventually, assuming Liveness and censorship-resistance of the source and destination chains.




### Message Format

Message {
 payload: bytes
 source address: bytes
 source chain id: bytes
 destination adress: bytes
 destination chain: bytes
 nonce: uint32
}

packet_id - unique id, bytes

// destination addr 
fn send (payload, (destination address?), destination chain) -> packet id


// Won't support for now
fn recv (packet id) -> Message


    function executeMessage(
        string calldata sourceChain, // [CAIP-2] chain identifier
        string calldata sender, // [CAIP-10] account address
        bytes calldata payload,
        bytes[] calldata attributes
    ) external payable returns (bytes4);



  Gas

 source chain: 
  pay_gas(packet id) payable
  
  estimate_gas(gas_limit, destination id, ) -> cost on origin gas token

 // Didn't add enough gas in the beginning
  add_gas (




Questions: 
* Should we allow tx hash as unique id? 

## Miscellaneous Questions (to be addressed later)
* Should the interface include gas quoting?
* Should the interface be pull or push-based? 
* Should the interface use Ethereum specific addresses?  
* Should the interface defined specific message formats for specific use cases? 
* Should the interface use 7786-style attributes
* Why don't existing messaging designs cover our use cases? 
* How should cross-chain addresses be represented? 
