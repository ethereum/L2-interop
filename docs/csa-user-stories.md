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
> - Error prevention if sending to a non-existent account (e.g. non-registered address given a name).
> - The chain-specific address should be clearly visualized.
> - Clear separation between source and destination chain.
> - Optional: Transfer method and costs are shown afterwards (Intents/Bridges).

### US2: Human-Readable Name Usage Across Chains

**As an end-user**:

I want to send assets using a human-readable Name.

**Rationale**: I want to avoid dealing with raw addresses.

> 📌
> **Acceptance Criteria:**
> -  The Human-Readable name to chain-specific addresses conversion is unambiguous; the wallet must display the checksum for safety.
> - Validates the existence of resolved addresses.
> - Chain information is shown.
> - Optional: shows full-resolved machine address.

### US3: Send Asset Error Prevention

**As an end-user**:

I want to be warned if I’m sending to potentially inexistent chain or address.

**Rationale**: I can avoid losing funds.

> 📌
> **Acceptance Criteria:**
> - All US2 acceptance criteria apply here.
> - If account is not found, warn about non-existent account.
> - If chain is not found, warn about a non-existent destination chain.
> - Optional: If chain identifier is intuitively distinguishable (e.g. ENS domains or CAIP-2 chain identifier mispellings), suggest correction and let the user decide whether to continue.

### US4: Sharing a Cross-Chain Address

**As an end-user:**

I want to share a complete, unambiguous machine address so that the recipient knows exactly which chain and address they are sending funds to.

**Rationale**: I can receive funds, preventing losses.

> 📌
> **Acceptance Criteria:**
> - Cross-chain address should be deterministically resolved into raw address + chain identifier.
> - No ambiguity is allowed.
> - Optional: US6 (see below) is triggered if chain is unknown.

### US5: Sharing a Name

**As an end-user:**

I want to share a human-readable name that can be resolved to one (or more) cross-chain addresses so that it’s more user-friendly and still prevents accidental misrouting.

**Rationale**: Sharing a name is more conveniente than a raw address.

> 📌
> **Acceptance Criteria:**
> - The human-readable name must be deterministically resolved into at least one cross-chain address.
> - If a name corresponds to multiple cross-chain addresses, default to one or show options.
> - Checksum should guard against accidental collisions.

### US6: Support for Unknown Chains

**As an end-user**:

I want to send assets to addresses on chains that my wallet hasn't explicitly integrated with.

**Rationale**: I shouldn't be limited to only sending assets to chains that my wallet already knows about.

> 📌
> **Acceptance Criteria:**
>  - Validates the existence of the chain through a trusted provider (e.g. bridging service in the short term, and through an on-chain list of chain configs in the long term).
> - Fetch chain config and confirms compatibility.
> - Validates the existence of resolved addresses.
> - Shows confirmation with chain information.
> - Clearly indicates this is a new/unknown chain to the user.
> - Optional: Allows user to save chain for future use.


## Developer Stories

### DS1: Parsing and Validating Addresses

**As an wallet/dApp developer**:

I want a straightforward API to parse a chain-specific address (e.g., "chain:0x123...") and validate its correctness,
so that I can accept user inputs and avoid sending transactions to invalid or ambiguous destinations.

**Rationale**: I can reduce the risk of ambiguity and then error.

> 📌
> **Acceptance Criteria:**
> - There is libraries that accepts a chain-specific string and outputs a result (e.g., {`chainIdentifier`, `rawAddress`}, which are based in ehtereum-lists/chains, ERC-7785 or others).
> - Distinguishable for cases where the same address format is used (e.g. in the case of raw addresses in EVM)
> - confirm chain identifiers against a known registry.
