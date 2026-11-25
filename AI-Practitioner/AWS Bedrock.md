## Generative AI
This is a branch of artificial intelligence that focuses on creating new content based on patterns learned from existing data

Unlike traditional AI, which primarily classifies, predicts or recognises information, Generative AI produces new outputs.

Examples: 
- Text: ChatGPT generating responses.
- Images: Creating images from text descriptions.
- Audio: Generating speech or music.
- Code: Generating software code.
- Video: Creating videos from prompts.

You feed them so many of the data, say dog and cartoon images, and if you ask to create a dog that looks like a cartoon, it can do so

The flow involves:
- Training data (text, image, audio, code, ...) sent to
- Foundation Model (learns patterns in data) then to
- prompt / input, then
- generated output (text, image, audio, code)
![[Pasted image 20261120083001.png]]

## Foundation Model
They can adapt to Text and image generation, summarizatoin, information extraction, can be a chatbox, can answer any question

In order to generate data, we need to have a Foundation Model
These are trained on a wide variety of input data and cost tens of millions of dollars to train
ex: GPT-4o is the foundation model behind ChatGPT

Some of these models are free, and others are paid for

## Large Language Model
A type of AI designed to generate human-like text. An example is GPT-4 (ChatGPT / Open AI)
They are trained on large corpus of text data

Can perform langauge-related tasks such as: Translation, Summarization, Question answering,  Content Creation ...

- You give a prompt
- Model leverages existing content learnt to generate new content
- The generated text is non-deterministic (not same response for same prompt)

### Generative AI for Images from Text Diffusion Models (ex: Stable diffusion)
**Diffusion models** are Generative AI models that create images by learning how to reverse a process of gradually adding noise to images.

Examples include **Stable Diffusion**, DALL·E and Midjourney.

### **How diffusion models work**

The process has two main stages:
1. **Forward diffusion (training):** The model learns by progressively adding random noise to training images until they become almost completely noisy.
2. **Reverse diffusion (generation):** The trained model learns to gradually remove noise, reconstructing an image based on a text prompt.
![[Pasted image 20261120085316.png]]

### **How text influences image generation**

A user provides a prompt, for example:

_"A futuristic city at sunset, photorealistic, cinematic lighting."_
The model:
- Converts the text into numerical representations (embeddings).
- Uses those representations to guide image generation.
- Gradually removes noise while producing an image that matches the prompt.


---
# Amazon Bedrock
A fully managed service from AWS, used to build generative AI applications

---
## Amazon Titan
A high performing Foundation Models directly from AWS
Can do Image, Text, multimodal model choices

titan vs llama vs claude vs stabble diffusion
![[Pasted image 20261113164626.png]]


---
### Custom Models
Customize models using available customization processes. Access and manage custom models based on your region's capabilities.
#### 1\. Reinforcement fine-tuning - _new_
Build more accurate models using reward-based learning with less manual effort. Ideal for complex, multi-step, reasoning tasks.
*How it works*
- Step 1 - Choose model
- Step 2 - Add input data
- Step 3 - Add reward function

Here you improve FM model using feedback based learning. You provide the input data (training data –– prompts). Then define a Reward Function  to evaluate the responses (output) and judge which responses are good. So the model knows what a good and bad response is

![[Pasted image 20261120093104.png]]

For the reward function:
	you can use Lambda if the task is objective (mathematical)
	for subjective task, use another model to "judge" by providing evaluation instructions (text)
The model learn iteratively from reward functions output scroes and will try to achieve high scores over time

```
example: tech customer support chatbot
sample customer prompt: my app is running very slow
judge model instruction: have empathy, and run diagnosis with the user

responses:
- restart the app (score 5.0)
- that sounds frustrating, i can help you figure this out ... (score 9.0)
- please open a support ticket and attach logs A, B, and C (score 2.0)
  
The judge model provides a score for each of them over 10
```

#### 2\. Supervised fine-tuning
Adapts pre-trained models to specific tasks using labeled data to improve performance in specific domains.
*How it works*
- Step 1 - Choose model
- Step 2 - Add input data

In Fine-tuning a model. the idea is that you want to:
- Adapt a copy of a foundation model with your **own data**
- fine-tuning will change the weights of the base foundation model
- training data must: "adhere to a specific format", "be stored in amazon s3"

*note:* not all models can be fine-tuned

2.1\ Supervised fine-tuning
The model is further trained on a particular field or area of knowledge. 
It uses **labeled examples** that are **input-output pairs**
example: 
```
prompt: "Who is Stephane Mareek?"
completion: "He is an aws instructor who dedicates his time to make best resource ..."
```
You provide many of these labeled examples that supervised fine-tuning expectes


So for **Supervised fine tuning**, you provide input and output prompt,
But for **Reinforcement fine tuning**, you provide input, and judge instruction, and weigh the generated output using a reward function
#### Distillation
Here we want to make models faster and smaller.
Up to 75% less expensive than original models, and a decrease in accuracy
Creates smaller, faster models by using a large teacher model to train a specialized student model with your invocation logs or generated training data.
The outcome is a smaller model that behaves similary with the original model

### Options for running a model
1\. On demand (price per token)
2\. Purchase provisioned throughput (billed per month)

___
## Amazon Bedrock - Evaluating a Model
### Automatic Evaluation
This evaluates a model for quality control
Some built in task types: text summarization, question & answer, text classification, open-ended text generation
Then you bring in your own prompt dataset / use built-in curated prompt datasets

There are benchmark questions (yours or from aws), then benchmark answers (these are ideal answers to the benchmark questions). 
Then allow the model to evaluate. Submit the questions to the model to be evaluated
This generates some answers by some Gen AI model.
You compare both (response + your benchmark answers) using a judge model to give a grading score.

Using a **benchmark dataset** allows you to very quickly detect any kind of bias & potential discrimination against a group of people

### Human Evaluation
You have same benchmark questions and answers
Then some human employees from the work team evaluate the generated answers from the benchmark answers. then give a grading score

___
## Automated Metrics to evaluate an FM
1\. ROUGE: Recall Oriented Understudy for Gisting Evaluation (Text Summarization)
This is an evaluation metric used to measure the quality of AI-generated text, particularly summaries by comparing generated text against a human-written reference
Suppose:
- **Reference summary:** "The cat sat on the mat."
- **AI-generated summary:** "The cat is sitting on the mat."
ROUGE compares overlapping words or word sequences between the two texts.
#### **Types of ROUGE**

| Metric  | What it measures                                    |
| ------- | --------------------------------------------------- |
| ROUGE-1 | Overlap of individual words (unigrams).             |
| ROUGE-2 | Overlap of consecutive word pairs (bigrams).        |
| ROUGE-L | Longest Common Subsequence (LCS) between the texts. |
- ROUGE is commonly used to evaluate text summarisation.
- Higher ROUGE scores generally indicate greater overlap with reference text.
- ROUGE does not necessarily measure factual correctness, meaning or overall quality.
- It is a reference-based evaluation metric

2\. BLEU: Bilingual Evaluation Understudy (Machine Translation)
**BLEU** is an evaluation metric used to assess machine-generated text, particularly machine translation, by comparing it against human-written reference translations.
- Measures overlapping words and word sequences (n-grams).
- Uses precision to measure how much generated text matches the reference.
- Includes a brevity penalty to discourage excessively short translations.
- Higher BLEU scores generally indicate better matching.

3\. BERTScore
**BERTScore** evaluates generated text by measuring semantic similarity between generated text and reference text using contextual embeddings from transformer models such as BERT.
Unlike ROUGE and BLEU, it considers the meaning and context of words rather than relying only on exact word matches.
- Uses contextual embeddings.
- Compares semantic similarity between tokens.
- Uses precision, recall and F1-score.
- Can recognise similar meanings even when different words are used.

**Exam memory trick:**

- ROUGE → Text summarisation.
- BLEU → Machine translation.
- BERTScore → Semantic similarity using contextual embeddings.

___
## FM Model Evaluations
- Automatic: 
	- Programmatic: Evaluates performances using just the model & metrics you select
	- Model as a judge: A pre-trained model evaluates your model's responses using metrics you've selected
- Human:
	- AWS Managed work team: Use aws curated work team to evaluate responses from up to 2 models
	- Bring your own work team: Evaluate responses from up to 2 models using your own work team

___
## RAG & Knowledge Base
### RAG: Retrieval Augmented Generation
This allows a foundation model to reference a data source outside of its training data without being fine tuned?
Relies on a data source, likely Amazon S3
Bedrock builds a knowledge base from it
Picture having actual core data in an external source that can be referenced.
You create a vector database as a knowledge base, this would be used for retrieving 

RAG Vector Databases:
- OpenSearch: Search & analytics db real time similarity queries.
- Aurora: Relationaly db on AWS
- Neptune: graph db enabling high performance graph analytics & graph-based RAGS (GraphRAG)
- S3 Vectors: Cost-effective and durable storage with sub-second query performance
- Mongodb
- Redis
- Pinecone

RAG Data Sources
- Amazon S3
- Confluence
- Microsoft SharePoint
- Salesforce
- Web pages (your website, social media feed)


![[Pasted image 20261120173914.png]]

---

## Amazon Nova
This is a family of foundation models (FM) developed by AWS for Gen AI apps. and available through Bedrock.

Premier: the most capable of Amazon's multimodal models for complex reasoning tasks & used as best teacher for distilling custom models
Pro: best combination of accuracy, speed, and cost for wide range tasks
Lite: very low-cost multimodal model that's lightning fast for image processing, video and text inputs
Micro: a text-only model deliverying the lowest latency responses at very low cost

Nova Canvas: a state of the art image generation model
Nova Reel: a state of the art video generation model

Nova Soni - conversational speed understanding and generation in multiple languages

### **Main Amazon Nova model families**

| Model family | Primary purpose                                                 |
| ------------ | --------------------------------------------------------------- |
| Nova Micro   | text-only model. Fast, low-cost text generation.                |
| Nova Lite    | Low-cost multimodal processing. image/text/video processing     |
| Nova Pro     | More advanced reasoning and complex tasks.                      |
| Nova Premier | High-capability model for complex tasks and model distillation. |
| Nova Canvas  | Image generation and editing.                                   |
| Nova Reel    | Video generation.                                               |
| Nova Sonic   | Speech-to-speech interaction.                                   |
| Nova 2       | Newer generation of Nova models with expanded capabilities.     |
![[Pasted image 20261120184541.png]]