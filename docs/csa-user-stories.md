# Chain-Specific Addresses: Users-Stories

This document outlines the user stories as a goal for implementing chain-specific addresses. These stories capture key use cases and acceptance criteria for both end-users and developers, and do not express an opinion on specific implementations.

**End-users** refers to those who consume the production release, and **developers** are those who integrate or build on top of it.

## End-User Stories

### US1: Basic Send to Another Chain

**As an end-user**:

I want to send asset to an address on a different chain.

**Rationale**: Since I noticed my assets are a different chain where I know the address where I should transfer any value, I can transfer it safely.

> 📌
> **Acceptance Criteria:**
> - Wallet detects and shows the destination chain when address is provided.
> - Error prevention if sending to a non-existent account.
> - The chain-specific address should be clearly visualized.
> - Clear separation between source and destination chain.
> - Optional: Transfer method and costs are showed afterwards (Intents/Bridges).

### US2: Human-Readable Addresses Usage Across Chains

**As an end-user**:

I want to send assets using an human-readable address.

**Rationale**: I can avoid dealing with raw addresses.

> 📌
> **Acceptance Criteria:**
> - The Human-Readable address is resolved deterministically into a correct chain-specific addresses.
> - Validates the existence of resolved addresses.
> - Chain information is shown.
> - Optional: shows resolved address.

### US3: Send Asset Error Prevention

**As an end-user**:

I want to be warned if I’m sending to potentially incorrect chain or address.

**Rationale**: I can avoid losing funds.

> 📌
> **Acceptance Criteria:**
> - Validates the existence of resolved addresses.
> - Shows confirmation with chain information.
> - Warn about a non-existent account on the destination chain.
> - Prevents common user mistakes.
> - Optional: shows the resolved address.

## Developer Stories


