## AI, ML, Deep Learning and GenAI

![[Pasted image 20261123100246.png]]

![[Pasted image 20261123100350.png]]

### Machine Learning
**ML** is a type of AI used to build methods that allow machines to learn
Can now make predictions based on data used to train the model
In ML we don't explicitly program the rules, we just give data, and the algorithm would create its own model for using the data

### ML Models (according to how they learn and their usecase)

- **Supervised learning models**: Learn from labelled data
	- Regression: predict continuous numerical values
	- Classification: predicts categories or classes

- **Unsupervised learning models**: Learn from data without predefined labels

- **Semi-supervised learning models**
	Combines labelled and unlabelled data. Used when labelling is expensive
	ex:
	 1,000 labelled images of cats and dogs
	 100,000 unlabelled images
	
- **Reinforcement learning models**
	Learns by interacting with an environment and receiving rewards/penalties

- **Deep learning models**
	A subset of ML using artificial neural networks with multiple layers

- **Generative AI models**
	Generate new content rather than simply predicting a label or numerical value
	examples: GPT, Claude, Gemini, Stable Diffusion and Llama


![[Pasted image 20261125160731.png]]



### **Deep Learning**
A subset of ML, we use a concept of neurons and synapses (like our brain) to train a model

With this, you're able to process more complex patterns in the data than traditional ML

This requires GPU (graphical processing unit) as it processes a large amount of input data

### **Neural Networks**
This is a ML models inspired by the how the human brain processes information.
Consists of connected units called *neurons* that learn patterns from data to make prediction or generate outputs.

In simple terms, it's a system that learns from examples and uses what it has learned to make predictions or decisions.

Imagine teaching a computer to recognise cats.

You show it thousands of pictures:

- Some contain cats.
- Others contain dogs.
- Each picture has a correct label.

The neural network gradually learns patterns that distinguish cats from dogs, such as shapes, ears, eyes and other visual features.

Eventually, you give it a new picture, and it predicts whether it contains a cat or dog.

**Artificial Intelligence → Machine Learning → Deep Learning → Neural Networks**

Neural networks are the foundation of many modern AI systems, including deep learning, LLMs, image recognition and Generative AI.

**A neural network learns by adjusting weights and biases based on training data.**

---
### **Gen AI**
A subset of Deep Learning. A multi-purpose foundation model backed by neural networks.
Can be fine-tuned if necessary

These Gen AI have a Transformer Model (LLM) allowing the processing of sentences as a whole instead of word by word
It is faster and more efficient for text processing

---
### **Multi-modal models (ex: gpt-4o)**
Have multiple type of inputs and multiple types of output. 
ex: can take a mix of audio, image and text and output a mix of video, text for example

___
***GPT** stands for ==**generative pre-trained transformer**==.*  (language)
	To generate human text or computer code based on input prompt

***BERT*** **==Bidirectional Encoder Representations for Transformers==** (language)
	Similar intent to GPT but reads the text in two direction

**RNN** **==Recurrent Neural Network==**  (speech/time-series)
	meant for sequential data such as time-series / text, useful in speech recognition, time series prediction

***ResNet*** **==Residual Network==**  (images)
	Deep convolutional Neural Network (CNN) for image recognition tasks, object detection, facial recognition

***SMV*** **==Support Vector Machine==**
	ML algorithm for classification and regression

***WaveNet***  (audio)
	model to generate raw audio waveform, used in Speech Synthesis

***GAN*** **==Generative Adversarial Network==** (data aug) [creating new image/video/sound data]
	models used to generate synthetic data such as images, videos or sounds that resemble the training data. used for data augmentation

***XGBoost*** **==Extreme Gradient Boosting==**  
	an implementation of gradient boosting algorithm used for regression and classification on tabular data

---
## Training Data
To train models, we must have good data
### Labeled vs Unlabeled Data
 ![[Pasted image 20261123105537.png]]

### Structured Data

![[Pasted image 20261123105706.png]]

### Unstructured Data
![[Pasted image 20261123105816.png]]

---
### Feature Engineering (input variables manipulation)
This is the process of selecting, transforming, creating or modifying input variables (features) to improve a machine learning model's performance

In simple terms, it involves preparing raw data into meaningful information that a model can learn from more effectively

#### Techniques:
**Feature Creation/Extraction**: extracting useful info from raw data such as age from date of birth

**Feature Selection**: selecting a subset of relevant features like choosing important predictors in a regression model
	Imagine having 15 features for a house, only 5 is needed for predicting house prices, so you select only those

**Feature Transformation**: Changing the format / scale of existing data so the model can work with it more effectively
	imagine dataset contains values like 250sqmeters, #150,000,000, ...
	you can transform these numerical values through normalization so they fall within a common range, such as 0 to 1.
	example:
		100 => 0.0
		200 => 0.5
		400 => 1.0
		
		lekki => 1
		ikeja => 2
		yaba => 3

---
## Supervised Learning
ML approach where a model learns from labelled training data. Each training example contains an input & corresponding correct output(label)

The goal is to learn the relationship btw inputs & output so model can make predictions on new, unseen data.

***Classification Type***: the output is a category (spam vs not spam. Dog, Cat, Giraffe)
***Regression Type***:  the output is a numerical value (predicting housing price)

**Training Set**: are used to train the model
	usually 60-80% of the dataset
	ex: 800 labeled images from dataset of 1000 images

**Validation set:** is used to tune model parameters and validate performance. they determine if the training set works effectively
	usually 10-20% of the dataset
	ex: 100 labeled images for hyperparameter tuning

**Test set:** Used to test the final model performance
	usually 10-20% of the dataset
	ex: 100 labeled images to test the models accuracy
	a cat input results to a cat response

---
## Unsupervised learning
ML approach where a model learns from unlabelled data. No predefined correct answers / output labels
The goal is to discover hidden patterns, relationships or groups within the data

**Techniques**

| Technique                | Purpose                                                                              | Example                                                                                                         |
| ------------------------ | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| Clustering               | Groups similar data points<br>                                                       | Customer segmentation<br>some customers love pizza, others fruits, ...<br>can now send diff marketting campaign |
| Dimensionality reduction | Reduces the number of features while preserving useful information                   | Simplifying complex datasets                                                                                    |
| Association              | Discovers relationships between items                                                | Customers who buy bread also buy butter                                                                         |
| Anomaly detection        | Identifies unusual patterns                                                          | Detecting suspicious transactions using Isolation technique                                                     |
| Semi-supervised          | use a small amount of labeled data & large amount of unlabeled data to train systems |                                                                                                                 |

---
## Self Supervised Learning
an ML approach where a model learns from unlabeled data by generating its own training targets from the data itself

Instead of humans manually labelling examples, the model creates a learning task from the input data. Important for training modern foundation models and LLMs

### **How it works**

Example: Training a language model.

Original sentence: "The cat is sitting on the mat."

The model receives: "The cat is sitting on the ___."
It must predict the missing word: **mat**.

The original data provides the answer, so human labelling isn't required.

### **Real-world examples**
- **BERT:** Uses masked language modelling to learn contextual relationships.
- **GPT models:** Use next-token prediction to learn language patterns.
- **Image models**: Learn visual representations from image relationships without manual labels.

### **Common techniques**

| Technique                       | How it works                                         |
| ------------------------------- | ---------------------------------------------------- |
| Masked Language Modelling (MLM) | Hides words and asks the model to predict them.      |
| Next-token prediction           | Predicts the next token in a sequence.               |
| Contrastive learning            | Learns which data examples are similar or different. |
| Denoising                       | Reconstructs original data from corrupted input.     |

---
## Reinforcement Learning
An ML approach where an agent learns by interacting with an environment, taking actions and receiving rewards/penalties based on its behavior.

The objective is to learn which actions maximise the total cumulative reward over time.

The reward/penalty is a feedback that helps the agent improve future decisions

**Techniques**

| Technique            | Description                                                      |
| -------------------- | ---------------------------------------------------------------- |
| Q-Learning           | Learns the expected value of taking actions in particular states |
| Deep Q-Network (DQN) | Uses neural networks to estimate Q-values                        |
| Policy Gradient      | Directly optimises the agent's action-selection policy           |
| Actor-Critic         | Combines policy learning (actor) with value estimation (critic)  |

---
## Reinforcement Learning from Human feedback (RLHF)
Aligns language model with human preferences/feedback
- Pretrain a foundation model
- collect human feedback on model responses
- train a reward model to represent human preferences
- use reinforcement learning to optimise the model's response against the reward model
ex: you're grading text translations from "technically correct" to "Humanly correct (it makes sense to a human)"
### **Compare the four learning approaches**

| Learning type   | How it learns                         |
| --------------- | ------------------------------------- |
| Supervised      | Learns from labelled examples         |
| Unsupervised    | Discovers patterns in unlabelled data |
| Self-supervised | Creates its own training targets      |
| Reinforcement   | Learns through actions and rewards    |
ex: to build an intenal company knowledge chatbot using rlhf
- Data collection
	- set of human generated prompts & responses are created
	- ex: where is the location of the hr department in boston
- Supervised fine-tuning of a language model
	- finetune an existing model with internal knowledge
	- the model creates responses for human-generated prompts
	- response are mathematically compared to human-generated answers
- Build a separate reward model
	- Humans indicate which response they prefer for same prompt (what gpt does[option A, and option B])
	- The reward model can now estimate how a human would prefer a prompt response
- Optimize the language model with the reward-based model
	- use the reward model as a reward function for RL
	- This now becomes 
![[Pasted image 20261123144049.png]]

---
## Some ML Concepts
### Model Fit, Bias and Variance

### Model Fit:
For managing performance of a model

**OverFitting**: when a model performs well on training data, but doesn't on evaluation data (new data not part of existing dataset)

**Underfitting**: the model is performing poorly on the training data. 

**Balance**: The goal for model fitting. performs okay on training data and evaluation data

***bias and variance help us understand whether a model is learning too little or becoming to sensitive to its training data***
### Bias (correctedness)
This is the error between predicted and actual value. This occurs when a model makes overly simplistic assumptions about the data

Imagine predicting house prices using only the number of bedrooms, ignoring location, house size and other important factors.

*The model may consistently make inaccurate predictions because it hasn't captured the actual relationships in the data.* This is called **high bias**

**High Bias**: when the model doesn't closely match the training data. 
The model is too simple, fials to capture important patterns, performs poorly on training data and associated with underfitting

ex: a linear regression function on a non-linear dataset. This is considered ***underfitting**

![[Pasted image 20261123145456.png]]

To reduce a bias, you use a more complex model, or increase the number of features

### Variance (changing responses)
**Variance describes how sensitive a model is to changes in its training data.**

This represents how much the performance of a model changes if trained on a diff dataset which has a similar distribution

**High Variance**: The model is very sensitive to changes in the training data. A case when overfitting. It performs well on training data, but poorly on unseen test data

To reduce variance, consider less, more important features, and split into training and test data sets multiple times

![[Pasted image 20261123145641.png]]

![[Pasted image 20261123145900.png]]

![[Pasted image 20261123150022.png]]

---
## Model Evaluation Metrics

### Binary Classification
This predicts one of two possible outcomes: **Positive or Negative**

Example: An AI model detecting fraudulent transactions.

| Term                    | Meaning                                                     | Example                                                      |
| ----------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **True Positive (TP)**  | Model predicts Positive, and the actual result is Positive. | Fraudulent transaction correctly detected.                   |
| **True Negative (TN)**  | Model predicts Negative, and the actual result is Negative. | Legitimate transaction correctly identified.                 |
| **False Positive (FP)** | Model predicts Positive, but the actual result is Negative. | Legitimate transaction incorrectly flagged as fraud.         |
| **False Negative (FN)** | Model predicts Negative, but the actual result is Positive. | Fraudulent transaction incorrectly classified as legitimate. |
***key***
- ***True** = Prediction was correct.*
- ***False** = Prediction was incorrect.*
- ***Positive/Negative** = What the model predicted.*

So the goal is to have either "TP or TN". "FP and FN" are incorrect.

### Binary classification example
Metrics:
- Precision: this is best when false positives are costly
- Recall: Best when false negatives are costly
- F 1 score - Best when you want a balance btw precision & recall (in imbalanced datasets)
- Accuracy: For balanced datasets

**AUC-ROC:  Area under the curve-receiver operator curve**
Value ranges from 0-1 (perfect model)
Uses sensitivity (true positive rate) and "1-specificity" (false positive rate)

This shows what the curve for true positive is compared to false positives at varios thresholds. with multiple confusion matrixes

Regression Metrics
![[Pasted image 20261123154601.png]]



![[Pasted image 20261123154847.png]]

---
### Inferencing
From the name, making inference. This is using an already-trained AI model to produce an answer, prediction or result

In simple terms:
- **Training:** Teaching the model using data.
- **Inference:** Using the trained model to produce results.

**Real-time**: 
	using a chatbot and need immediate response. Computers have to make decisions quickly as data arrives
	Speed is preferred here over perfect accuracy
**Batch:**
	Large amount of data is analyzed all at once.
	a large dataset is given to the model and wait for days, minutes or weeks for response.
	Maximum accuracy is required here and not speed

### Inferencing at the Edge
Edge devices are devices with less computing power, closer to where the data is generated, in places where internet connections can be limited

Edge inference is the process of running a trainined ML model directly on/near the device where data is generated, instead of sending all data to a central cloud env for processing.

Examples: smart cameras detecting people, self-driving vehicles recognising obstacles, smartphones recognizing faces

![[Pasted image 20261123190647.png]]
This can be your phone, ... Typically for Small Language Models
	very low latency (faster responses as data doesn't travel out to the cloud)
	low compute footprint
	offline capability
	privacy
alternatively, run the LLM on a remote server, then the edge device does calls over internet to the LLM

---
## Phases of a Machine Learning Project
### The ML lifecycle

1. **Business Problem Identification**
    Define the problem, objectives and success criteria.
    Example: Predict customers likely to default on loans.
2. **Data Collection**
    Gather relevant data from databases, APIs, files and other sources.
3. **Data Preparation**
    Clean data, handle missing values, remove duplicates and transform data.
4. **Feature Engineering**
    Select, create and transform useful input variables.
5. **Model Training**
    Select an algorithm and train it using training data.
6. **Model Evaluation**
    Test model performance using appropriate metrics such as accuracy, precision, recall and RMSE.
7. **Model Deployment**
    Make the trained model available for real-world predictions through endpoints or batch processing.
8. **Monitoring and Maintenance**
    Monitor performance, detect model drift and retrain when necessary.

---
## Hyperparameter Tuning
Hyperparameters are settings defined before or during the training process that influence how a model learns. It improves the model during development (not after development)

**Hyperparameter tuning** is the process of finding the optimal configuration of a machine learning model to improve its performance.

### **Hyperparameters vs Parameters**

This is an important exam distinction.

| Parameters                            | Hyperparameters                                |
| ------------------------------------- | ---------------------------------------------- |
| Learned automatically during training | Set before training or by the training process |
| Weights and biases                    | Learning rate, batch size, epochs              |
| Determine what the model learns       | Control how the model learns                   |
**Example:**

When training a neural network:

- Weights are parameters learned by the model.
- Learning rate determines how much weights change during each update.
- Batch size determines how many training examples are processed before an update.
- Epochs determine how many times the model processes the full training dataset.
- Regularization is adjusting the balance btw simple & complex model. if you wish to reduce overfitting, increase regularization.

To help with Overfitting, you should:
- increase training data size
- early stopping the training of the model (doing more epoch wouldn't help)
- data augmentation (to increase diversity (more coverage) in the dataset)
- adjust hyperparameters (but you can't add them)