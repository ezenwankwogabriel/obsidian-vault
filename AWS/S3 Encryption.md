# S3 encryption — AWS SAA-C03

The easiest way to understand S3 encryption is to ask two questions:

1. Who encrypts the data and manages the keys?
2. When is the data encrypted and decrypted?

For the exam, the most important distinction is between encryption at rest (data stored in S3) and encryption in transit (data travelling between your application and S3).

## 1. Encryption at rest vs in transit

Encryption at rest

Your file is stored in an S3 bucket in encrypted form. If someone obtains the underlying stored data without the appropriate decryption capability, they cannot simply read it.

Encryption in transit

Your application uses HTTPS/TLS when uploading to or downloading from S3, protecting data as it travels over the network.

Exam clue: If a question asks about protecting stored S3 objects, think server-side encryption. If it asks about protecting data travelling over the network, think HTTPS/TLS.

## 2. The four server-side encryption options

Server-side means S3 performs the encryption before storing your object and decrypts it when an authorised request retrieves it.

1\. SSE-S3 (Amazon S3 Managed Keys) 

Default

S3 manages the encryption keys for you. New objects are automatically encrypted at rest using SSE-S3 by default. You don't need to set up KMS.

Choose it when: You want straightforward encryption at rest with minimal configuration and key-management overhead.

Encryption type is AES-256. You must set header "x-amz-server-side-encryption": "AES-256"

2\. SSE-KMS

AWS Key Management Service

S3 uses a KMS key to protect the data-encryption key. You gain more control over key policies, permissions, rotation and auditing. Gives you control + audit key usage trail using CloudTrail

Must set header "x-amz-server-side-encryption": "aws:kms"

Choose it when: You need finer control over who can use encryption keys, or need to audit key usage. KMS requests can add cost, and users or services need appropriate KMS permissions.

*Limitations*
- You can be impacted by KMS limits
- You can request a quota increase

3\. DSSE-KMS (Dual-layer SSE with KMS) 

Applies two independent layers of server-side encryption [SSE and KMS]

Choose it when: A compliance or security requirement specifically calls for dual-layer server-side encryption. It can cost more than ordinary SSE-KMS.

Must set header "x-amz-server-side-encryption": "aws:kms:dsse"

4\. SSE-C (Customer-provided key)

This can only be done from the CLI, and not from the aws dashboard/console

You supply the encryption key with the request. S3 performs the encryption but does not store that key for you.

Choose it when: You must provide and control the key material yourself. You must securely retain the key and provide it again for subsequent access. Current S3 security settings disable SSE-C for new general-purpose buckets by default, so it must be deliberately enabled when needed.
### How to remember the names

- SSE-S3: S3 manages the keys.
- SSE-KMS: KMS provides more key control and auditing.
- DSSE-KMS: Two independent server-side encryption layers.
- SSE-C: Customer supplies the key with requests.

## 3. What about client-side encryption?

This is different from all four options above.

With client-side encryption, your application encrypts the file before uploading it to S3.

Your application: encrypt file

S3 stores the already-encrypted file

Your application: decrypt after downloading

The application manages the encryption/decryption process and key access. This gives you more control over the data before it reaches S3, but also more responsibility. If you lose the necessary key, you may permanently lose access to the data


## Amazon S3 - Force Encryption in Transit aws:SecureTransport
You can apply a bucket policy to your s3 statement, denying get operations if the connection is aws:securetransport: false. 
This blocks http, allowing https

## 4. Important exam traps
What happens if you don't enable encryption manually?

Since January 2023, Amazon S3 automatically applies SSE-S3 (server-side encryption with S3-managed keys) as the default for new object uploads.

- S3 encrypts new objects by default. The default for general-purpose buckets is SSE-S3; you don't need to enable encryption manually for basic at-rest encryption.&#x20;

- Encryption does not equal access control. An encrypted object can still be accessed by an authorised principal. Use IAM policies, bucket policies and appropriate access controls.

- Changing the bucket's default encryption doesn't re-encrypt existing objects. Existing objects retain their previous encryption until you explicitly rewrite or re-encrypt them.

- SSE-KMS needs key permissions as well as S3 permissions. A principal may have permission to read an S3 object but still be unable to decrypt it because it lacks the required KMS permissions.

- HTTPS protects data in transit; SSE protects data at rest. For stronger transport security, a *bucket policy* can deny requests that don't use HTTPS/TLS.
