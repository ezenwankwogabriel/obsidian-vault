ACID is ==a set of four core principles that guarantee database transactions are processed reliably==. It ensures that even if your application crashes, the server loses power, or multiple users update the exact same data at the same moment, your data never becomes corrupted.

Here is what each letter stands for:

## 1. Atomicity (All or Nothing)

Every transaction is treated as a single "atom." Either the entire transaction succeeds, or the entire transaction fails and rolls back.

- _Example:_ If you transfer ₦5,000 from Account A to Account B, the database must deduct the money from A _and_ add it to B. If the database crashes halfway through, Atomicity ensures the money isn't lost in limbo; the whole transfer is canceled.

## 2. Consistency (Valid Data Only)

A transaction can only take the database from one valid state to another, maintaining all predefined rules, data integrity constraints, and cascades.

- _Example:_ If your database has a rule that an account balance cannot drop below zero, a transaction that attempts to withdraw more money than available will be rejected automatically.

## 3. Isolation (No Interference)

Multiple transactions happening at the identical time cannot interfere with each other. The database makes concurrent transactions execute as if they were running one after the other in isolation.

- _Example:_ If two people try to book the very last seat on a flight at the exact same millisecond, Isolation ensures only one person gets it, and the second person sees the seat as unavailable.

## 4. Durability (Saved for Good)

Once a transaction is committed, it is permanently saved in non-volatile storage (like a hard drive or SSD). It will not be lost even if the power goes out or the system crashes immediately afterward.

---

## In the Context of Our Conversation

- Postgres achieves ACID on a single server by writing changes to a Write-Ahead Log (WAL) before updating the actual database files.
- FoundationDB is unique because it achieves Distributed ACID. It coordinates these four guarantees across hundreds of separate computers simultaneously, which is incredibly difficult to engineer.
