### Short version

> **Dependency Inversion = a design principle.**  
> **Dependency Injection = a technique for implementing that principle.**

---

## 1. Dependency Inversion

The problem:

```
type OrderService struct {
    db *PostgresDB
}
```

Your business code directly depends on PostgreSQL.

```
OrderService
     ↓
 PostgreSQL
```

Dependency Inversion says:

> **Your high-level business logic shouldn't depend directly on low-level implementation details. Both should depend on an abstraction.**

So:

```
type OrderRepository interface {
    Save(order Order) error
}
```

Then:

```
              OrderService
                   ↓
          OrderRepository
             (interface)
                   ↑
                   │
          PostgresRepository
```

The important change is **the direction of the dependency**.

Instead of:

```
Business → PostgreSQL
```

you get:

```
Business → Interface ← PostgreSQL
```

That's **Dependency Inversion**.

---

# 2. Dependency Injection

Now you have this:

```
type OrderService struct {
    repository OrderRepository
}
```

How does `OrderService` get its repository?

You **inject** it:

```
repo := PostgresRepository{}

service := OrderService{
    repository: repo,
}
```

The dependency is being **given to** the object rather than created inside it.

That's Dependency Injection.

---

# 3. The easiest comparison

Think:

### Dependency Inversion

**What should I depend on?**

> "I should depend on an abstraction, not PostgreSQL."

### Dependency Injection

**How do I get that dependency?**

> "I'll give the repository to the service from outside."