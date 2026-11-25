### Comprehend
For Natural Language Processing (NLP). Uses ML to understand and analyse text
Identifies meanings, patterns and relationships in written language

supports text, pdf, word, images

**Named Entity Recognition (NER)**: One of comprehends capability, that extracts predefined, general purpose entities like people, places, organizations, dates, .. from text

**Custom Entity Recognition** (CER): To analyze text for specific terms and noun-based phrases
Extract terms like policy numbers, or phrases that imply a customer escalation, ...

___
## Translate
A natural & accurate language translation. Allowing you to localize content - such as websites and applications 

___
### Transcribe
Converts speech into text. Uses deep learning process called automatic speech recognition (ASR) to convert speect to text quickly and accurately. Auto removing PII using Redaction

___
### Polly (opp of transcribe)
Turns text into lifelike speech using deep learning
- Lexicon: define how to read certain pieces of text
	- aws => amazon web service
- SSML: markups for how your text should be pronounced
	- example: "hello, <break/> how are you" (waits)
- Voice Engine: 
- Speech mark: encodes where a sentence/word starts/ends in the audio (helpful for lip-syncing)

___
## Rekognition
A service that finds objects, people, text, scenes in images & videos using ML

It's content moderation auto detects inappropriate, unwanted or offensive content
Bringing down review to 1-5% of total content volume
Is integrated with Amazon A2I (augmented AI) for human reviews

Custom Moderation Adaptors extends rekognition capabilities by providing your own labeled set of images
This is like training the model with your own labeled set of images (100s to 1000s of them) so the model can uniquely identify your logos or anything unique you wish to be captured

Users request an image > chatbot generates one > passes it to Amazon rekognition for review for certain criteria

![[Pasted image 20261124073431.png]]

___
### Amazon Lex
A way to build chatbots quickly for your app using voice or text. Supports multiple languages and integrates with AWS lambda, Connect, Comprehend, Kendra
Can trigger lambda functions (allowing users to use backend services) to fulfil an intent 

___
### Amazon Personalize
To build apps with real-time personalized recommendations
ex: personalized product recommendations/re-ranking, customized direct marketing (ex: user bought gardening tools, provide recommendations for next to buy)
This is same technology used by Amazon.com

This integrates to existing websites, app, to send SMS, ...
Don't need to build, train and deploy ML, can be done in days

**Recipes**: are algorithms in personalize, you must provide the training config first. 
ex: recommending items for users "USER_PERSONALIZATION recipes"

---
### Textract
To extract text, handwriting and data from any scanned documents, images and PDFs using AI and ML

### **Quick comparison** with comprehend

|Feature|Comprehend|Textract|
|---|---|---|
|Primary purpose|Understand text|Extract text and document structure|
|Technology area|NLP|OCR + document analysis|
|Input|Text|PDFs, images and scanned documents|
|Sentiment analysis|Yes|No|
|Entity recognition|Yes|Extracts document information, not general NLP entity analysis|
|Table extraction|No|Yes|
|Form extraction|No|Yes|
|PII detection|Yes|Not its primary purpose|

---
## Kendra
A document search service
Extract answers from within a document (text, pdf, html, powerpoint, ...)

---
## Augmented AI (A2I)
When ML is making decisions but you wish to have human oversight to ensure models work as expected

AI service makes prediction > Sent to A2I > if okay is sent to client else is sent for human review then > sent to client app

**Example:** A bank uses an ML model to approve loan applications. Applications with uncertain predictions are sent to human reviewers for verification.

![[Pasted image 20261124080611.png]]


---
## **Amazon SageMaker AI**

**Amazon SageMaker AI** is a fully managed machine learning platform for building, training, tuning and deploying ML models.

Key capabilities:

- Prepare and process training data.
- Build and train machine learning models.
- Perform hyperparameter tuning.
- Deploy models for real-time or batch inference.
- Monitor model performance.
- Build and manage ML workflows.

**Example:** A bank uses SageMaker AI to train a model that predicts whether a loan applicant is likely to default.

---
## Comprehend Medical and Transcribe Medical
**Transcribe Medical**: Converts medical-related speech to text (HIPAA compliant)
specialized in med terminologies like drug names, procedures, conditions & diseases
ex: voice apps that enable physicians to dictate medical notes 

**Comprehend Medical**: Once from the audio you have text, you can use comprehend to return useful information in unstructured clinical text


---
## HealthScribe (a transcribe feature)

HIPAA service that auto generates clinical notes by analyzing patient-clinician conversations

You send the audio into healthscribe and it sends a clinician note for the conversation

---
## Hardware for AI

Asides the regular hardware for EC2, aws gives us

**AWS Trainium (TRn)** : These are ML chip to perform Deep Learning on 100B+ parameter models
50% cost reduction when training a model

**AWS Inferentia**: ML chip built to deliver inference at high performance & low cost. Inf1 and Inf2 instances for EC2 provided by AWS
Up to 4x throughput and 70% cost reduction

Trn and Inf have the lowest environmental footprint
