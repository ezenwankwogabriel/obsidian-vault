Think of these like recipes for standard problems in real life. You don't invent a new way to bake bread every morning; you follow a proven recipe.

When you realize "there's got to be a better way to do this". you look out for the proven recipe instead of guessing a solution, you find how experienced programmers fixed this 30 years ago. 
you find a pattern.

#### Repository Pattern
This creates an abstraction layer btw business logic and your data source(db, API, FS, ...). Instead of services directly querying db, they interact with a repository.
```
#without repository pattern
@Injectable()
export class UserService{
  constructor() {}
  async findUser(id: string) {
	  return this.userRepository.findOne({
		  where: { id }
	  })
  }
}
```
The above tightly couples business logic to TypeORM
```
#with repository pattern

# define an interface
export interface IUserRepository {
	findById(id: string): Promise<User | null>;
	create(user: CreateUserDto): Promise<User>;
}

# implement repository
export class UserRepository implements IUserRepository {
	constructor(
		@InjectRepository(User)
		private readonly repository: Repository<User>
	) {}
	
	async findById(id: string) {
		return this.repository.findOne({ where: { id } })
	}
	async create(data: CreateUserDto) {
		const user = this.repository.create(data);
		return this.repository.save(user)
	}
}

# user in service
export class UserService {
	constructor(
		private readonly userRepository: UserRepository
	) {}
	
	async getUser(id: string) {
		return this.userRepository.findById(id)
	}
}
```

Now the service doesn't care whether data comes from "postgres, mongo, redis, ..."


#### Strategy Pattern
```
# without strategy pattern

if (provider === "stripe") {...}
else if (provider === "paystack") {...}
else if (provider === "flutterwave") {...}

# with strategy pattern

export interface PaymentStrategy {
	charge(amount: number, currency: string): Promise<string>;
}

# create concreate strategies

export class StripePaymentStrategy implements PaymentStrategy {
	async charge(amount: number, currency: string): Promise<string> {
		return `Stripe charged ${amount} ${currency}`
	}
}

export class PaystackPaymentStrategy implements PaymentStrategy {
  async charge(amount: number, currency: string): Promise<string> {
    return `Paystack charged ${amount} ${currency}`;
  }
}

# Context (service using the strategy) it doesn't really know which strategy is used

export class PaymentStrategy {
	private strategy: PaymentStrategy
	
	setStrategy(strategy: PaymentStrategy) {
		this.strategy = strategy
	}
	
	async pay(amount: number, currency: string) {
		if !this.strategy {
			throw new Error("Payment strategy not set")
		}
		
		return this.strategy.charge(amount, currency)
	}
}

# using it

async function run() {
	paymentService = new PaymentService()
	
	paymentService.setStrategy(new StripePaymentStrategy())
	console.log(await paymentService.pay(100, "USD"))
	
	paymentService.setStrategy(new PaystackPaymentStrategy())
	console.log(await paymentService.pay(100, "NGN"))
}
```


#### Adapter Pattern
Your app has one interface, but an external library or API gives you something different. Instead of rewriting your code everywhere, you "adapt" it

```
# your system expects
sendEmail(to: string, subject: string, body: string): Promise<void>

# third party provider (mailgun) exposes
sendMail(data: { recipient: string; title: string })

# with adapter, define your apps expected interface

export interface EmailService {
	sendEmail(to: string, subject: string, body: string): Proimse<void>
}

# create adapter for the external service

export class MailgunAdapter implements EmailService {
	constructor(private mailgun: MailgunService) {}
	
	async sendEmail(to: string, subject: string, body:string): Promise<void> {
		await this.mailgun.sendMail({...})
	}
}
```


## 4. Observer Pattern

**What it is:** One object notifies many others when something happens.

**Core idea:**

> “When X happens, everyone interested should react.”

**Use when:**

- Events (user created, order placed)
- Logging, notifications, analytics
- Decoupled workflows

**Mental model:**

```
Event → Subscribers react
```

---

## 🧱 Supporting Patterns (quick add-ons)

### Singleton

> “Only one instance exists everywhere.”

Used for:

- config
- database client
- logger

```
One class → One global instance
```

---

### Dependency Injection (DI)

> “Don’t create dependencies, receive them.”

Used for:

- decoupling code
- testing
- scalable architecture

```
Constructor receives dependencies
```

---

# ⚖️ Ultra-fast differentiation (1-liners)

- **Repository** → “How do I access data cleanly?”
- **Strategy** → “Which behavior should I use?”
- **Adapter** → “How do I make this external thing work?”
- **Observer** → “Who should react when something happens?”
- **Singleton** → “Ensure only one instance exists”
- **DI** → “Pass dependencies instead of creating them”