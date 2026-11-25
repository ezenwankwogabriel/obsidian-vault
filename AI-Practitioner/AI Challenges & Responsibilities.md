## Responsible AI 

**Interpretability** is the ability to understand how an AI/ML model makes decisions or produces predictions.

In simple terms, it answers:
- Why did the model make this prediction?
- Which factors influenced its decision?
- Can humans understand and trust its output

### **Responsible AI principles**

Interpretability is one aspect of responsible AI. Understand these broader principles:

| Principle      | Meaning                                                 |
| -------------- | ------------------------------------------------------- |
| Fairness       | Avoid unfair discrimination.                            |
| Explainability | Make AI decisions understandable.                       |
| Transparency   | Provide visibility into how AI systems operate.         |
| Privacy        | Protect personal and sensitive information.             |
| Security       | Protect models and data against attacks.                |
| Robustness     | Ensure reliable performance under different conditions. |
| Governance     | Establish accountability, controls and oversight.       |

### **The major interpretability trade-offs**

| Trade-off                     | Explanation                                                                               |
| ----------------------------- | ----------------------------------------------------------------------------------------- |
| Accuracy vs Interpretability  | More complex models may achieve better predictive performance but be harder to explain.   |
| Complexity vs Transparency    | Complex architectures can make decision-making less transparent.                          |
| Performance vs Explainability | Additional explanation requirements can introduce computational overhead or constraints.  |
| Automation vs Human Oversight | Automated decisions may be faster, but human review is important for sensitive decisions. |
| Personalisation vs Privacy    | More detailed user data may improve predictions but introduce privacy risks.              |
**Important:** These are potential trade-offs, not absolute rules. A complex model is not necessarily more accurate, and a simpler model is not necessarily less accurate.

### **Techniques for improving interpretability**

Know these for the exam:

| Technique                | What it does                                                                   |
| ------------------------ | ------------------------------------------------------------------------------ |
| Feature Importance       | Identifies which input features influence predictions most.                    |
| SHAP                     | Explains how individual features contribute to a prediction.                   |
| LIME                     | Explains an individual prediction using a simpler, locally approximated model. |
| Decision Trees           | Provides understandable decision paths.                                        |
| Partial Dependence Plots | Shows how changes in a feature affect model predictions on average.            |

---
## Human-Centered Design (HCD) for explainable AI

The goal is that you'd want to design AI systems that give priority for humans' needs

The goal is not just to make an AI model explainable technically, but to make its explanations useful and understandable to humans.

### **Relatable example**

Imagine a bank uses AI to reject loan applications.

A technical explanation might say:

	"Feature contribution: debt-to-income ratio = -0.72."

A human-centred explanation would say:

	"Your loan application was rejected because your existing debt is high relative to your monthly income."

The second explanation is more useful to an ordinary customer.

Considers approach to design AI systems with priorities for human needs

1. Design for amplified decision-making: 
	- You're taking decisions in a stressful or high-pressure env and would like to minimize risk & errors.
	- Should design for clarity, simplicity & usability

2. Design for unbiased decision-making: 
	- to make sure our decision process is free from any form of bias
	- should train decisino-makers to recognize & mitigate bias

3. Design for human and AI learning
	- Cognitive apprenticeship where the AI system must learn from human instructors & experts (like in RLHF)

---
## Gen AI Capabilities & Challenges

![[Pasted image 20261124173130.png]]

---
## Generative AI Security Scoping Matrix
This is a framework designed to identify and manage security risks associated with deploying GenAI apps. 

It classifies your app in 5 defined GenAI scopes from low-high ownership

![[Pasted image 20261124190103.png]]

---
### MLOps
the key here is devops for ML. 