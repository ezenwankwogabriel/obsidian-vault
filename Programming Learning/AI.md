we'd be working on AI healthcare operations platform
```
                    ┌──────────────┐
                    │    Client    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  NestJS API  │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │ AI Agent     │
                    └──┬────┬───┬──┘
                       │    │   │
                       ▼    ▼   ▼
                     RAG  Tools LLM
                       │    │
                       ▼    ▼
                  pgvector  APIs
                       │
                       ▼
                    Postgres

Document ingestion:
API → Queue → Worker → Chunk → Embed → pgvector
```

- we initiate a nestjs project
- install deps: openai, @nestjs/config, pg, class-validator, class-transformer