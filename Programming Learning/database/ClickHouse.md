An opensource, column-oriented DBMS built for real-time analytical processing (OLAP). 
Enables sub-second SQL queries over billions of rows, making it a popular choice for web analytics, log management, metrics collection, and real-time dashboard

Architectural Strengths:
- Column Oriented Storage: Data is stored in columns rather than rows (cutting disk I/O)
- Data Compression
- Computations operate on array of values rather than single cells

```
-- 1. Create a table using the MergeTree engine sorted by timestamp
CREATE TABLE user_events
(
    event_time DateTime,
    user_id UInt64,
    event_type String,
    device String
)
ENGINE = MergeTree()
ORDER BY (event_type, event_time);

-- 2. Insert batch data
INSERT INTO user_events VALUES 
    (now(), 101, 'click', 'mobile'),
    (now(), 102, 'view', 'desktop');

-- 3. Run high-speed analytical aggregation
SELECT 
    event_type, 
    count(*) AS total_events
FROM user_events
WHERE event_time >= now() - INTERVAL 1 DAY
GROUP BY event_type;
```