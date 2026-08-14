
### Atomicity

**All or nothing.**

A transaction's operations either **all succeed or all get rolled back**.

> "Don't leave the database half-done."

Example: Transfer money → subtract from A **and** add to B, or do neither.

### Isolation

**Transactions don't interfere with each other.**

It controls how concurrent transactions interact with and see each other's changes.

> "Don't let another transaction mess up my assumptions while I'm working."

Example: Two requests both trying to use the **last available trusted-contact slot** shouldn't both think the slot is available.