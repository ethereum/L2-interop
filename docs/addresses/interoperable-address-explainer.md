Introducing: Interoperable Addresses
====

*Formal format specifications are written [here](/specs/addresses/cross-chain-interoperable-addresses-spec.md)*.

Interoperable Addresses is a standard that defines address formats for a multi-chain world.

For a normal user, they look like this:

<p>
    <code>
        <span style="color:grey">3::</span><span style="color: blue">vitalik.eth</span>@<span style="color: magenta">eth</span>#<span style="color:grey">5966be0f</span>
    </code>
</p>

- <code><span style="color:grey">3::</span></code>: the Interoperable Address Resolver version, which the wallet may omit.
- <code><span style="color: blue">vitalik.eth</span></code>: A human-readable name specific to a chain, supported by systems similar to ENS.
- <code><span style="color: magenta">eth</span></code>: A human-readable chain name, defined by https://github.com/ethereum-lists/chains if available (same registry as ERC-3770), or by CAIP-2 in all other cases.
- <code><span style="color:grey">5966be0f</span></code>: A checksum that allows quick validation that what is displayed matches what is expected.

---

For nerds and computers, however, they look like this:

<p>
    <code>
        <span style="color:grey">3:</span><span style="color:green">eip155:1</span>:<span style="color: red">0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e</span>::<span style="color:magenta">eip155:1</span>:<span style="color: blue">0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045</span>#<span style="color:grey">5966be0f</span>
    </code>
</p>

- <code><span style="color:grey">3:</span></code>: Interoperable Address version.
- <code><span style="color:green">eip155:1</span></code>: CAIP-2 ID of the chain where the naming registry is located.
- <code><span style="color: red">0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e</span></code>: Address of the naming registry.
- <code><span style="color:magenta">eip155:1</span></code>: CAIP-2 ID of the chain.
- <code><span style="color: blue">0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045</span></code>: Chain-specific address target.
- <code><span style="color:grey">5966be0f</span></code>: Checksum of all previous fields to prevent any loss in transit.

---

The Interoperable Address is a format for strings to:
- Fully specify an address on a particular chain (using CAIP-10), supporting any EVM-ecosystem chain and any chain on SLIP-044.
- Include information for displaying it in a human-readable format, as shown above.

Some of its features are:
- It uses mature technologies such as CAIP-10 and ENS.
- It is extensible for future resolving mechanisms (eg: ERC-7785, when/if it reaches production).
- The resolution of Interoperable Addresses to human readable names is fully deterministic.
- The edge cases can be securely abstracted into a library/SDK for wallets.
- It does not enshrine a specific ENS contract, allowing flexibility for future naming-only rollups, deployments on other rollups, or even an ENS fork.

And, in all honesty, it has some drawbacks as well:
- It exposes a version number to users, though this can be mitigated through wallet UX.
- Supporting different name resolution contracts means that resolving a human-readable name to a machine address may produce different results. However, this is mitigated by the checksum.
- Wallets must maintain a set of trusted name-resolving contracts, using a trust model similar to RPC URLs. This can be handled by a library/SDK, but users should be able to override it.
