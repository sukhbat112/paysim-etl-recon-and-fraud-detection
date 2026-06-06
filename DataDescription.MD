# PaySim Dataset

PaySim is a synthetic dataset that mimics mobile money transactions. It was built from about one month of real transaction logs from a mobile money service in an African country, then scaled down to roughly a quarter of the original size for the public Kaggle release. Because it is synthetic, it can include fraud labels that real, privacy-sensitive data usually cannot.

- **Source:** [Kaggle: Synthetic Financial Datasets For Fraud Detection](https://www.kaggle.com/datasets/ealaxi/paysim1)
- **Paper:** E. A. Lopez-Rojas, A. Elmir, and S. Axelsson, "PaySim: A financial mobile money simulator for fraud detection," 28th European Modeling and Simulation Symposium (EMSS), 2016.
- **Size:** 6,362,620 rows, 11 columns.
- **Time span:** 744 steps, where 1 step is 1 hour, so about 30 days.

## Columns

| Column | Type | Description |
|--------|------|-------------|
| step | int | Time as hours from the start (1 step = 1 hour, 744 total). |
| type | text | Transaction type: CASH_IN, CASH_OUT, DEBIT, PAYMENT, or TRANSFER. |
| amount | float | Transaction amount in the local currency. |
| nameOrig | text | The account that started the transaction. |
| oldbalanceOrg | float | Sender's balance before the transaction. |
| newbalanceOrig | float | Sender's balance after the transaction. |
| nameDest | text | The account that received the transaction. |
| oldbalanceDest | float | Receiver's balance before the transaction. |
| newbalanceDest | float | Receiver's balance after the transaction. |
| isFraud | int | 1 if the transaction was made by a fraud agent in the simulation, else 0. |
| isFlaggedFraud | int | 1 if a simple business rule flagged it (a single transfer over 200,000), else 0. |

Example row:

```
1, PAYMENT, 9839.64, C1231006815, 170136.0, 160296.36, M1979787155, 0.0, 0.0, 0, 0
```

## Things to know

- **Account prefixes.** Names that start with `C` are customers; names that start with `M` are merchants. Merchants have no balance information, so `oldbalanceDest` and `newbalanceDest` are 0 for them.
- **Where fraud happens.** All fraud is in `TRANSFER` and `CASH_OUT` transactions. The pattern is: take over an account, move the money out with a transfer, then cash it out. About 0.13% of all transactions are fraud (8,213 out of 6.36M).
- **isFlaggedFraud is a weak rule, not a label.** It only fires on transfers over 200,000 and flags just 16 transactions in the whole dataset. `isFraud` is the real label; `isFlaggedFraud` is the output of an existing simple rule, useful as a baseline to beat.

## Important note on the balance columns

The original dataset note says:

> Transactions which are detected as fraud are cancelled, so for fraud detection these columns (oldbalanceOrg, newbalanceOrig, oldbalanceDest, newbalanceDest) must not be used.

The reason is that the simulator builds fraud by emptying the account, so the balance columns end up basically restating the label. Using them gives a near-perfect but unrealistic score. The appendix in [`02_fraud_detection.ipynb`](02_fraud_detection.ipynb) measures this directly (for example, `amount == oldbalanceOrg` holds for about 98% of fraud and about 0% of legitimate transactions), and the notebook handles it by splitting features into a real-time set and an after-the-transaction set.
