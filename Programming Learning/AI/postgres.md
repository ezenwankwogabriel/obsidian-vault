```
psql --version

// exec into postgres db
docker exec -it ai-platform psql -U guest -d ai

// connect using posgres url postgresq://username:password@localhost:port/db
psql postgresql://guest:guest@localhost:5432/ai

psql postgresql://guest:guest@localhost:5432/ai

// create extension if not exist
CREATE EXTENSION vector;


SELECT extname
FROM pg_extension
WHERE extname = 'vector';
```

