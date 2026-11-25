**SageMaker AI focuses on building, training and managing machine learning models. Bedrock focuses on accessing pretrained foundation models and building Generative AI applications.**

### **1. Amazon SageMaker AI**

SageMaker AI is a managed platform for developing the machine learning lifecycle.

You use it when you want to:

- Build custom ML models.
- Train models using your own datasets.
- Tune hyperparameters.
- Deploy models for inference.
- Monitor model performance.
- Manage the ML lifecycle.

**Example:** A Nigerian bank wants to build its own fraud detection model using millions of historical transactions. It uses SageMaker AI to prepare data, train the model, tune it and deploy it.

### **2\. Amazon Bedrock**

Amazon Bedrock is a fully managed service that provides access to foundation models from AWS and third-party providers through APIs.

You use it when you want to:

- Build AI chatbots.
- Generate text, images and other content.
- Implement RAG.
- Build AI agents.
- Apply guardrails to AI responses.
- Customise foundation models for specific tasks.

**Example:** A bank wants to build an AI assistant that answers customer questions using its internal documentation. It uses Bedrock with a foundation model and Knowledge Bases for RAG.

### **3. Quick comparison**

| Feature               | SageMaker AI                   | Bedrock                                               |
| --------------------- | ------------------------------ | ----------------------------------------------------- |
| Primary purpose       | ML development platform        | Generative AI application platform                    |
| Model training        | Train custom ML models         | Use pretrained FMs; supports model customisation      |
| Foundation models     | Can develop and deploy models  | Provides access to multiple FMs                       |
| Hyperparameter tuning | Yes                            | Not its primary purpose                               |
| RAG                   | Can build custom RAG solutions | Managed Knowledge Bases                               |
| AI agents             | Can build custom solutions     | Managed Agents capability                             |
| Model deployment      | Managed ML endpoints           | Managed FM inference APIs                             |
| Customisation         | Extensive ML control           | Fine-tuning and other supported customisation options |

### **4. Important Amazon Bedrock components**

| Component                   | Purpose                               |
| --------------------------- | ------------------------------------- |
| Bedrock Knowledge Bases     | RAG and retrieval                     |
| Bedrock Agents              | Execute multi-step tasks using tools  |
| Bedrock Guardrails          | Filter harmful or unwanted content    |
| Bedrock Model Evaluation    | Evaluate foundation model performance |
| Bedrock Prompt Management   | Manage and reuse prompts              |
| Bedrock Flows               | Visually orchestrate GenAI workflows  |
| Bedrock Model Customisation | Adapt supported foundation models     |

### **5. Exam scenario mapping**

| Scenario                                       | Correct answer          |
| ---------------------------------------------- | ----------------------- |
| Train a custom fraud detection model           | SageMaker AI            |
| Build an enterprise chatbot using Claude       | Bedrock                 |
| Access Amazon Nova and other foundation models | Bedrock                 |
| Perform automated hyperparameter tuning        | SageMaker AI            |
| Implement managed RAG using company documents  | Bedrock Knowledge Bases |
| Build an AI agent that calls external APIs     | Bedrock Agents          |
| Monitor custom ML model drift                  | SageMaker Model Monitor |
| Build and deploy custom deep learning models   | SageMaker AI            |