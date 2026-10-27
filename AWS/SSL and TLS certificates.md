SSL ––> Secure Socket Layer: used to encrypt connections
TLS ––> Transport Layer Security: newer version

Public ssl certs are issued by CA (certificate authorities) such as Comodo, GoDaddy, Letsencrypt, ...

Load balancers uses an X.509 certificate (SSL/TLS server certificate)
You can mange these certificate using ACM (AWS Certificate Manager)
You can upload your own cert to ACM

## SNI (Server Name Indication)
This solves the problem of loading multiple SSL certs onto one web server (to serve multiple websites). Requires the client to indicate the hostname of the target server in the initial SSL handshake
Only works for ALB, NLB, CloudFront. doens't work with CLB

Client --> ALB [SSl Cert 1, SSL Cert 2] ––> [Target group for SSL 1, Target group for SSL 2]
