A list of numbers that represent the meaning of some data, usually text.
example: "How can i find my appointment"
might be [0.21, -0.73, 0.44, 0.09, ...]

*key: texts with similar meanings tend to have vectors that are close together*

for example:
```
A: "How do I cancel my appointment?"
B: "I need to call off my doctor's appointment."
C: "What time does the clinic open?"
```
might get something conceptually like
```
             vector space

                 B
                /
               /
              A


                              C
```
A & B are close becasue they mean roughly same thing
C is further away because its about opening hours

## the need for RAG
picture our medical SaaS has these documents
```
doc1:
"Patients may cancel appointments up to 24 hours
before the scheduled appointment."

doc2:
"Appointments can be rescheduled through the patient portal."

doc3:
"The clinic operates Monday through Friday from 8am to 5pm."
```
and user asks
```
"Can I call off my doctor's appointment tomorrow?"
```
we need to find `doc1`. but the words don't match exactly. the document says cancel appointments while the user is saying "call off doctor's appointment"

Making traditionaly keyword search can struggle with this.
Embedding allows us to search based on semantic similarity


## RAG pipeline
```
                 DOCUMENT INGESTION
                 
Document
   ↓
Split into chunks
   ↓
Embedding model
   ↓
Vector
   ↓
PostgreSQL + pgvector
```

then when user asks something
```
                 USER QUESTION

"Can I cancel tomorrow's appointment?"
                 ↓
          Embedding model
                 ↓
            Query vector
                 ↓
       Similarity search
                 ↓
          Top 3 chunks
                 ↓
              LLM
                 ↓
             Answer
```

Two different AI jobs happen here:
- Embedding model
	- This answers what existing pieces of text are semantically related to the question
- LLM
	- answers: given these relevant pieces of info, formulate a useful answer for the user


## Embeddings aren't the LLM
given a request to an llm `client.responses.create(...)`. this is a generative LLM
takes input -> llm -> generated text

An embedding model does this instead
```
text
 ↓
embedding model
 ↓
[0.123, -0.44, 0.82, ...]
```
it doesn't answer the user, but produces mathematical representation allowing us perform similarity search

## what is "similarity" in this context
```
"cancel appointment"

[0.9, 0.8, 0.1]

"reschedule appointment"

[0.8, 0.7, 0.2]

"clinic opening hours"

[0.1, 0.2, 0.9]
```
given the above, (assuming 3 dimensions), the first two are relatively close compared to the third
Vector databases use distance/similarity functions to determine this. using *cosine similarity*

## where pgvector comes in
normally youd have sql commands to create tables with columns
with pgvector, you can additionally store things like:
```
content
embedding
tenant_id
metadata


documents
┌────────┬───────────┬─────────────────────────┬────────────────────┐
│ id     │ tenant_id │ content                 │ embedding          │
├────────┼───────────┼─────────────────────────┼────────────────────┤
│ 1      │ clinic-A  │ Cancel up to 24 hours...│ [0.12, ...]        │
│ 2      │ clinic-A  │ Reschedule through...   │ [0.81, ...]        │
│ 3      │ clinic-B  │ Cancellation policy...  │ [0.14, ...]        │
└────────┴───────────┴─────────────────────────┴────────────────────┘

user question
      ↓
embedding
      ↓
[0.13, ...]
      ↓
Postgres / pgvector
      ↓
"Which vectors are closest?"
      ↓
relevant chunks
```

## multi-tenancy part is crucial
from the above, we have client A and client B with different embeddings. if a user asks: "what is our cancellation policy", we want to return response from his tenancy.
```
find nearest vectors
WHERE tenant_id = current_tenant
```

## why chunking
suppose we upload a 100-page medical policy. we don't want one enormous embedding
instead:
```
100-page document
       ↓
    chunks
       ↓
 ┌─────┬─────┬─────┬─────┐
 │ C1  │ C2  │ C3  │ C4  │ ...
 └─────┴─────┴─────┴─────┘
       ↓
 embedding each chunk
       ↓
 pgvector
```
a question about "cancellation policy" would now return the section related to that part, maybe C38

## The complete system being built
```
                         ┌──────────────┐
                         │    Client    │
                         └──────┬───────┘
                                │
                                ▼
                         ┌──────────────┐
                         │   NestJS API │
                         └──────┬───────┘
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
             ┌─────────────┐        ┌─────────────┐
             │ RAG Service │        │ Agent       │
             └──────┬──────┘        │ Service     │
                    │               └──────┬──────┘
                    ▼                      │
             ┌─────────────┐               │
             │ PostgreSQL  │               │
             │ + pgvector  │               │
             └─────────────┘               │
                                           ▼
                                     ┌─────────────┐
                                     │ LLM Gateway │
                                     └──────┬──────┘
                                            │
                                            ▼
                                      OpenRouter
                                            │
                                      ┌─────┴─────┐
                                      ▼           ▼
                                    Model 1     Model 2
```

```
NestJS
   │
   ├── LLM ──────────────── OpenRouter ── free model
   │
   └── Embeddings ───────── OpenRouter ── LFM2.5-Embedding-350M
                                     
PostgreSQL
     │
     └── pgvector
```