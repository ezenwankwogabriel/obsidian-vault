### Why we're using an advisory lock

Imagine a user is allowed to have a maximum of **3 trusted contacts**, and currently has 2.

Two requests arrive at almost the exact same time:

```text
Request A: Add Alice
Request B: Add Bob
```

Without a lock, both requests could check the database and see:

```text
"User currently has 2 contacts."
"2 is less than the limit of 3, so I'm allowed to add one."
```

Then both add their contact:

```text
2 + Alice + Bob = 4 contacts
```

We've broken the 3-contact limit.

An **advisory lock** is basically a database-level mutex.

Before checking the limit, we say:

```text
"Lock this user's account."
```

So the requests are forced to happen one after another:

```text
Request A
   ↓
Lock user
   ↓
Check → 2 contacts
   ↓
Add Alice → 3 contacts
   ↓
Commit / unlock
   ↓
Request B gets the lock
   ↓
Check → 3 contacts
   ↓
Reject: limit reached
```

The important part is that we're locking **per user**, not the entire database. So if User A is adding a contact, User B can still add one at the same time.

In simple terms:

> **The advisory lock prevents two requests from simultaneously making a decision based on the same old database state.**