Introducing: Interoperable Addresses
====

*Specs in the formal format are written [here](/specs/addresses/cross-chain-interoperable-addresses-spec.md)*.

Interoperable Addresses is a standard defining address formats for a multi-chain world.

For a normal user, they look like this:

<p>
    <code>
        <span style="color:grey">3::</span><span style="color: blue">vitalik.eth</span>@<span style="color: magenta">eth</span>#<span style="color:grey">5966be0f</span>
    </code>
</p>

- <code><span style="color:grey">3::</span></code>: the Interoperable Address Resolver version, which your wallet may very well omit.
- <code><span style="color: blue">vitalik.eth</span></code>: A human-readable name, specific to a chain. Supported by (anything looking like) ENS.
- <code><span style="color: magenta">eth</span></code>: A human-readable chain name, defined by either https://github.com/ethereum-lists/chains if available (same registry for ERC-3770) or CAIP-2 in all other cases.
- <code><span style="color:grey">5966be0f</span></code>: A checksum so you can at a glance validate what you see matches what you expect to be under the hood.

---

For nerds and computers, however, they look like this:

<p>
    <code>
        <span style="color:grey">3:</span><span style="color:green">eip155:1</span>:<span style="color: red">0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e</span>::<span style="color:magenta">eip155:1</span>:<span style="color: blue">0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045</span>#<span style="color:grey">5966be0f</span>
    </code>
</p>

- <code><span style="color:grey">3:</span></code>: Interoperable Address version.
- <code><span style="color:green">eip155:1</span></code>: CAIP-2 id of chain where the naming registry is located
- <code><span style="color: red">0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e</span></code>: Address where to find the naming registry
- <code><span style="color:magenta">eip155:1</span></code>: CAIP-2 id of chain we are referring to
- <code><span style="color: blue">0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045</span></code>: Chain-specific address we are targeting in that chain
- <code><span style="color:grey">5966be0f</span></code>: Checksum of all previous fields, to make sure nothing got lost in transit

---

The Interoperable Address is a format for strings to:
- Fully specify an address on a particular chain (using CAIP-10), supporting any EVM-ecosystem chain and any chain on SLIP-044.
- Also include the information to display it to a human in a readable way, as displayed above.

Some of its features are:
- Uses mature technologies such as CAIP-10 and ENS.
- Is extensible for future resolving mechanisms (eg: ERC-7785, when/if it reaches production).
- Resolution of Interoperable Addresses to human readable names is fully deterministic.
- Edge cases are securely abstractable to a library/SDK for wallets.
- Does not enshrine a particular ENS contract, making it flexible for a future naming-only rollup, instances on other rollups or even an ENS fork.

And, in all honesty, it has some drawbacks as well:
- Exposes a version number to users (although that can be mitigated by wallet UX)
- Supporting different name resolution contracts mean human-readable name -> machine address resolution can produce different results (which is mitigated by the checksum)
- Wallets will have to maintain a set of name resolving contracts considered trustworthy, with a trust model similar to RPC urls (can be taken care of by a library/SDK, but user overrides should be supported).
