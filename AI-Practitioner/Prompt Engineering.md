An enhanced prompt consists of:
- Instructions
- Context
- Input Data
- Output Indicator

### Negative Prompting:

Explicitly instruct the model on what not to include or do in its response
This helps to:
- Avoid unwanted content - explicitly states what not to include
- maintain focus
- enhance clarity

### Prompt Performance Optimization
Temperature: how accurate the model response should be (from low (very accurate) to high(more verbose))

Top P: how accurate the next word options should be

Top K: the number of words to consider for the next word

Prompt Latency isn't affected by Top P, K, and Temperature

### Prompt Engineering Techniques
#### Zero Shot Prompting
Presents a task to the model without providing examples or explicit training for that specific task
Here, you fully rely on the models general knowledge, depending on the capability of the FM for good results

#### Few Shots Prompting
Provide an example of a task to the model to guide its output. 
We provide a "few shots" to the model to perform the task

With one example, that is called **One Shot or Single Shot Prompting**

#### Chain of thought prompting
Here, you divide the task into a sequence of reasoning steps, leading to more structure and coherence
Use sentences like: "think step by step"
mostly for when the request requires several steps.
```
first, describe the settings and the dog
then, introduce the mystery
next, show the dog ...
finally, uncover how big the dog is

write a short story following this play. think step by step
```

This can be combined with zero-shot or few-shot prompting

#### Retrieval augmented generation (RAG)
Here we combine the models capability with external data sources to generate a more informed and contextually rich response

### Prompt Templates
The idea is that we want to simplify and standardize the process of generating Prompts.
Uses place holders the user has to fill in

This is subject to an attack: Ignorning the prompt template attack. A prompt template injection

You can protect from this by giving explicit instructions to ignore any unrelated / potential malicious content