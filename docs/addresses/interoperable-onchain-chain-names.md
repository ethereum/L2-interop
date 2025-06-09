# Issue l2.eth to Enable Chain-Specific Addresses

# Intro

Issuing `l2.eth` (details below) will give ERC-7828 “Interoperable Addresses” a permanent home inside ENS via a future custom wildcard resolver. This proposal has already been submitted as a temperature check to the ENS DAO and received a positive signal from the community.


# Context

The chain-specific-address initiative is driven by two complementary standards:

- **ERC-7930** introduces a canonical binary format for *(chain, address)* pairs.
- **ERC-7828** layers human-readable names on top of that binary format, allowing strings such as `alice.eth@rollupname.l2.eth#ABCD1234`.

For ERC-7828 to work in a fully decentralized way, chain names themselves should live inside a reliable registry, such as ENS. Registering **`l2.eth`** gives the DAO a neutral namespace under which every L2 can publish its canonical name (e.g., `optimism.l2.eth ⇆ chainId 10`).

# Approach

There are two equally safe ways the DAO can register `l2.eth`, both already used for routine ENS maintenance:

1. Use the DAO’s existing Controller wallet: This wallet already has permission to register `.eth` names, so it can claim `l2.eth` in a single transaction and hand it over to the multisig.
2. Temporarily give the Owner wallet controller rights: The Owner wallet briefly grants itself permission, registers `l2.eth`, transfers the name to the multisig, and then removes its own permission. This keeps the long-standing Controller wallet untouched but adds one extra step.

> **Upcoming option:** The [new controller](https://github.com/ensdomains/ens-contracts/pull/438) introduces a method called `registerWithoutPayment()` that can only be called by the DAO. It bypasses both the payment and the minimum character length requirement, enabling the DAO to register `l2.eth` directly once the update is live.

Regardless of which path we choose, the end-state is identical:

- A multisig chosen by the DAO becomes the sole owner of `l2.eth`.
- The multisig can later “wrap” the name for fine-grained permissions, set a wildcard resolver, and manage sub-names.

For this temperature check, we focused on plan tu submit a proposal that issues `l2.eth` and transfers it to a new multisig. The resolver design, record format and first batch of chains will come back as a separate executable proposal once the name is safely in DAO hands.

# Benefits

Taking ownership of l2.eth does three very concrete things for ENS and for every wallet that relies on its service.

First, it lets the two pending standards for Interoperable Addresses move from “nice idea” to “live feature.”  ERC-7930 already defines how to pack `(chain, address)` into bytes, and ERC-7828 defines how to turn that into text.  What they both lack is a guaranteed place to resolve the chain part.  `l2.eth` is that place.

Second, it pretends to replace the current off-chain JSON list of chain names with an on-chain registry that lives at `l2.eth` and is controlled by the DAO.  Right now, Optimism’s chain ID 10, Arbitrum One’s 42161, Base’s 8453, and hundreds of others sit in a GitHub repo that no contract can trust without manual vetting.  Once `optimism.l2.eth`, `arbitrum.l2.eth`, and `base.l2.eth` are recorded under a resolver the DAO governs, any smart contract or dApp can fetch that data directly from ENS.

Third, the wildcard resolver planned for `*.l2.eth` keeps scaling costs tiny.  The DAO registers the parent once; after that, adding a new roll-up means the multisig writes a single storage slot with the chain ID.

# Next Steps

1. Set up the multisig: Gather DAO feedback on signer selection and threshold (details to be finalized before the executable vote).
2. Executable proposal: Specify the exact on-chain calls for the chosen registration path and list the multisig signers as per the [executable-proposal-template](https://github.com/ensdomains/docs/blob/master/src/public/governance/executable-proposal-template.md). Awaiting [new controller](https://github.com/ensdomains/ens-contracts/pull/438) to be production ready and approved by ENS DAO.
3. Execution: Register `l2.eth`, then confirm that the multisig holds the name.
4. Resolver development: A wildcard resolver, specified in the [L2Resolver spec](specs/addresses/L2Resolver.md), that resolves ENS chain names to their corresponding EIP-7930 chain identifiers, and vice versa.
