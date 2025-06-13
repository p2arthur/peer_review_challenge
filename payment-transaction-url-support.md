# 💸 Payment Transaction Support via URL Params in Transaction Wizard

This guide documents the step-by-step process of implementing **URL parameter support** for **payment transactions** in the Transaction Wizard, part of the [lora.algokit.io](https://lora.algokit.io) project.

## 🚀 Overview

The goal was to allow users to create payment transactions by visiting a URL with search parameters like `type[0]=pay`, `sender[0]`, `receiver[0]`, and `amount[0]`. This feature enhances testing, sharing, and automating transaction creation.

## 🔍 Initial Exploration

### Selected Transaction Type
After reviewing the project spec, I chose to implement **payment transactions** because they are fundamental to blockchain interactions.

### Local Setup
Attempted to run the project immediately but encountered a localnet error. A quick:

```bash
algokit localnet start
```

solved it.

## 🔧 Implementation Steps

### 1. Entry Point: `useTransactionSearchParamsBuilder`

```ts
export function useTransactionSearchParamsBuilder() {
  const [searchParams] = useSearchParams();
  const transformedParams = transformSearchParams(searchParams);
  const { transactions, errors } = transformSearchParamsTransactions(transformedParams);

  useEffect(() => {
    if (errors?.length > 0) {
      for (const error of errors) toast.error(error);
    }
  }, [errors]);

  return transactions;
}
```

This hook collects and parses the URL parameters.

---

### 2. Transform Search Params

```ts
const transformSearchParams = (searchParams: URLSearchParams) => {
  const entries = Array.from(searchParams.entries());
  // Groups entries by [index] (e.g. type[0], sender[0])
}
```

Uses regex to group parameters based on index.

---

### 3. Dispatch Parser by Transaction Type

In `transformSearchParamsTransactions`, we define which parser to use:

```ts
if (searchParamTransaction.type === algosdk.TransactionType.pay) {
  const paymentTransaction = transformPaymentTransaction(searchParamTransaction);
  paymentTransactionFormSchema.parse(paymentTransaction);
  transactionsFromSearchParams.push(paymentTransaction);
}
```

---

### 4. Payment Transformer Function

```ts
export const transformPaymentTransaction = (params: BaseSearchParamTransaction): BuildPaymentTransactionResult => ({
  id: randomGuid(),
  type: BuildableTransactionType.Payment,
  sender: { value: params.sender, resolvedAddress: params.sender },
  receiver: { value: params.receiver, resolvedAddress: params.receiver },
  amount: Number(params.amount),
  fee: params.fee ? { setAutomatically: false, value: microAlgo(Number(params.fee)).algo } : { setAutomatically: true },
  note: params.note,
  validRounds: { setAutomatically: true, firstValid: undefined, lastValid: undefined },
});
```

---

### 5. Zod Validation Schema

```ts
export const paymentTransactionFormSchema = z.object({
  ...commonSchema,
  ...senderFieldSchema,
  ...receiverFieldSchema,
  amount: bigIntSchema(z.bigint().min(1n)),
  closeRemainderTo: z.string().optional(),
  note: z.string().optional(),
});
```

- Ensures `amount > 0`
- Handles optional fields like `note` and `closeRemainderTo`

---

### 6. Test with Live URL

```txt
http://localhost:1420/localnet/transaction-wizard?type[0]=pay&sender[0]=TGIPEO...&receiver[0]=TGIPEO...&amount[0]=1
```

The transaction appeared perfectly in the UI 🎉

---

## 🧪 Testing

### ✅ Integration Test

```ts
it('should render a transaction with the correct sender and receiver', () => {
  renderTxnsWizardPageWithSearchParams({
    searchParams: new URLSearchParams({
      'type[0]': 'pay',
      'sender[0]': sender,
      'receiver[0]': receiver,
      'amount[0]': '1',
      'fee[0]': '1000',
    }),
  });
  expect(screen.getByText(sender)).toBeInTheDocument();
  expect(screen.getByText(receiver)).toBeInTheDocument();
});
```

### ❌ Edge Case Test

```ts
it('should not allow amount = 0', () => {
  renderTxnsWizardPageWithSearchParams({
    searchParams: new URLSearchParams({
      'type[0]': 'pay',
      'sender[0]': sender,
      'receiver[0]': receiver,
      'amount[0]': '0',
    }),
  });
  expect(screen.getByText('No transactions.')).toBeInTheDocument();
});
```

### 🧪 Unit Test

```ts
it('parses a valid payment transaction', () => {
  const result = transformPaymentTransaction({
    type: 'pay',
    sender: 'ADDR1',
    receiver: 'ADDR2',
    amount: '1',
    fee: '1000',
  });

  expect(result).toMatchObject({
    sender: { value: 'ADDR1' },
    receiver: { value: 'ADDR2' },
    amount: 1,
    fee: { value: expect.any(Number) },
  });
});
```

---

## 🧾 Documentation & Dev Notes

- Added support to README for new params
- Included example URLs and error cases
- Used `@param` and `@throws` JSDoc comments in transformers for better clarity

```ts
/**
 * @param params Parsed search parameters
 * @returns Payment transaction object
 * @throws Error if required fields are missing or invalid
 */
```

---

## 👋 About the Author

This feature was implemented by **Arthur Dias Rabelo**, a fullstack + smart contract developer who met the AlgoKit Lora team at the [Castell Dcode Retreat](https://linkedin.com/in/arthurrabelo).

During the retreat, Arthur co-built `weDev`, a launchpad for DAO tools using AlgoKit 3.0 and TypeScript contracts.

**LinkedIn:** [@arthurrabelo](https://linkedin.com/in/arthurrabelo)  
**GitHub:** [p2arthur](https://github.com/p2arthur)

---

This implementation ensures deep-linkable payment transactions that follow the architecture and code quality standards of the Transaction Wizard project. Future work could include extending support for asset transfers and ABI method calls.

Happy building! 🚀
