# Address Encoding guide
This document is meant for wallet developers trying to reason with and encode/decode Interoperable Addresses.

You can find the Interoperable Address specification: [here](https://github.com/ethereum/ERCs/pull/1002)

Feedback on the standard itself should be directed to the ethereum-magicians thread: https://ethereum-magicians.org/t/erc-interoperable-addresses/23365

## Section 1: resolution-less addresses
The Interoperable Address specification defines a suggested text format for Interoperable Addresses:

```
<human readable name> ::= <account>@<chain>#<checksum>
```

And its binary representation looks barely like this:

```
┌─────────┬────────────┬─────────┬─────────┬─────────┐
│ Version │ Chainidlen │ Chainid │ Addrlen │ Address │
└─────────┴────────────┴─────────┴─────────┴─────────┘
```

### Serializing an Interoperable Address
Starting from:

Chain
: Ethereum Mainnet

Address
: `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

The algorithm for serializing it to an Interoperable Address is as follows:
1. Start from an empty byte array: `[]`
2. Append the version field as a big-endian two-byte integer: `[0001]` [^1]
3. Serialize the chain
    1. Chain _namespace_ spans two bytes and should be taken from the table in the standard's appendix A, in this case (`eip155`) it corresponds to `0000`
    2. Chain _reference_ in this case is a big-endian integer corresponding to the network's chainid (1 for mainnet), left-padded it to a byte boundary: `01`.
    3. Concatenate the two items above: `000001`
4. Compute the length of the above, in this case 3
5. Append the length as a single byte: `[0001 03]` (spaces are just for clarity, it is a one-dimension array)
6. Append the chainid from step 3: `[0001 03 000001]`
7. Serialize the address, in this case 20 raw bytes: `[D8DA6BF26964AF9D7EED9E03E53415D37AA96045]`.
8. Compute the length of the above, in this case `20 == 0x14`
9. Append the length as a single byte: `[0001 03 000001 14]`
10. Append the address from step 7: `[0001 03 000001 14 D8DA6BF26964AF9D7EED9E03E53415D37AA96045]`
11. Done! Interoperable Address v1 is the 26-byte payload: `00010300000114D8DA6BF26964AF9D7EED9E03E53415D37AA96045`

[^1]: Note that you can't assume integers on an arbitrary runtime to be big-endian, e.g. version `256` will be stored on memory as `0100` on big-endian runtimes vs `0001` on little-endian ones, so you'll most likely have to perform this transformation explicitly.

### Parsing an Interoperable Address
Starting from:
```
000122000245296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0205333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5
```

1. Take the first two bytes and interpret them as a big-endian integer: `0x0001 == 1`. We know how to parse version 1, so we can proceed. Remaining payload: `[22000245296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0205333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5]`
2. Take the next byte, `0x22 == 34`. This is the length of the chainid. Remaining payload: `[000245296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0205333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5]`
3. Parse next 34 bytes as the chainid: `000245296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0`, remaining payload: `[205333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5]`
    1. Take the first two bytes (`0002`) and look them up in the table in Appendix A: the chain _namespace_ corresponding to `0002` is 'solana'
    2. The remaining 32 bytes are the chain _reference_: `45296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0`. We will have to refer back to appendix A later, to know how to display it to users.
4. Take the next byte, `0x20 == 32`. Remaining payload: `[5333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5]`
5. Interpret the next 32 bytes as the address: `5333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5`
6. Remaining payload: `[]` Done! We got:
    - Chain namespace: `0002`, solana
    - Chain reference: `45296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0`. (Solana Mainnet genesis blockhash)
    - Address: `5333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5`

### Showing an Interoperable Address
While the algorithm above allows us to make sense of an Interoperable Address, it is not enough to be able to display it in a way that users will find meaningful or even familiar. For that, we have to rely on the suggested human-readable name format at the beginning of this section.

Starting where the last example left off:

Chain namespace
: `0002`, solana

Chain reference
: `45296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0`. (Solana Mainnet genesis blockhash)

Address
: `5333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5`

Let's try to fit that into the `<account>@<chain>#<checksum>` format:

1. Chain id: `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d`
    - human-readable representation of namespace: `solana`
    - separator: always `:`
    - human-readable representation of chain reference, referring to Appendix A:
        > In the human-readable name, it should be displayed in full and base58btc-encoded, as returned by the node.

        display the 32 bytes `45296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0` base58btc-encoded: `5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d`
2. Address: Referring to Appendix B:
    > base58btc public keys should be decoded and stored as a 32 byte payload

    ... the inverse for that will be to base58btc encode the 32-byte payload :grin: 
    from `5333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5` to `MJKqp326RZCHnAAbew9MDdui3iCKWco7fsK9sVuZTX2`
3. Checksum: 
    > calculated by computing the keccak256 hash of the concatenated `Chainidlen`, `Chainid`, `Addrlen` and `Address` fields of the binary representation (that is, the v1 binary representation skipping the `Version` field), and truncating all but the first 4 bytes of the output. Represented as a base16 string as defined in RFC-4648


    - Interoperable Address sans the 'version' field is `000122000245296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef0205333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5`.
    - The keccak256 of the above is `0xaa7e9354e0bdd58d51688fdbcba0e4a77efe398198e1683752f1c1188e44d90d`
    - First 4 bytes, in uppercase base16: `AA7E9354`
4. Tying it all together: `5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d@solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d#AA7E9354`

## Section 2: more human readable addresses
This section is non-normative.

The previous section ended with an unambiguous but verbose representation of the Interoperable Address. Wallets are free to improve upon said representation, replacing the `address` and `chainid` fields with more human-readable alternatives.

### Chainid
For non-EVM chains, wallets could use a shorter format with a private mapping functioning as a layer-zero registry. For example, they may replace `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d` with `solana:mainnet` or even `solana`. Wallets may reference the [CASA] namespaces as a repository of chain types and popular names for them.

In the case of EVM chains, wallets could refer to [ethereum-lists/chains](https://github.com/ethereum-lists/chains) and use the shortnames defined there, replacing:
- `eip155:1` -> `eth`
- `eip155:10` -> `oeth`
- `eip155:11155111` -> `sep`

Ideally, this will be solvable without trusting centralized actors when standards like ERC-7785 reach maturity.

### Address
Wallets are recommended to use existing naming registries to show _names_ to the user in place of _addresses_. The following is a guide on how to implement multi-chain lookup on ENS as an example, but wallets are encouraged to support other naming registries as well.

While ENS was initially focused on Ethereum Mainnet, since the introduction of [ENSIP-9] and [ENSIP-11] it is possible to resolve a name to an address in any chain supported by [SLIP-44] or eip155 via the following algorithm:

#### `address -> name` resolution
1. Start by knowing `chainId` and `address`, presumably from parsing an Interoperable Address as described in [the above section](#parsing-an-interoperable-address).
2. Convert `chainId` from step above into [ENSIP-11] `coinType`
3. Compute the namehash for the [ENSIP-19](https://docs.ens.domains/ensip/19/) reverse resolution from the results of step 1 and 2. Will look something like `<address>.<cointype>.reverse`
4. Call `resolver(bytes32 node)`on the ENS registry contract, using the value from the step above for `node`.
5. Call `name(bytes32 node)` on the contract returned by the step above. Save the response as the `<punycode-encoded name>`.
6. Check direct resolution of name obtained in the step above, and fail the resolution if it does not match, as described in [ENSIP-19]
7. Convert `<punycode-encoded name>` into unicode and display it to the user

#### `name -> address ` resolution
1. Let the user input a destination chain name in whatever format the wallet chooses to support, and convert it to a [CAIP-2] chainId.
2. Convert the result from step 1 into an [ENSIP-11] `coinType`.
3. Let the user input a unicode string instead of the punycode-encoded name.
4. Convert the string from step 3 into a punycode-encoded name.
5. Compute the punycode-encoded name's namehash.
6. Call `addr(namehash, coinType)` on the wallet's default resolver, with the values from steps 5 and 2 respectively. This will return a byte array which as to be interpreted according to the instructions in [ENSIP-9]. Ideally, the algorithm would match the one on appendix B, and therefore the returned value could be used to construct the Interoperable Address directly. But it's possible that they differ for some chain and some ad-hoc conversion is warranted for specific chains and address types. Abstracting this away should be the focus of a wallet-focused Interoperable Address SDK.
    - Failure mode: name is not registered on ENS for the given chain.
7. Construct Interoperable Address with the address returned in step 6 and the destination chain from step 1.
8. The user should obviously not be prompted for the checksum, but it should be displayed somewhere in the wallet UI.

#### Further reading
- It's possible that the lookup information is not present on-chain, but instead the on-chain registry points to an off-chain location for further resolution. If you want to implement support for that, refer to [ENSIP-10] and [ENSIP-16].
- ENS has special treatment of ethereum mainnet / cointype `60` for backwards compatibility reasons, refer to [ENSIP-9] and [ENSIP-11] for details.

[CASA]: https://namespaces.chainagnostic.org/
[CAIP-2]: https://chainagnostic.org/CAIPs/caip-2
[SLIP-44]: https://github.com/satoshilabs/slips/blob/master/slip-0044.md
[ENSIP-9]: https://docs.ens.domains/ensip/9
[ENSIP-10]: https://docs.ens.domains/ensip/10
[ENSIP-11]: https://docs.ens.domains/ensip/11
[ENSIP-16]: https://docs.ens.domains/ensip/16
[ENSIP-19]: https://docs.ens.domains/ensip/19
