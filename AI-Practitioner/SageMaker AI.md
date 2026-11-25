 For building, training, tuning and deploying ML models at scale.
It is to "Collect & Prepare" data, to "Build & Train ML models" and "Deploy models"

Supervised Algorithms
Unsupervised Algorithms
Textual Algorithms: machine learning on text (NLP, summarization)

## Automatic Model Tuning
a sagemaker ai capability that auto searchs for the best hyperparameter values to improve a machine learning performance
Instead of manually testing diff configs, Sagemaker runs multiple training jobs with diff hyperparameter combo and identifies best-performing config for defined objective metric

Hyperparameters are the settings. Automatic Model Tuning is the process of automatically experimenting with those settings to find the best combination

#### SageMaker Model Deployment Type Comparison

**Real-time inference**: low latency, up to 25mb per record, max 60secs, for instant predictions

**Serverless Inference**: low latency, up to 4mb per record, max 60 sec processing time, short-term inference without infrastructure

**Asynchronous Inference**: Medium-high latency (near real time), up to 1gb payload size, max 1 hour processing time, for large payloads requiring longer processing times

**Batch Transform**: high (mins to hours), up to 100mb per

### Sagemaker Studio
Allows you to do End-to-end ML development from a unified interface
Allows for team collaboration, tune debug, and deploy ML models

---
## SageMaker –– Data Wrangler
This allows you prepare tabular and image data for machine learning
It does data preparation, transformation & feature engineering
Giving you a single interface for data selection, cleansing, exploration, and processing

![[Pasted image 20261124111022.png]]

---
## SageMaker –– Feature Store
**SageMaker Feature Store** is a centralised repository for storing, managing and sharing machine learning features

First, remember:

A _feature_ is an individual input variable that a machine learning model uses to make predictions.

Relatable example

Imagine a bank building several ML models:
- Loan approval prediction.
- Fraud detection.
- Customer credit scoring.

All three models may need information about customers, such as:
- Customer age.
- Average monthly income.
- Transaction frequency.
- Credit history.
- Number of previous loan defaults.

Instead of every team independently collecting, cleaning and preparing these features, the bank stores and manages reusable features in SageMaker Feature Store.

**Feature Store = Where do we store and reuse the data features?**

---
### SageMaker Clarify (bias identification)
**SageMaker Clarify** helps identify potential bias in machine learning datasets and models, and provides explanations for model predictions.

In simple terms, it helps answer two questions:
1. Is my model treating different groups unfairly?
2. Why did my model make a particular prediction?


### **Relatable example**

Imagine a bank builds an AI model to approve loan applications.

After training, you discover that the model approves significantly fewer applications from a particular demographic group.

SageMaker Clarify can help:

- Detect potential bias in the training data.
- Identify bias in model predictions.
- Explain which input features influenced predictions.
- Help developers investigate fairness issues.

You give some task, and clarify evaluates based on the task based on friednliness or humor

You can bring in your own built-in datasets or bring in your own employees.

### **Key capabilities**

| Capability          | Purpose                                                      |
| ------------------- | ------------------------------------------------------------ |
| Bias detection      | Identifies potential unfairness in data and predictions      |
| Explainability      | Explains factors influencing model predictions               |
| Feature attribution | Shows how much individual features contribute to predictions |

**Clarify = Why did the model decide this? Is it biased?**

---
## SageMaker Ground Truth (data labeling)
**SageMaker Ground Truth** is an AWS service used to label datasets for machine learning.

In simple terms, it helps humans or automated systems assign meaningful labels to raw data so that it can be used to train supervised machine learning models.

### **Relatable example**

Imagine you're building an AI model to detect fraudulent transactions.

You have 100,000 transaction records, but the model doesn't know which transactions are fraudulent.

Ground Truth helps you label the data:

|Transaction|Label|
|---|---|
|Transaction A|Fraudulent|
|Transaction B|Legitimate|
|Transaction C|Fraudulent|
|Transaction D|Legitimate|

These labelled examples can then be used to train your fraud detection model.
### **What can Ground Truth label?**

| Data type       | Example                                   |
| --------------- | ----------------------------------------- |
| Images          | Identifying cars, people or objects       |
| Text            | Classifying customer reviews              |
| Video           | Identifying objects across video frames   |
| Audio           | Transcribing or classifying audio         |
| 3D point clouds | Labelling objects for autonomous vehicles |

---
### ML Governance
**SageMaker ML Governance** refers to capabilities that help organisations control, manage, audit and ensure compliance throughout the machine learning lifecycle.

In simple terms, it helps organisations answer:
- Who can access ML models?
- Who approved a model for production?
- Where did the training data come from?
- Is the model compliant with organisational policies?
- Can we audit changes made to a model?

### **Three important ML Governance capabilities**

| Capability      | Purpose                                                                                 |
| --------------- | --------------------------------------------------------------------------------------- |
| Model Dashboard | Provides visibility into deployed models, monitoring status and governance information. |
| Model Cards     | Documents model details, intended use, performance, limitations and risks.              |
| Model Registry  | Catalogues model versions and manages their approval and deployment lifecycle.          |
### **Important components**

**A. SageMaker Model Registry**

Think of this as a version-controlled catalogue of ML models.

- Stores model versions.
- Tracks model metadata.
- Manages model approval status.
- Supports controlled model deployment workflows.

**B. SageMaker Model Cards**

Think of these as documentation or identity cards for ML models.

They document:

- Model purpose and intended use.
- Training data information.
- Evaluation results.
- Performance metrics.
- Limitations and risks.
- Ethical considerations.

**C. SageMaker Model Dashboard**

Think of this as a central monitoring and visibility dashboard.

It helps teams review:

- Registered and deployed models.
- Model monitoring information.
- Model governance status.
- Relevant model metadata.

**D. SageMaker Role Manager**

- Define roles for personas
- Example: data scientists, MLOPs engineers

**E. SageMaker Model Monitor**

- Monitor the quality of your model in prod
- Alerts for deviations in the model quality

---
## SageMaker JumpStart
This is a pretrained FM to find computer vision models; or NLP models
You have large collection of models here more than you'd find in amazon bedrock

In simple terms, instead of building an ML model from scratch, you start with an existing model and adapt it to your needs.

**Relatable example**
Imagine you want to build an AI application that analyses customer reviews.

Without JumpStart:
- Find an appropriate model.
- Set up the ML environment.
- Configure the model.
- Deploy it.
With JumpStart:
- Browse available pretrained models.
- Select a suitable text analysis model.
- Customise it if necessary.
- Deploy it for inference.

---
## Sagemaker Canvas
If you are not a developer and still which to use SageMaker, you use the canvas
Builds ML models using a visual interface (no coding required )

---
## MLFlow on Amazon SageMaker
An opensource etool which helps ML teeams manage an entire ML lifecycle


![[Pasted image 20261124123803.png]]

![[Pasted image 20261124125656.png]]