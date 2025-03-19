# Chain-Specific Addresses: Users-Stories

This document outlines the user stories as a goal for implementing chain-specific addresses. These stories capture key use cases and acceptance criteria for both end-users and developers, and do not express an opinion on specific implementations.

**End-users** refers to those who consume the production release, and **developers** are those who integrate or build on top of it.

## End-User Stories

### US1: Basic Send to Another Chain

**As an end-user**:

I want to send asset to any address, regardless of which chain it is on.

**Rationale**: I should be able to send assets as easily as if everything was on a single chain.

> 📌
> **Acceptance Criteria:**
> - Wallet detects and shows the destination chain when address is provided.
> - Error prevention when sending to a non-existent account, like an unregistered address assigned to a name.
> - The chain-specific address should be clearly visualized.
> - Clear separation between source and destination chain.
> - Optional: Transfer method and costs are shown afterwards (Intents/Bridges).

### US2: Human-Readable Name Usage Across Chains

**As an end-user**:

I want to send assets using a human-readable name.

**Rationale**: I want to avoid dealing with raw addresses.

> 📌
> **Acceptance Criteria:**
> - The human-readable name to chain-specific addresses conversion is unambiguous; the wallet must display the checksum for safety.
> - Validates the existence of resolved addresses.
> - Chain information is shown.
> - Optional: shows the full-resolved machine address.

### US3: Send Asset Error Prevention

**As an end-user**:

I want to be warned if I’m sending to potentially inexistent chain or address.

**Rationale**: I want to avoid losing funds.

> 📌
> **Acceptance Criteria:**
> - All US2 acceptance criteria apply here.
> - If the account is not found, warn about non-existent account.
> - If the chain is not found, warn about a non-existent destination chain.
> - Optional: If the chain identifier is intuitively distinguishable (e.g., ENS domains or CAIP-2 chain identifier misspellings), suggest a correction and let the user decide whether to continue.

### US4: Sharing a Cross-Chain Address

**As an end-user:**

I want to share a complete, unambiguous machine address so the recipient knows exactly which chain and address they are sending funds to.

**Rationale**: I can receive funds, preventing losses.

> 📌
> **Acceptance Criteria:**
> - Cross-chain address should be deterministically resolved into raw address + chain identifier.
> - No ambiguity is allowed.
> - Optional: US6 (see below) is triggered if the chain is unknown.

### US5: Sharing a Name

**As an end-user:**

I want to share a human-readable name that can be resolved to one (or more) cross-chain addresses making it more user-friendly while still preventing accidental misrouting.

**Rationale**: Sharing a name is more convenient than a raw address.

> 📌
> **Acceptance Criteria:**
> - The human-readable name must be deterministically resolved into at least one cross-chain address.
> - If a name corresponds to multiple cross-chain addresses, default to one or show options.
> - The checksum should prevent accidental collisions.

### US6: Support for Unknown Chains

**As an end-user**:

I want to send assets to addresses on chains that my wallet hasn't explicitly integrated.

**Rationale**: I shouldn't be limited to only sending assets to chains that my wallet recognizes.

> 📌
> **Acceptance Criteria:**
>  - Validates the existence of the chain through a trusted provider (e.g. bridging service in the short term, and through an on-chain list of chain configs in the long term).
> - Fetches the chain config and confirms compatibility.
> - Validates the existence of resolved addresses.
> - Shows confirmation with chain information.
> - Clearly indicates if it is a new/unknown chain to the user.
> - Optional: Allows the user to save the chain for future use.


## Developer Stories

### DS1: Parsing and Validating Addresses

**As an wallet/dApp developer**:

I want a simple API to parse a chain-specific address (e.g., 'chain:0x123...') and validate it, so I can accept user input while preventing transactions to invalid or ambiguous destinations.

**Rationale**: I want to reduce the risk of ambiguity and errors.

> 📌
> **Acceptance Criteria:**
> - Some libraries accept a chain-specific string and return a result like {`chainIdentifier`, `rawAddress`}, based on sources such as ethereum-lists/chains, ERC-7785, or others.
> - Distinguishable in cases where the same address format is used, such as raw addresses in EVM.
> - Verify chain identifiers against a known registry.
