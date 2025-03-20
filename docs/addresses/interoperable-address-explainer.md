Introducing: Interoperable Addresses
====

*Specs in the formal, normative format can be found [here](/specs/addresses/cross-chain-interoperable-addresses-spec.md)*.

Interoperable Addresses is a standard defining address formats for a multi-chain world.

They contain the following information:

```solidity
    struct InteroperableAddress {
        bytes targetChainId;
        bytes targetAddress;
        bytes resolverChainId;
        bytes resolverAddress;
    }
```

Let's take an example:

<pre>
InteroperableAddress addy = InteroperableAddress({
    targetChainId: hex'<span style="color:green">0000</span><span style="color:orange">01</span>',
    targetAddress: hex'<span style="color:blue">D8DA6BF26964AF9D7EED9E03E53415D37AA96045</span>',
    resolverChainId: hex'<span style="color:black">0000<span style="color:pink">01</span>',
    resolverAddress: hex'<span style="color:magenta">00000000000C2E074EC69A0DFB2997BA6C7D2E1E</span>'
})
</pre>

Would be seen by users like this:

<pre>
    <span style="color:blue">vitalik.eth</span>@<span style="color:green">eip:155</span>:<span style="color:orange">1</span>#<span style="color:grey">618AD0D1</span>
</pre>

...and in memory, it would actually be laid out like this:

<pre>
C000618AD0D10000000000000000000000000000000000000000000000000001
<span style="color:green">0000</span><span style="color:orange">01</span><span style="color: grey">000000000000000000000000000000000000000000000000000000000C</span>
<span style="color:blue">D8DA6BF26964AF9D7EED9E03E53415D37AA96045</span><span style="color: grey">000000000000000000000028</span>
<span style="color:black">0000</span><span style="color:pink">01</span><span style="color: grey">000000000000000000000000000000000000000000000000000000000C</span>
<span style="color:magenta">00000000000C2E074EC69A0DFB2997BA6C7D2E1E</span><span style="color: grey">000000000000000000000028</span>
</pre>

It includes:
- A full specification of what (address,chain) pair we're referring to
- A full specification of on which (address,chain) pair the contract responsible for resolving addresses to names lives
- ... and a full 32-byte word before any of that?

Said word is what enables us to make this standard extensible, let's take a closer look:

<pre>
MSB                                                          LSB
<span style="color: red">C000</span><span style="color:green">6164DA1</span><span style="color:blue">00000000000000000000000000000000000000000000000000001</span>
^^^^------------------------ 255-240 <span style="color:red">Interoperable Address Version</span>
    ^^^^^^^^ --------------- 239-208 <span style="color:green">Checksum</span>
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
            \--------------- 207-0   <span style="color:blue">Resolver Interface Key</span>
</pre>

- <span style="color:red">Interoperable Address Version</span>: First two bytes of the payload allows us to discern how we should interpret it, more on this later
- <span style="color:green">Checksum</span>: Covers only the (address,chain) we are referring to -> is independent of the method used to display the address, or how it is serialized -> makes naming collisions evident 
- <span style="color:blue">Resolver Interface Key</span>: in addresses version `0xC000`, instructs us on what to expect from `nameContractAddress` and `nameContractChain`. In particular, this value means to expect an ENSIP-11 contract.

But wait! I actually want...
- A canonical representation: use Interoperable Address Version `0x8000`
- To be as space-efficient as possible for EVM chains: use Interoperable Address Version `0x0000` or `0x0001`
- To display human-readable chain names to users, but not rely on centralized lists: define a new 'Resolver Interface Key' as soon as ERC-7785 reaches production

What some other resolvers look like:

`0x0000`:
<pre>
MSB                                                            LSB
0x<span style="color: red">0000</span><span style="color: green">618ad0d1</span><span style="color: blue">00000000000</span><span style="color: magenta">1D8DA6BF26964AF9D7EED9E03E53415D37AA96045</span>
  ^^^^------------------------ 255-240 <span style="color: red">Interoperable Address version</span>
      ^^^^^^^^ --------------- 239-208 <span style="color: green">Checksum</span>
              ^^^^^^^^^^^^
                \------------- 207-160 <span style="color: blue">Chainid</span>
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                            \- 159-0   <span style="color: magenta">Address</<span>
</pre>
Is represented as:
<pre>
<span style="color: magenta">0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045</span>@eip155:<span style="color: blue">1</span>#<span style="color: green">618AD0D1</span>
</pre>

To fit in a single word, it sacrifices:
* Ability to refer to CAIP-2 namespaces other than eip155 (it's implied by the version number)
* Ability to represent addresses longer than 20 bytes
* Ability to refer to chains with chainids longer than 48 bits (rules out ERC-7785)

`0x8000`:
```
MSB                                                          LSB
8000618AD0D10000000000000000000000000000000000000000000000000000
^^^^------------------------ 255-240 Interoperable Address Version
    ^^^^^^^^ --------------- 239-208 Checksum
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              \------------- 207-0   not used, always zero
MSB                                                          LSB
00000100000000000000000000000000000000000000000000000000000000C0
^^^^^^------------------------------------------------------------- 255-208 bytes array payload
      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ---- 9-207   padding
                                                              ^^ -- 0-8    2*6 (payload length)
MSB                                                          LSB
D8DA6BF26964AF9D7EED9E03E53415D37AA96045000000000000000000000028
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ------------------------- 255-97 bytes array payload
                                        ^^^^^^^^^^^^^^^^^^^^^^ ---- 9-96   padding
                                                              ^^ -- 0-8    2*20 (payload length)
```
Is represented as:
`0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155:1#618AD0D1`

Some properties of this system:
- Interoperable addresses of any version can always be converted to format `0x8000` -> forwards-compatibility and graceful degradation of interfaces.
- Interoperable addresses of any version can always be casted to a valid CAIP-10.
- Addresses of any length in chains of any supported CAIP-2 namespace can be represented
- Future-proofing for developments on future resolving mechanisms.
- Implementable right now with current tools & standards.
- Edge cases are securely abstractable to a library/SDK for wallets.
- Does not enshrine a particular naming contract, making it flexible for a future naming-only rollup, instances on other rollups or even an ENS competitor.

And, in all honesty, some drawbacks as well:
- Supporting different name resolution contracts mean human-readable name -> machine address resolution can produce different results (which is mitigated by the checksum)
- Wallets will have to maintain a set of name resolving contracts considered trustworthy, with a trust model similar to RPC urls (can be taken care of by a library/SDK, but user overrides should be supported).
