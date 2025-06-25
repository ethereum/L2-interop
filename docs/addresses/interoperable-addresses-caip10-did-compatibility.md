# ERC-7930 & ERC-7828: CAIP-10 and DID Compatibility

This document analyzes the compatibility between ERC-7930 (binary interoperable addresses) and ERC-7828 (human readable addresses) with existing standards CAIP-10 and W3C Decentralized Identifiers (DID). While these standards share similar goals, they have compatibility challenges to consider.

## TL;DR

**ERC-7930 & CAIP-10 Compatibility:**

ERC-7828/7930 is fully compatible with CAIP-10, as it uses the same core fields: `address` and `chainId`. While ERC-7930 uses `@` as a separator for readability (`address@chain`), this can be safely percent-encoded as `%40` to comply with CAIP-10’s character restrictions. Since the data structure is equivalent, conversion between formats is trivial, making ERC-7930 fully interoperable with CAIP-10.

**ERC-7928/7930 & DID Compatibility:**

Although ERC-7828/7930 lack a formal DID method (analogous to not having a resolver in ENS), they are compatible with the W3C DID standard. They ensure globally unique, permissionless, and verifiable identifiers via `address + chainId`, satisfying the core DID requirements. Once a formal DID method is specified, ERC-7930 would be fully DID-compliant.

## CAIP-10 Compatibility

ERC-7930 builds upon the foundation of CAIP-10's string based account identifier (`<chain_id>:<address>`) by introducing a more robust, dual format system. It defines a compact, length prefixed **binary envelope** designed for on chain efficiency, capable of carrying any `(chain ID, address)` pair regardless of byte length or cryptographic system. This is paired with a human readable string, `address@chain`, which contains the exact same information as CAIP-10. Because the core data is identical, conversion between the two formats is a trivial process of reordering the elements and swapping the separator (`@` ↔ `:`). This ensures that any ERC-7930 address can be seamlessly expressed as a valid CAIP-10 ID and viceversa, while also gaining ERC-7930's critical advantages: a canonical binary format for smart contracts, guaranteed future extensibility and namespace flexibility.

### 1. The Chain ID Length Problem

The most significant point of incompatibility arises from the constraints on chain identifier length. CAIP-10 leverages **CAIP-2**, which formally limits the `reference` portion of a chain ID to a **maximum of 32 characters** composed of `[a-zA-Z0-9]`. This design was based on practical assumptions from when the standard was written, for example, many early chain profiles, especially for UTXO based chains like Bitcoin, expected identifiers no longer than 16 bytes (32 hex characters). Consequently, many applications built around CAIP-10 assumed reasonably short chain IDs.

In contrast, ERC-7930 was explicitly designed to be future-proof and **does not impose any limit on chain ID size**, allowing it to natively support full 256-bit (32 byte) identifiers in its binary format. This foresight is critical for emerging standards like **ERC-7785**, which proposes using a `keccak256` hash of a chain's name to create a collision resistant ID. Such a hash is 32 bytes long, and its hexadecimal string representation is 64 characters long (plus `0x`).

This creates a direct conflict: a 64-character hash string cannot be stored in a 32-character field. In CAIP-2 terms, there’s no clear or standard way to truncate a 32 byte hash into a 32-character reference without breaking the specification and losing the guarantee of uniqueness**.** ERC-7930 gracefully handles this, while CAIP-10's format cannot.

### **Example 1: The CAIP-2 Limit**

```
// CAIP-2 Chain Reference: Maximum 32 characters.
// A real-world example is Bitcoin, whose `reference` is a 32-character hex string.
bip122:000000000019d6689c085ae165831e93:1A1z...
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        Exactly 32 characters, matching the limit.

```

### **Example 2: The ERC-7785 Problem**

```
// ERC-7785 uses keccak256 for chain IDs. Hashes are 32 bytes long.
bytes32 chainIdHash = keccak256("arbitrum-nova-zk-rollup-2025");

// The hexadecimal string representation of a 32-byte hash is 64 characters long.
// Result: 0x59a35e1b33e2842d416b2e1a3a6971ce181d2da2834655f2b322a36b33b23e1b
//         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//         64 characters - Far exceeds CAIP-10's 32-character limit.

```

### **Example 3: The ERC-7930 Solution**

```
// ERC-7930's binary format natively supports full 256-bit (32-byte) chain IDs.
// Using the same hash from the problem above, the binary payload would be structured as:

0x000100002059a35e1b33e2842d416b2e1a3a6971ce181d2da2834655f2b322a36b33b23e1b...
// │  │   │ └─ ChainReference: The full 32-byte keccak256 hash
// │  │   └─ ChainRefLen: 32 bytes (0x20)
// │  └─ ChainType: eip155 (0x0000)
// └─ Version: 1 (0x0001)

```

### 2. Field Order and Separator **(`address@chain` vs `chain:address`)**

ERC-7930’s human format puts the address first and chain second (using `@`), whereas CAIP-10 puts the chain first (using `:`). T**his difference is purely superficial,** there is no ambiguity or loss of information, it’s simply a reversed order. In practice, this does not cause problems in conversion.

 A parser can easily detect the `@` in an ERC-7930 string and flip the order to produce `chain:address` (CAIP-10) or vice versa.  As long as libraries are aware of both formats, **bidirectional conversion is trivial**. The choice of `@` was likely made to improve human readability (it mimics email addresses like `user@domain`)  but it doesn’t conflict with the CAIP-10 meaning.

**CAIP-10 Order:**

```
chain:address
eip155:1:0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb

```

**ERC-7828 Order:**

```
address@chain#checksum
alice.eth@ethereum#ABC123

```

### 3. Special Character Handling

ERC-7930 is flexible in that the chain and address fields can be arbitrary binary data (the binary format even allows lengths of zero for certain fields in special cases). However, CAIP-10 account IDs are plain text strings intended for use in URIs/URLs, which means **only a limited set of characters are allowed** (letters, digits, `:`, `-`, and `%` , essentially the URL safe character set). If an address or chain ID in ERC-7930 contains bytes outside the URL safe range, we need a way to represent those bytes in the CAIP-10 string. The solution is standard **percent encoding**. ERC-7930’s text format explicitly permits the `%` character so that any byte value can be hex encoded as `%XX`. This is analogous to how URLs encode special characters or binary data as `%20`, `%3A`, etc. (per [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986)).   URL percent encoding is the standard method to include “arbitrary data in a URI using only the US-ASCII characters legal within a URI”. ERC-7930 basically builds this capability in by allowing `%` in its text syntax.

```
Original ERC-7828: alice.eth@ethereum#ABC123
URL Encoded:      alice.eth%40ethereum%23ABC123
CAIP-10 Format:   eip155:1:alice.eth%40ethereum%23ABC123
```

## DID Compatibility Analysis

The potential for ERC-7930/7828 to serve as a basis for Decentralized Identifiers (DIDs) hinges on its alignment with the core tenets of the W3C standard. This framework requires any identifier to be globally unique, cryptographically verifiable, and created without reliance on a central authority. Crucially, a DID must also be resolvable to a **DID Document** a machine readable file containing its public keys and metadata. With these principles as our benchmark, we can analyze the inherent capabilities and current limitations of the ERC-7930/7828 standards. The following analysis examines how ERC-7930/7828 perform against each of these key requirements:

- **Globally Unique Identifier:** By pairing a specific chain ID with a unique account address, ERC-7930 creates a globally unique identifier. This follows the same proven principle as CAIP-10 and is a core requirement for any DID method.
- **No Central Registry Required:**  An address is constructed from public chain data and a user-generated key, making its creation permissionless and free from any central registry. The core address functions independently, aligning with the DID goal of avoiding centralized issuance.  A nuance exists with ERC-7828's use of ENS names, which introduces a dependency on the ENS registry itself, though ENS is a decentralized system.
- **Cryptographically Verifiable:** Control is proven by signing a message with the private key tied to the address. The standard is built on top of native blockchain accounts and simply inherits their strong built in verifiability.
- **Resolvable to a DID Document:**  This is the one missing piece. ERC-7930 defines the **identifier format**, but does not yet specify a **DID Method**—the standardized process for resolving that identifier into a DID Document. It is "DID-ready" but requires a formal method specification to become fully DID-complete.

## **The Role of CAIP-350**

We've seen that ERC-7930 provides a flexible binary format for addresses. But a critical question remains: if the format is generic, how does a wallet or application know what the bytes in the `ChainReference` and `Address` fields actually represent for a *specific* blockchain? An Ethereum address is structured differently from a Bitcoin or Solana address.

This is the problem that **CAIP-350** solves. It acts as a companion standard to ERC-7930, defining the precise serialization rules for different chain families.

- **ERC-7930 defines the *envelope*:** It specifies a generic binary structure:
    
    `Version | ChainType | ChainReferenceLength | ChainReference | AddressLength | Address`. 
    It provides the universal container but is agnostic about its contents.
    
- **CAIP-350 defines the *contents*:** It tells you *what* the data inside the envelope means for a specific chain type (namespace).

Essentially, **ERC-7930 is the box, and CAIP-350 is the set of instructions for what to put in the box and how to pack it**. This modularity allows ERC-7930 to support any new blockchain without changing the ERC-7930 standard itself, you just need to create a new CAIP-350 profile for that new chain.

**Is CAIP-50 compatible with CAIP-350?**

CAIP-50 is a *predecessor* and a different approach to what CAIP-350 and ERC-7930 are trying to solve. They are **not compatible** because they embody fundamentally different design philosophies.

- **CAIP-50 (monolithic):** Defines a *self-contained, all-in-one* binary format using multicodec. Everything, including the multicodec identifier (`0xca`), namespace codes, lengths, and data, is concatenated into a single byte array and then base58-encoded. It's a complete, standalone specification.
- **CAIP-350 + ERC-7930 (modular):** Separates the container from its contents. ERC-7930 defines the *outer structure* (the envelope)  while CAIP-350 defines the *inner serialization rules* for each chain type.

They are incompatible because a CAIP-50 parser would not understand an ERC-7930 binary object, and vice-versa.