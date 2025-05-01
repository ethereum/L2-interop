# bytes[] attributes vs bytes extraData

## extraData Field

Instead of requiring structured arrays of attributes, we propose to use a single `bytes extraData` field to encode optional gateway-specific metadata.

```solidity
// Optional encoded gateway metadata (leave empty if unused)
bytes calldata extraData 
```

- `extraData` is an opaque blob interpreted by the origin gateway.
- It MAY contain data related to the underlying messenger logic such as gas limits, instructions, callback options, or any other protocol-specific configurations.
- If unused, `extraData` should be left empty.
- Optionally the Gateways could expose a view function that decodes their `extraData` format. This is a function defined in the `IGateway` interface but a nice to have of the Gateway implementations.

This approach offers the following advantages:

- **Simplicity**: The `sendMessage` interface remains minimal and easy to implement.
- **Flexibility**: Each gateway or adapter can define and interpret `extraData` freely, without requiring standardized attribute formats.
- **Gas Efficiency**: Encoding and decoding a single blob of bytes is cheaper than parsing an array of structured attributes.
- **Future Extensibility**: New metadata types can be supported within `extraData` without changing the interface.
- Attributes force upgrades whenever the underlying messenger changes, as their structure must be explicitly supported. In contrast, `extraData` avoids this need by being fully opaque and simple, allowing changes without requiring interface upgrades.

It is recommended that gateways and adapters document their expected `extraData` formats if interoperability between adapters is desired.

>💡
>**Non-EVM Friendliness:**
>Using a single opaque `extraData` field improves compatibility with non-EVM chains, which may not natively support ABI encoding. It ensures that both EVM and non-EVM ecosystems can adopt the standard naturally, using their preferred serialization methods.
>This design enables cross-chain adapters to interpret metadata according to the serialization standards of their respective ecosystems  without requiring translation of EVM-specific ABI structures.

## Comparison

| Aspect | `bytes[] attributes` | `bytes extraData` |
| --- | --- | --- |
| Format | Structured array (each entry = ABI function call) | Single opaque blob |
| Structure | Complete structured | Still structured, but it’s “proprietary” structured by the underlying messenger |
| Flexibility | High, but depends on standardizing attribute formats | Very high, gateway-specific interpretation/implementation |
| Simplicity | More complex for developers (attribute definition and parsing) | Simpler for developers (optional opaque data) |
| Composability | Facilitates standardized cross-bridge features | Relies on bridge or adapter-level documentation and agreement |
| Standardization level | can be standardized with new ERCs | the content is hard to standardize |
| Compatibility between them | cannot support opaque bytes extraData | can be encoded from bytes[] attributes |
| Gas Usage | Higher (decoding multiple structured elements) | Lower (decoding a single blob) |
| Spec Complexity | Higher (attribute standardization and management required) | Lower (no structure mandated by the interface) |
