# 🧑‍💻 Arthur Dias Rabelo – AlgoKit Code Challenge Submission

Welcome! This README provides a structured overview of my background, Algorand-specific experience, and a deep dive into my implementation of URL parameter support for payment transactions in the Transaction Wizard on lora.algokit.io.

---

## 🙋‍♂️ About Me

**Name:** Arthur Dias Rabelo  
**Location:** Vancouver, BC, Canada  
**GitHub:** [p2arthur](https://github.com/p2arthur)  
**LinkedIn:** [arthurrabelo](https://linkedin.com/in/arthurrabelo)  
**Portfolio:** [behance.net/arthurrabelouxui](https://behance.net/arthurrabelouxui)

I'm a fullstack web developer with over 4 years of experience focused on React, Node.js, and smart contract development using AlgoKit. I bring a blend of UX design thinking and scalable coding practices, and I thrive on shipping meaningful products.

---

## 🧠 Algorand Experience

- **Smart Contract Development** using AlgoKit with TypeScript and TEAL.
- **Co-founder of [Wecoop](https://github.com/p2arthur)** – a Web3 social platform built on Algorand with on-chain voting and rewards.
- **Participant in the Castell Dcode Dev Retreat** (Mar 2024), hosted by the Algorand Foundation:
  - Built `weDev`, a no-code DAO launchpad using AlgoKit 3.0
  - Collaborated with Algorand Foundation team members and senior ecosystem developers
- Regularly work on **LangChain AI integrations**, combining Web3 data with intelligent agents.
- Deeply connected with the ecosystem and passionate about making Web3 tools more accessible and developer-friendly.

---

## 🧪 Code Challenge Overview

**Task:** Implement URL parameter parsing for one of 12 Algorand transaction types in the Transaction Wizard.

**My Choice:** `pay` (Payment transactions)

**Goal:** Allow users to pre-fill transaction forms in the UI by passing URL parameters like `type[0]=pay`, `sender[0]`, etc.

This was an excellent opportunity to work within a real production-grade codebase and follow patterns designed for clarity and extensibility.

---

## 🛠️ Implementation Breakdown

### 🧩 Entry Point – `useTransactionSearchParamsBuilder`

_Uses react router useSearchParams() hook under the hood to get the transaction params_

**transformedParams()** - parameter transformation  
**transformSearchParamsTransations()** - transaction transformation and zod validation

1. Transforms the params from the URL into transaction params and groups them by index
2. Dispatches the params based on the transaction type in the URL (e.g., `pay`)
3. Transforms grouped params into transaction objects
4. Validates the fields using Zod schema
5. Pushes the transactions into the transaction wizard for UI rendering

```ts
export function useTransactionSearchParamsBuilder() {
  const [searchParams] = useSearchParams();
  const transformedParams = transformSearchParams(searchParams);
  const { transactions, errors } =
    transformSearchParamsTransactions(transformedParams);

  useEffect(() => {
    if (errors?.length > 0) {
      for (const error of errors) toast.error(error);
    }
  }, [errors]);

  return transactions;
}
```

---

### 🔁 Parameter Transformation – `transformSearchParams`

```ts
const transformSearchParams = (searchParams: URLSearchParams) => {
  const entries = Array.from(searchParams.entries());
  // Uses regex to group by key index, e.g., type[0], sender[0]
};
```

---

### 🚦 Dispatch Logic – Add Support for `pay`

```ts
if (searchParamTransaction.type === algosdk.TransactionType.pay) {
  const paymentTransaction = transformPaymentTransaction(
    searchParamTransaction
  );
  paymentTransactionFormSchema.parse(paymentTransaction);
  transactionsFromSearchParams.push(paymentTransaction);
}
```

---

### ⚙️ Transaction Transformer – `transformPaymentTransaction`

```ts
export const transformPaymentTransaction = (
  params: BaseSearchParamTransaction
): BuildPaymentTransactionResult => ({
  id: randomGuid(),
  type: BuildableTransactionType.Payment,
  sender: { value: params.sender, resolvedAddress: params.sender },
  receiver: { value: params.receiver, resolvedAddress: params.receiver },
  amount: Number(params.amount),
  fee: params.fee
    ? { setAutomatically: false, value: microAlgo(Number(params.fee)).algo }
    : { setAutomatically: true },
  note: params.note,
  validRounds: {
    setAutomatically: true,
    firstValid: undefined,
    lastValid: undefined,
  },
});
```

---

### ✅ Zod Schema Validation – `paymentTransactionFormSchema`

```ts
export const paymentTransactionFormSchema = z.object({
  ...commonSchema,
  ...senderFieldSchema,
  ...receiverFieldSchema,
  amount: bigIntSchema(z.bigint().min(1n)), // Ensures > 0
  closeRemainderTo: z.string().optional(),
  note: z.string().optional(),
});
```

---

### 🔗 Example Deep Link URL

```txt
http://localhost:1420/localnet/transaction-wizard?type[0]=pay&sender[0]=ADDR1&receiver[0]=ADDR2&amount[0]=1
```

---

## 🧪 Testing Coverage

### ✅ Integration Test

```ts
it("should render sender and receiver", () => {
  renderTxnsWizardPageWithSearchParams({
    searchParams: new URLSearchParams({
      "type[0]": "pay",
      "sender[0]": "ADDR1",
      "receiver[0]": "ADDR2",
      "amount[0]": "1",
    }),
  });
  expect(screen.getByText("ADDR1")).toBeInTheDocument();
  expect(screen.getByText("ADDR2")).toBeInTheDocument();
});
```

### 🚫 Edge Case

```ts
it("should reject amount = 0", () => {
  renderTxnsWizardPageWithSearchParams({
    searchParams: new URLSearchParams({
      "type[0]": "pay",
      "sender[0]": "ADDR1",
      "receiver[0]": "ADDR2",
      "amount[0]": "0",
    }),
  });
  expect(screen.getByText("No transactions.")).toBeInTheDocument();
});
```

---

## 📝 Documentation Practices

- Documented supported fields and examples in the README.
- Added JSDoc-style comments to describe function parameters and expected outputs:

```ts
/**
 * @param params Parsed search parameters
 * @returns A structured payment transaction
 * @throws Error if required fields are missing
 */
```

This helped me better understand how to make code easier to onboard and safer to extend—skills I want to keep improving.

---

## 🌱 Reflections & Improvements

Throughout this challenge, I learned a lot by following the architecture and conventions of a well-maintained codebase. It gave me new insight into:

- Writing clearer documentation using JSDoc
- Handling edge cases proactively with Zod
- Designing modular feature logic to integrate cleanly with existing systems

Some areas I’d like to keep improving:

- **Feature design:** I want to be more deliberate in planning, scoping, and documenting features.
- **Testing strategy:** I’d start with unit tests earlier to validate isolated logic before layering on integration.
- **Developer empathy:** Writing better docs and validation logic makes a huge difference for others working in the same code.

This challenge made me excited about the kind of team and culture I want to be part of. I’d love to work in a highly collaborative, well-organized, and impact-driven environment.

---

## 🙌 Final Thoughts

This was more than just a coding challenge—it was a glimpse into the kind of codebase I want to contribute to every day.

Joining Algorand and working with the developers I met at the Castell Dcode retreat—including some of my biggest inspirations—would be a dream come true.

Thanks for reviewing my work! 🚀
