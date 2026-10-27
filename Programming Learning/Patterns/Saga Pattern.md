A **Saga** is a way to manage a business transaction that spans **multiple independent services**, without requiring one giant database transaction.

This is directly relevant to our Online Boutique checkout discussion.

### The problem

Imagine checkout:

```text
Checkout
   │
   ├── Cart ✓
   ├── Payment ✓
   ├── Shipping ✗
   └── Email
```

Payment has already happened, but shipping failed.

You can't do:

```text
BEGIN TRANSACTION
  cart
  payment
  shipping
COMMIT
```

because these are separate services/databases.

A Saga says:

> Break the overall transaction into smaller local transactions, and define what to do when a later step fails.

---

## Simple example

Suppose an order involves:

```text
1. Reserve inventory
2. Charge payment
3. Create shipment
```

Successful flow:

```text
Reserve inventory
       ↓
Charge payment
       ↓
Create shipment
       ↓
      DONE
```

But suppose shipment fails:

```text
Reserve inventory ✓
       ↓
Charge payment ✓
       ↓
Create shipment ✗
```

The Saga executes **compensating actions**:

```text
Create shipment ✗
       ↓
Refund payment
       ↓
Release inventory
       ↓
Order cancelled
```

So:

```text
Forward actions
────────────────────────────
Reserve inventory
Charge payment
Create shipment


Compensating actions
────────────────────────────
Release inventory
Refund payment
```

The compensation isn't necessarily a literal database rollback. It's a **new business operation that reverses the effect**.

---

## Two common Saga styles

### 1. Choreography

Services react to events.

```text
OrderCreated
     ↓
Inventory
     ↓
InventoryReserved
     ↓
Payment
     ↓
PaymentCompleted
     ↓
Shipping
```

If something fails, another event triggers compensation.

There's no central coordinator.

```text
Inventory → Payment → Shipping
    ↑          ↑          ↑
       events
```

### 2. Orchestration

A central orchestrator controls the workflow.

```text
              Checkout/Saga
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Inventory    Payment     Shipping
```

It effectively says:

```text
Reserve inventory
       ↓
Charge payment
       ↓
Create shipment
```

If shipping fails:

```text
Shipping ✗
   ↓
Checkout says:
"Refund payment"
   ↓
Payment
   ↓
"Release inventory"
   ↓
Inventory
```

---

## Why this matters for our repo

Online Boutique's `checkoutservice` is already an **orchestrator**:

```text
                 Checkout
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
     Cart        Payment       Shipping
```

But **orchestration ≠ Saga**.

The important distinction is:

> An orchestrator coordinates calls.  
> A Saga coordinates calls **with explicit recovery/compensation semantics for partial failure**.

That's a distinction I want you to keep.

Our current Online Boutique checkout is useful for showing us the **problem**, but it's a simplified demo rather than a full production Saga implementation.

### The mental model

```text
Distributed transaction
        ↓
Can't use one DB transaction
        ↓
Split into local transactions
        ↓
Each step commits independently
        ↓
Failure creates partial state
        ↓
Compensating actions restore business consistency
        ↓
             SAGA
```

And this connects directly to the question we raised earlier:

> **"What happens if payment succeeds but shipping fails?"**

That's exactly the kind of question that leads you to Saga.