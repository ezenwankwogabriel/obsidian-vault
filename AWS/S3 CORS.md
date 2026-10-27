Cross Origin Resource Sharing. Controls websites that are allowed by a browser to make cross-origin requests to your s3 bucket.
given you have an app that uses s3 to make uploads directly on an s3 bucket, CORS determines whether the browser is allowed to make those uploads under the bucket's CORs rules

### Settings to remember
AllowedOrigins: Which origins may make requests (https://app.com)
AllowedMethods: Which HTTP methods are permitted (GET, PUT, POST)
AllowedHeaders: Which request headers are allowed (Content-Type, Authorzation)
ExposeHeaders: Which response headers browser Javascript may read (ETag)

## Preflight request:
Some cross-origins requests trigger a browser OPTIONS request before the actual request.
This is like asking the origin whether the origin, method and headers are allowed:
- S3 responds with the relevant cORS headers
- Browser proceeds with the actual upload if preflight succeeds
- if the preflight fails, the browser blocks the actual request
Not every cross-origin triggers a preflight.

CORS is enforced by browsers, not replacement for S3 bucket policies or IAM
CORS is configured on the S3 bucket, rather than on the EC2 instance running the backend