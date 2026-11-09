SSL ––> Secure Socket Layer: used to encrypt connections
TLS ––> Transport Layer Security: newer version

Public ssl certs are issued by CA (certificate authorities) such as Comodo, GoDaddy, Letsencrypt, ...

Load balancers uses an X.509 certificate (SSL/TLS server certificate)
You can mange these certificate using ACM (AWS Certificate Manager)
You can upload your own cert to ACM

## SNI (Server Name Indication)

This is an extension that allows a client specify which hostname it wants to connect to during the TLS handshake.

It solves a problem that occurs when multiple HTTPS websites share the same IP address or load balancer

Imagine you host 3 websites behind one Application Load Balancer:
`example.com`, `shop.example.com`, `api.example.com`
They all use HTTPS to share same ALB. Each website has a different SSL/TLS certificate

When a browser connects to the ALB, it needs the correct cert for the hostname it requested

*Ideally TLS handshake happens before HTTP request is sent*. Server therefore needs to know which hostname the client wants before selecting a certificate.

SNI solves this by including the requested hostname in the TLS ClientHello message

This solves the problem of loading multiple SSL certs onto one web server (to serve multiple websites). Requires the client to indicate the hostname of the target server in the initial SSL handshake
Only works for ALB, NLB, CloudFront. doens't work with CLB

Client --> ALB [SSl Cert 1, SSL Cert 2] ––> [Target group for SSL 1, Target group for SSL 2]
