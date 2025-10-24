# 7683 (Open Intents Framework) - Goals & Issues

## Context
There’s general consensus around pushing 7683 as an generic intents standard. In this document, we’ll attempt to clarify some of the goals of the effort and questions around the interface. Feel free to add your team’s POV to the doc!

## Goals
Are we all in agreement on the high level goals of the open intents framework (OIF)?

* Should the generic solver implementation in the OIF work out of the box for any 7683 intents protocol (w/ no additional changes required)?
* Should the Base7683 implementation work out of the box for any intents protocol to make them 7683 compliant?
* Is the end goal to have a single 7683 contract impl deployed to each chain that protocols can plug into or is it to have each intent protocol inherit from the Base7683 contract and create/deploy their own implementations?
Is it worth aiming to arrive at a version that could eventually be adopted as a precompile to standardize interactions? 

## Issues

* Deposits.
  * Across’s implementation:
    * ERC7683OrderDepositor exposes an internal `_callDeposit` method that lets the underlying protocol define the deposit mechanics.
	* ERC7683OrderDepositorExternal translates the `_callDeposit()` call into an Across’s own deposit impl in their “spoke pool”.
	* `_callDeposit` API exposes Across specific params such as exclusivityPeriod that are not defined in the 7683 spec. 
  * OIF implementation:
    * Open methods implement the deposit logic. This means that protocol specific deposit logic is currently not supported.
  * Resource lock impls generally require deposits to be made within the resource lock itself. This makes them incompatible with OIF’s current impl. 
* Order data encoding.
  * Across’s impl has many Across specific fields in their order data. The OIF impl does leave _fillOrder up to the inheriting contract to impl. 
  * This means that we’ll need a separate solver impl per intents protocol. Is this okay? Do we want to standardize some of the order data fields?

# Catalyst @ LI.FI

## General Incompatibilities

* ERC-7683 puts a lot of focus on the open interface. This makes sense, since the initial version of ERC-7683 was conceived before resource locks. However, with resource locks, it presents a challenge as orders don’t always start with on-chain calls anymore. 
* The ERC-7683 fill function is expensive since it works with memory, and not calldata, because of the transparent bytes provided.

## Our Recommendations

* Add address user, uint256 nonce to OnchainCrossChainOrder.
  * Goal: To clear up confusion regarding who the depositor is. (Tokens should still be collected from msg.sender)
* Convert the open event into event Open(bytes32 indexed orderId, bytes resolveContext) and add resolve functions taking the original order and resolveContext to accurately produce the ResolvedCrossChainOrder from an off-chain view call.
  * Goal: To reduce the cost of emitting the Open event.
* Make function open(...), openFor(...) optional.
  * Goal: Make it explicit that open should only be used for depositing.
* Make function validate(...) view returns (uint32 fillDeadline) function to easily validate whether an order is valid.
  * Goal: Provide an explicit way for pure off-chain orders to be validated

Additionally, the specification should  contain a description of how to integrate the lock flow: Outputs first, inputs second.

## Open Intents Framework (Most severe to least severe)
* Does not support resource locks.
* Does not support deposit on behalf of. ERC-7683 flaw though. (aggregation breaker)
* You store bytes on-chain; Can't support complex order types as it would balloon the cost.
* Uniswap Tribunal is better designed.
* Why are nonces duplicated on orderData. Just standardize it in ERC-7683.
* It is very expensive.

## Recommendation
* **Standardized components** such as each part of the leg can be mixed up instead of requiring inheritance between order types, messaging protocols, settlement systems etc.
  * Goal: Allow composability between elements rather than power scaling.
* Focus on machine readability instead of human readability.
  * Goal: Make the framework efficient by default.
