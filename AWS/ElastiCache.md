Just as RDS is to get managed Relational Databases, ElastiCache is to get managed Redis or Memcached 
These are in-memory dbs with really high performance and low latency
Helps to make your app stateless 
AWS takes care of OS maintenance/patching/optimization/setup config/monitoring/failover/backup and recovery 


## Security
supports IAM auth for Redis
IAM policies on elasticache are only used for aws api-level security
### Redis auth
	can set password/token on creating a redis cluster
	supports ssl in flight encryption
### Memcached
	supports SASL-based authentication

## Patterns for ElastiCache
- Lazy Loading: all the read data is cached, data can become stale in cache
- Write through: Adds or updates data in cache when written to db (no stale data)
- Session store: store temp session data in cache (using ttl feature)

