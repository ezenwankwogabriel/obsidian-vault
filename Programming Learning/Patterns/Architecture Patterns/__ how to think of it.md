
```
ARCHITECTURE
├── Hexagonal
├── Onion
└── Clean

DESIGN PATTERNS
├── Mediator
├── Repository
├── Factory
├── Strategy
└── Adapter

APPLICATION PATTERNS
├── CQRS
├── Saga
└── Event Sourcing

COMMUNICATION
├── HTTP
├── gRPC
├── RabbitMQ
└── Kafka

TECHNIQUES
├── Dependency Injection
└── Interfaces
```

Think of architecture as several questions.

### Question 1: "How do I organize my code?"

**Layered**

```
Controller
Service
Repository
```

**Vertical Slice**

```
CreateOrder
CancelOrder
GetOrder
```

---

### Question 2: "How do I protect my business logic?"

**Onion**

```
Infrastructure
     ↓
Application
     ↓
Domain
```

**Clean**

```
Outside → Inside
```

**Hexagonal**

```
Adapters → Core ← Adapters
```

These three are very closely related philosophically.

---

### Question 3: "How do I separate reads and writes?"

**CQRS**

```
Commands → Write
Queries  → Read
```

---

### Question 4: "How do components communicate?"

**Synchronous**

```
A → HTTP → B
```

**Event-driven**

```
A → RabbitMQ → B
```

---

### Question 5: "How do I split the entire system?"

**Microservices**

```
User Service
Order Service
Payment Service
```