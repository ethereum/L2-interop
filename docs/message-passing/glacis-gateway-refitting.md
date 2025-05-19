## Glacis 7786 Adapter to Gateway Conversion
[Glacis](https://glacislabs.com) has worked in the bridge abstraction space for over a year, and have created adapters 
for their [Glacis Core](https://github.com/glacislabs/v1-core) product that abstract away message passing to a single 
interface. It should be feazible to rewrite only parts of these adapters to follow the 7786 standard.  

Overall, not much work has to be done, but fees remain to be a difficulty. Our adapters are not set up to separate 
the send and the relay steps. This might be possible with some protocols, like Axelar & Hyperlane.  

To begin, if Glacis wants to provide ERC7786 support, there are two non-exclusive paths:

1. Convert the Glacis router itself to support ERC7786
2. Convert each adapter to support ERC7786 independently

### Converting the Glacis Router
The router has the following interface:
```solidity
interface IGlacisRouter {
    /// @notice Routes the payload to the specific address on the destination chain
    /// using specified adapters
    /// @param chainId Destination chain (EIP-155)
    /// @param to Destination address on remote chain
    /// @param payload Payload to be routed
    /// @param adapters An array of adapters to be used for the routing (addresses 0x01-0xF8 for Glacis adapters 
    /// or specific addresses for custom adapters)
    /// @param fees Array of fees to be sent to each GMP & custom adapter for routing (must be same length as gmps)
    /// @param refundAddress An address for native currency to be sent to that are greater than fees charged. If it is a 
    /// contract it needs to support receive function, reverted otherwise
    /// @param retryable True if this message could pottentially be retried
    /// @return A tuple with a bytes32 messageId and a uint256 nonce
    function route(
        uint256 chainId,
        bytes32 to,
        bytes memory payload,
        address[] memory adapters,
        GlacisCommons.CrossChainGas[] memory fees,
        address refundAddress,
        bool retryable
    ) external payable returns (bytes32, uint256);

    /// @notice Retries routing the payload to the specific address on destination chain
    /// using specified GMPs and quorum
    /// @param chainId Destination chain (EIP-155)
    /// @param to Destination address on remote chain
    /// @param payload Payload to be routed
    /// @param adapters An array of adapters to be used for the routing (addresses 0x01-0xF8 for Glacis adapters 
    /// or specific addresses for custom adapters)
    /// @param fees Array of fees to be sent to each GMP & custom adapter for routing (must be same length as gmps)
    /// @param refundAddress An address for native currency to be sent to that are greater than fees charged. If it is a 
    /// contract it needs to support receive function, tx will revert otherwise
    /// @param messageId The messageId to retry
    /// @param nonce Unique value for this message routing
    /// @return A tuple with a bytes32 messageId and a uint256 nonce
    function routeRetry(
        uint256 chainId,
        bytes32 to,
        bytes memory payload,
        address[] memory adapters,
        GlacisCommons.CrossChainGas[] memory fees,
        address refundAddress,
        bytes32 messageId,
        uint256 nonce
    ) external payable returns (bytes32, uint256);
}

struct CrossChainGas {
    uint128 gasLimit;
    uint128 nativeCurrencyValue;
}
```

To make the `route` function compliant to `sendMessage`, there must be: 

1. A conversion of CAP-2/CAP-10 addresses to bytes format
2. Definition of attributes for the following features:
    1. Fees
    2. Quorum/Multisend/Adapters
    3. Retries
3. General attribute handling

### Glacis Adapters
If we were to only convert the Glacis adapters, we would be getting rid of all of Glacis’ special functionality: 
quorum, retries, etc. This might be the simplest conversion/use case for users of ERC7786.  

Each adapter has the following interface:  
```solidity
/// @title IGlacisAdapter
/// @notice An interface that defines the GMP modules (adapters) that the GlacisRouter interacts with.
interface IGlacisAdapter {
    /// Determines If a chain is available through this adapter
    /// @param chainId The Glacis chain ID
    /// @return True if the chain is available, false otherwise
    function chainIsAvailable(uint256 chainId) external view returns (bool);

    /// Sends a payload across chains to the destination router.
    /// @param chainId The Glacis chain ID
    /// @param refundAddress The address to refund excessive gas payment to
    /// @param payload The data packet to send across chains
    function sendMessage(
        uint256 chainId,
        address refundAddress,
        GlacisCommons.CrossChainGas memory incentives,
        bytes memory payload
    ) external payable;
}
```

To convert the Glacis adapter `sendMessage` to the new ERC7786 `sendMessage` function, the following must be added
to each adapter:  

1. A conversion of CAP-2/CAP-10 addresses to bytes format
2. General attribute handling

### Fees
It is important to note that Glacis does not have generic fee estimation, which should be added for developer ease. 
Currently, all developers must quote off-chain and would otherwise have to provide an overestimation of native currency
to pay for relayers. This is a poor and somewhat unpopular developer experience.  

If the current setup for the separation of relay and send is kept, as described [here](https://github.com/ethereum/ERCs/pull/981), 
then both routers and/or adapters must keep some sort of storage of the message parameters before sending them to the user. 
This will be a significant change from the current setup, and will cost more gas. Both the GlacisRouter and GlacisAdapters 
are not set up to separate the message and relay, as these two are inherently linked for many protocols.  

```solidity
function quoteRelay(
        string calldata destinationChain,
        string calldata receiver,
        bytes calldata payload,
        bytes[] calldata attributes
        uint256 gasLimit,
        string[] calldata refundReceivers
    ) external returns (uint256);

    function requestRelay(
        bytes32 outboxId,
        uint256 gasLimit,
        string[] calldata refundReceivers
    ) external payable;
```

When implementing, it is important to avoid “storing” a message for relaying as this costs additional gas, even if attempting
something with transient storage.
