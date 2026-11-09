This is a fully managed service for coordinating tasks across distributed applications.

idea is: SWF makes sure a multi-step workflow progresses correctly, even when individual tasks take a long time, fail or need to be retried

Imagine an insurance company processing a customer's insurance claim.

Processing the claim requires four different tasks:
1. Verify the customer's identity.
2. Assess the damage.
3. Calculate the compensation.
4. Approve or reject the claim.
Each task might be performed by a different application or server.

Who coordinates all these tasks?

That's what Amazon Simple Workflow Service (SWF) does

---
# AWS Step Functions
This solves similar problem, but makes workflow coordination much easier to define and manage.
Instead of writing workflow coordination logic yourself, you define a workflow as a state machine
![[Pasted image 20261030175139.png]]

---
# SQS vs SWF
Amazon SQS is a queue for work

SWF coordinates a workflow, these steps must happen in the correct sequence.