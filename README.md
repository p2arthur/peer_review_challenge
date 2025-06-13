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

---

## 🧪 Code Challenge Overview

**Task:** Implement URL parameter parsing for one of 12 Algorand transaction types in the Transaction Wizard.

**My Choice:** `pay` (Payment transactions)

**Goal:** Allow users to pre-fill transaction forms in the UI by passing URL parameters like `type[0]=pay`, `sender[0]`, etc.

---

## 🛠️ Implementation Breakdown

### 🧩 Entry Point – `useTransactionSearchParamsBuilder`

**Uses react router useSearchParams() hook under the hood to get the transaction params**
**transformedParams()**
1 - Transforms the params from the URL into transaction params and group them by index

**transformSearchParamsTransations()**
2 - Transform the groups of transaction params into actual transaction objects and pass them into a zod form
3 - Validates the form fields with zod
4 - P

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
- Used `@param` and `@throws` JSDoc annotations:

```ts
/**
 * @param params Parsed search parameters
 * @returns A structured payment transaction
 * @throws Error if required fields are missing
 */
```

---

## 🌱 Reflections & Improvements

Throughout this challenge, I learned a lot by following the existing architecture and patterns in the codebase. I became more conscious of how to:

- **Write better inline documentation** with JSDoc
- **Think through edge cases and validation** with Zod
- **Structure feature design** to align with product goals

One thing I would improve is **starting with unit tests** earlier to isolate and validate logic before jumping into integration. I also realized that while I’m comfortable with building features, I want to improve how I document them for others, and how I approach modular, scalable design from day one.

This project helped me grow in that direction, and I’d love to keep building in environments that value these practices.

---

## 🙌 Final Thoughts

This challenge was incredibly rewarding and aligned with the kind of thoughtful, organized, and forward-looking development environment I hope to be part of.

Joining Algorand and working with developers I met at the Castell Dcode retreat—including some of my biggest inspirations in this space—would be a dream opportunity.

Thanks for reviewing my work! 🚀
