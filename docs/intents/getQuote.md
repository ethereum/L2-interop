# GetQuote API Standard

## Request Format

```typescript
interface GetQuoteRequest {
    user: string; // erc-7930 interoperable address
    availableInputs: {
        user: string; // erc-7930 interoperable address
        asset: string; // erc-7930 interoperable address
        amount: bigint;
        lock?: { // If undefined, asset is not locked and needs to be escrowed.
            kind: 'the-compact' | 'rhinestone'; // to be expanded as more become available
            params?: unknown; // depends on kind, eg. lockTag for the compact
        };
    }[]; // order of inputs significant if preference is 'input-priority'
    requestedOutputs: {
        receiver: string; // erc-7930 interoperable address
        asset: string; // erc-7930 interoperable address
        amount: bigint;
        calldata?: string; // details TBD (eg, what interface is used?)
    }[];
    minValidUntil?: number;
    preference?: 'price' | 'speed' | 'input-priority' | 'trust-minimization';
}
```

## Response Format

```typescript
interface GetQuoteResponse {
    quotes: {
        orders: { // Full EIP712 compliant
            signatureType: 'eip-712' | 'erc-3009';
            domain: string; // erc-7930 interoperable address
            primaryType: string;
            message: object; // to be signed and submitted back
        }[];
        details: {
            requestedOutputs: {
                user: string; // erc-7930 interoperable address
                asset: string; // erc-7930 interoperable address
                amount: bigint;
                calldata?: string;
            }[];
            availableInputs: {
                user: string; // erc-7930 interoperable address
                asset: string; // erc-7930 interoperable address
                amount: bigint;
                lock?: { // If undefined, asset is not locked and needs to be escrowed.
                    kind: 'the-compact' | 'rhinestone'; // to be expanded as more become available
                    params?: unknown; // depends on kind, eg. lockTag for the compact
                };
            }[];
        };
        validUntil?: number;
        eta?: number;
        quoteId: string;
        provider: string;
    }[];
}
```

# SubmitIntent API Standard

## Request

```typescript
interface IntentRequest {
    order: object; // GaslessCrossChainOrder
    signature: object;
    quoteId?: string;
    provider: string;
}
```

## Response

```typescript
interface IntentResponse {
    orderId?: string;
    status: string;
    message?: string;
}
```