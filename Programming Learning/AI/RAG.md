**RAG** stands for **Retrieval-Augmented Generation**. It’s a technique used with large language models (LLMs) to make them more accurate, up-to-date, and grounded in specific knowledge.

Instead of relying only on what the model learned during training, RAG lets the model **retrieve relevant information from an external source** and then use that information to generate an answer.

### How RAG works

1. **Index / store knowledge**
   - Documents, FAQs, manuals, databases, webpages, etc. are split into chunks.
   - Each chunk is converted into a vector embedding and stored in a vector database.

2. **Retrieve**
   - When a user asks a question, the question is also converted into an embedding.
   - The system searches the vector database for the most relevant chunks.

3. **Augment**
   - Those retrieved chunks are inserted intothe prompt given to the LLM.
   - Example prompt:  
     *“Use the following context to answer the question: [retrieved text]. Question: …”*

4. **Generate**
   - The LLM writes an answer using both its general language ability and the retrieved context.

### Simple example

Without RAG:  
**User:** “What is our company’s refund policy for international orders?”
**LLM:** Guesses or says it doesn’t know.

With RAG:  
The system retrieves the exact refund policy document, feeds it to the LLM, and the LLM answers:
“International orders can be refunded within 30 days if the item is unused…”

### Why RAG is useful

- **Up-to-date information**: Can use currentdocuments without retraining the model.
- **Private/specific knowledge**: Works with internal company data.
- **Reduces hallucinations**: Answers are grounded in retrieved sources.
- **Citations**: Can point to the source documents.
- **Cheaper than fine-tuning**: Often easier to update a knowledge base than retrain a model.

### Limitations

- Retrieval quality matters: if it fetches wrong context, the answer may be wrong.
- Chunking and embedding strategy affect performance.
- The model can still misinterpret retrieved text.
- Needs good document organization and maintenance.

In short: **RAG = search + LLM generation**. It gives an LLM a kind of open-book exam, where it first looks up relevant information and then answers based on what it found.%


## STEPS
```
                 INGESTION (parse chunk to get embedding and save to postgres)

Document/chunk
      ↓
   Embedding
      ↓
   PostgreSQL
```

```
                  RETRIEVAL (for a question, get embedding with similar search)

User question
      ↓
   Embedding
      ↓
Vector similarity search
      ↓
Relevant chunks
```

```
                  GENERATION (Actual RAG (answer = Ques + chunks => LLM => answer))

Question + retrieved chunks
             ↓
            LLM
             ↓
           Answer
```


```
TOGETHER

                  RAG
                   │
        ┌──────────┴──────────┐
        │                     │
    RETRIEVAL             GENERATION
        │                     │
 question → embedding     question
        │                     +
        ↓                  context
   vector search              │
        │                     ↓
        ↓                    LLM
   relevant chunks             │
        └──────────┬──────────┘
                   ↓
                 answer
```