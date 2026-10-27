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


