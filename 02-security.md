# Domain 2: Security (26%)

> 🔰 New to a term? Concepts link to plain-English explanations in [the basics page](00-developer-basics.md) on first mention, and every abbreviation is spelled out on first use and listed in the [glossary](00-abbreviations-glossary.md).

> **About 13 of the 50 scored questions.**
>
> Tests whether you can authenticate users and services, authorize them correctly, call Amazon Web Services (AWS) securely from code, protect data with encryption, [certificates](00-developer-basics.md#certificates-and-ssh-keys) and secrets, handle sensitive data, and (new in C03) use Artificial Intelligence (AI) services safely.
>
> Tags like `[also: DOP]` mark material you met in another exam. It is still covered in full here.

## Table of contents

- [Exam tasks and skills](#exam-tasks-and-skills)
- [Task 2.1: Authentication and authorization](#task-21-authentication-and-authorization)
- [Task 2.2: Encryption and sensitive data](#task-22-encryption-and-sensitive-data)
- [Task 2.3: AI service security risks](#task-23-ai-service-security-risks)
- [Gotchas and exam traps](#gotchas-and-exam-traps)
- [Quick-recall summary](#quick-recall-summary)

## Exam tasks and skills

> Summarized from the official DVA-C03 exam guide. Exact wording: https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html

### Task 2.1: Implement authentication and authorization for applications and AWS services
- **2.1.1** Implement identity [federation](00-developer-basics.md#federation-oauth-20-oidc-saml-and-jwt).
- **2.1.2** Implement access control (fine-grained permissions, security tokens, [Multi-Factor Authentication [MFA]](00-developer-basics.md#mfa)).
- **2.1.3** Configure programmatic access to AWS.
- **2.1.4** Use presigned URLs for time-limited access (Simple Storage Service [S3], CloudFront).
- **2.1.5** Make authenticated calls to AWS services.
- **2.1.6** Define permissions for Identity and Access Management (IAM) principals (users, roles, services) and resources.
- **2.1.7** Assume an IAM role.

### Task 2.2: Implement encryption and manage sensitive data
- **2.2.1** Apply [encryption at rest](00-developer-basics.md#encryption-at-rest-and-in-transit-symmetric-and-asymmetric) and in transit.
- **2.2.2** Manage certificates (AWS Certificate Manager [ACM], AWS Private Certificate Authority [CA]).
- **2.2.3** Select and apply [client-side](00-developer-basics.md#client-side-vs-server-side-encryption) or server-side encryption.
- **2.2.4** Use encryption keys to encrypt and decrypt data.
- **2.2.5** Generate certificates and Secure Shell (SSH) keys for development.
- **2.2.6** Identify, mask and limit access to sensitive data (Personally Identifiable Information [PII], [Protected Health Information [PHI]](00-developer-basics.md#pii-and-phi-masking-and-redaction)).
- **2.2.7** Use secret management services.

### Task 2.3: Identify and mitigate security risks of using AI services (new in C03)
- **2.3.1** Request and manage access to AI services.
- **2.3.2** Use data privacy controls when sending data to AI services (Virtual Private Cloud [VPC] endpoints for private connectivity; inputs/outputs not used to train models).
- **2.3.3** Filter and control model inputs and outputs (content filtering, sensitive information detection, PII/PHI redaction, denied topics, [prompt injection](00-developer-basics.md#prompt-injection) protection).
- **2.3.4** Secure [AI agent](00-developer-basics.md#ai-agents-tools-and-human-in-the-loop) interactions (tool-use [authorization](00-developer-basics.md#authentication-vs-authorization), session isolation, human-in-the-loop approvals).
- **2.3.5** Protect sensitive content in monitoring logs from AI interactions.

---

## Task 2.1: Authentication and authorization

### Amazon Cognito and identity federation (skill 2.1.1) `[also: SAA]`

- **User pools** (authentication for your app's users):
  - Sign-up/sign-in, password policies, **MFA** (Time-based One-Time Password [TOTP], SMS, email), account recovery, email/phone verification, **advanced security / threat protection** (compromised credential detection, adaptive authentication).
  - **Federation**: social IdPs (Google, Facebook, Apple, Amazon), **Security Assertion Markup Language (SAML) 2.0** and **OpenID Connect (OIDC)** enterprise IdPs → users get user pool tokens.
  - **Managed login / hosted User Interface (UI)** with Open Authorization (OAuth) 2.0 endpoints (`/oauth2/authorize`, `/oauth2/token`, `/oauth2/userInfo`); **app clients** (client ID, optional secret for confidential clients; allowed OAuth flows: **authorization code with Proof Key for Code Exchange [PKCE]** (recommended for web/mobile), client credentials (machine-to-machine with **resource servers and custom scopes**)); callback URLs.
  - **Tokens (JWTs)**: **ID token** (identity claims: `sub`, `email`, `cognito:groups`), **access token** (scopes, for calling Application Programming Interfaces [APIs]), **refresh token** (get new tokens; default 30 days, configurable). ID/access tokens default **1 hour** (5 min–1 day). Validate tokens by signature using the pool's **JSON Web Key Set [JWKS]** (`/.well-known/jwks.json`), and check `iss`, `aud`/`client_id`, `token_use`, `exp`.
  - **Groups** (with optional IAM role and precedence), custom attributes, **Lambda triggers**: pre sign-up, post confirmation, pre authentication, post authentication, **pre token generation** (add/override claims and scopes), custom message, custom auth challenge (define/create/verify), user migration.
  - Integrations: **[Application Programming Interface [API]](00-developer-basics.md#api) Gateway Cognito authorizer** ([REST](00-developer-basics.md#rest-and-http-methods)) / **JSON Web Token (JWT) authorizer** (Hypertext Transfer Protocol [HTTP] API), **Application Load Balancer (ALB) authentication**, AppSync, Amplify Auth.
- **Identity pools** (federated identities → AWS credentials):
  - Exchange tokens from a user pool, social Identity Provider (IdP), SAML, OIDC or **developer-authenticated identities** (your own backend) for **temporary AWS credentials** via Security Token Service (STS) `AssumeRoleWithWebIdentity`.
  - **Authenticated** and **unauthenticated (guest)** roles; **role-based access control** (choose role from token claims/groups, rules-based mapping); **attributes for access control** (map token claims to principal tags for Attribute-Based Access Control [ABAC]).
  - Use policy variables like `${cognito-identity.amazonaws.com:sub}` to restrict each user to their own S3 prefix/DynamoDB items.
- **Other federation**: **IAM Identity Center** (workforce access to AWS accounts/CLI via Single Sign-On [SSO]), direct IAM **SAML/OIDC identity providers** (e.g., GitHub Actions OIDC → role for [Continuous Integration / Continuous Delivery [CI/CD]](00-developer-basics.md#cicd) with no long-term keys), **IAM Roles Anywhere** (on-prem workloads with X.509 certificates get [temporary credentials](00-developer-basics.md#temporary-credentials)).
- **Exam triggers:** "users sign in with Google and call our API" → **user pool (federated) + API Gateway Cognito/JWT authorizer**. "Mobile app uploads directly to S3 per user" → **identity pool** + per-user policy variable. "Guest users get limited access" → **unauthenticated role**. "Add custom claims to tokens" → **pre token generation trigger**. "CI pipeline in GitHub needs AWS access without keys" → **OIDC provider + role**.

### Access control: fine-grained permissions, tokens, MFA (skill 2.1.2) `[also: DOP]`

- **Fine-grained IAM**: conditions on keys such as `dynamodb:LeadingKeys` (only items whose [partition key](00-developer-basics.md#partition-key-and-cardinality) equals the user's ID), `dynamodb:Attributes` (only certain attributes), `s3:prefix`, `aws:ResourceTag`/`aws:PrincipalTag` (**ABAC**), `aws:SourceIp`, `aws:SourceVpce`, `aws:SecureTransport`, `aws:MultiFactorAuthPresent`.
- **Security tokens**: STS session tokens (temporary credentials), Cognito JWTs, API Gateway **API keys** (identify clients for usage plans, **not** an authentication mechanism), Lambda authorizers validating custom/OAuth tokens (return IAM policy + context, cache by [Time To Live [TTL]](00-developer-basics.md#ttl-and-data-lifecycle)).
- **API Gateway authorization options**: **IAM ([SigV4](00-developer-basics.md#request-signing-sigv4))** for AWS principals/service-to-service; **Cognito user pool authorizer** (REST) or **JWT authorizer** (HTTP API) for app users; **Lambda authorizer** (token or request based) for custom schemes (third-party OAuth, API tokens, IP/headers); **resource policies** (restrict by account, Internet Protocol [IP], VPC endpoint; required for private APIs and cross-account); **mutual Transport Layer Security [TLS]** (client certificates).
- **MFA**: Cognito user pool MFA for app users; IAM MFA for AWS users; enforce via `aws:MultiFactorAuthPresent`; get MFA-backed temporary credentials with `sts:GetSessionToken --serial-number --token-code`.
- **Amazon Verified Permissions** (fine-grained app authorization with Cedar policies) is a related service if you see it.
- **Exam triggers:** "each user can only read their own DynamoDB items" → **`dynamodb:LeadingKeys` with Cognito identity ID**. "Third-party OAuth tokens must authorize API calls" → **Lambda authorizer**. "Service-to-service calls with AWS credentials" → **IAM auth (SigV4)**.

### Programmatic access and authenticated calls (skills 2.1.3, 2.1.5) `[also: DOP]`

- **Programmatic access options (best to worst)**: IAM roles for compute (Elastic Compute Cloud [EC2] instance profile, Lambda execution role, Elastic Container Service [ECS] task role, Elastic Kubernetes Service [EKS] Pod Identity/IRSA) → **IAM Identity Center** short-term Command Line Interface (CLI) credentials (`aws configure sso`, `aws sso login`) → federation with OIDC/SAML → IAM Roles Anywhere → **IAM user access keys** (last resort; rotate, [least privilege](00-developer-basics.md#principal-policy-and-least-privilege), never commit).
- **Credential files**: `~/.aws/credentials` (keys per profile), `~/.aws/config` (Region, output, `role_arn` + `source_profile` for role assumption, `mfa_serial`, `sso_session`, `credential_process`).
- **Authenticated calls**: Software Development Kits (SDKs)/CLI sign requests with **SigV4** (signature over method, path, headers, payload hash, timestamp; includes the **session token** for temporary credentials; requests expire after ~5–15 minutes, so clock skew causes `SignatureDoesNotMatch`/`RequestExpired`). Calling **IAM-protected API Gateway** or **Lambda function URLs** with `AWS_IAM` auth from code requires SigV4 signing ([Software Development Kit [SDK]](00-developer-basics.md#sdk-and-cli) signer, `aws-sigv4` libraries, Amplify). **SigV4a** for multi-Region endpoints (S3 Multi-Region Access Points).
- **Errors**: `UnrecognizedClientException`/`InvalidClientTokenId` (bad key), `ExpiredToken` (refresh temporary credentials), `AccessDenied`/`UnauthorizedOperation` (permissions), `SignatureDoesNotMatch` (wrong secret, clock skew, modified request).
- **Exam triggers:** "developer needs CLI access without long-term keys" → **Identity Center SSO profile**. "Call IAM-authorized API Gateway from a Lambda" → **SigV4-sign the request** with the function's role credentials. "`ExpiredToken` after an hour" → refresh/assume role again.

### Presigned URLs and signed CloudFront access (skill 2.1.4) `[also: SAA]`

- **S3 presigned URLs**: generated with the SDK (`generate_presigned_url`, `getSignedUrl`, `PresignedPost` for browser form uploads) using the **signer's credentials**; grant time-limited `GET` (download) or `PUT`/`POST` (upload) to anyone holding the Uniform Resource Locator (URL).
  - Expiry: up to **7 days** with IAM user long-term credentials (SigV4); limited by the **session's expiry** when signed with temporary credentials (e.g., Lambda role → URL dies when the session token expires, ~up to 12 h; often less); console-generated up to 12 h.
  - The request succeeds only if the **signer** has permission at the time of use (revoke by removing the signer's permissions).
  - Upload use case: client gets a presigned PUT URL from your API, uploads directly to S3 (no data through your servers); restrict content type/size with presigned POST conditions.
- **CloudFront signed URLs and signed cookies**: for private content served via CloudFront (origin locked with **Origin Access Control [OAC]**). Create a **key group** with your public key; sign with the private key. **Canned policy** (one URL, expiry only) or **custom policy** (wildcards, start time, IP range). **Signed URL** = one file or clients that don't support cookies; **signed cookies** = many files (HLS video segments, whole site section) without changing URLs.
- **Exam triggers:** "let users upload directly to S3 from the browser securely" → **presigned PUT/POST URL**. "Premium users access all videos in a path via CloudFront" → **signed cookies**. "Presigned URL stops working after an hour although expiry is 7 days" → signed with **temporary credentials** that expired.

### Permissions for IAM principals and resources (skill 2.1.6) `[also: DOP, SAA]`

- **Identity-based policies** (attached to users, groups, roles): AWS managed, customer managed, inline.
- **Resource-based policies** (attached to resources, with `Principal`): S3 bucket policy, Simple Queue Service (SQS) queue policy, Simple Notification Service (SNS) topic policy, **Lambda function policy** (who can invoke), Key Management Service (KMS) key policy, Secrets Manager secret policy, Elastic Container Registry (ECR) repository policy, API Gateway resource policy, EventBridge bus policy.
- **Service principals**: allow services to act (`"Principal": {"Service": "lambda.amazonaws.com"}` in a role **trust policy**; `s3.amazonaws.com` in a Lambda/SNS/SQS resource policy with `aws:SourceArn`/`aws:SourceAccount` to prevent **confused deputy**).
- **Lambda**: **execution role** (what the function can do) vs **resource policy** (who can invoke). **ECS**: **task role** (app permissions) vs **task execution role** (pull images from ECR, write logs, fetch secrets for injection).
- **Permissions boundaries** (cap what a role can get; let developers create roles safely), **Service Control Policies [SCPs]** (org [guardrails](00-developer-basics.md#guardrails)), **session policies**.
- **Policy evaluation**: explicit deny > allow; same-account access via identity **or** resource policy; cross-account needs **both**; all ceilings (Service Control Policy [SCP], boundary, session) must allow.
- **Policy elements**: `Effect`, `Action` (wildcards like `s3:Get*`), `NotAction`, `Resource` (Amazon Resource Names [ARNs], e.g., `arn:aws:s3:::bucket/*` for objects vs `arn:aws:s3:::bucket` for `ListBucket`), `Condition`, **policy variables** (`${aws:username}`, `${aws:PrincipalTag/team}`, `${cognito-identity.amazonaws.com:sub}`).
- **Tools**: **IAM Access Analyzer** (policy validation, generation from CloudTrail, external access), **IAM policy simulator**.
- **Exam triggers:** "Lambda gets AccessDenied writing to DynamoDB" → add permission to the **execution role**. "S3 can't invoke Lambda" → **Lambda resource policy**. "ECS can't pull image" → **task execution role**. "Allow only `ListBucket` on bucket and `GetObject` on objects" → two resource ARNs.

### Assuming IAM roles (skill 2.1.7) `[also: DOP, SAA]`

- **STS `AssumeRole`**: returns temporary credentials (access key, secret, session token, expiration). Parameters: `RoleArn`, `RoleSessionName` (shows in CloudTrail), `DurationSeconds` (900 s up to the role's max session duration, 1–12 h; **role chaining limited to 1 h**), `ExternalId` (third-party access), `SerialNumber`/`TokenCode` (MFA), `Policy`/`PolicyArns` (session policies to further restrict), `Tags` (session tags for ABAC), `SourceIdentity`.
- **Requirements**: the role's **trust policy** allows the caller principal (`sts:AssumeRole`), and the caller's identity policy allows `sts:AssumeRole` on that role (for cross-account, both sides).
- **Other STS APIs**: `AssumeRoleWithWebIdentity` (OIDC tokens: Cognito identity pools, EKS IAM Roles for Service Accounts [IRSA], GitHub Actions), `AssumeRoleWithSAML`, `GetSessionToken` (MFA for IAM users), `GetCallerIdentity` (who am I; no permissions needed), `DecodeAuthorizationMessage`.
- **CLI/SDK**: profile with `role_arn` + `source_profile` (+ `mfa_serial`) assumes automatically and caches credentials; SDK `STS client.assume_role(...)` then create service clients with the returned credentials (refresh before expiry).
- **Exam triggers:** "app in account A must write to a bucket in account B" → role in B trusted by A (+ A's policy allows AssumeRole), or bucket policy in B. "Third-party Software as a Service (SaaS) needs access" → role with **external ID**. "Check which identity the code runs as" → **`GetCallerIdentity`**.

---

## Task 2.2: Encryption and sensitive data

### Encryption at rest and in transit (skill 2.2.1) `[also: SAA]`

- **At rest (server-side, KMS-integrated)**: **S3** (Server-Side Encryption with S3-managed keys [SSE-S3] default; Server-Side Encryption with KMS keys [SSE-KMS]; Dual-layer Server-Side Encryption with KMS keys [DSSE-KMS]; Server-Side Encryption with Customer-provided keys [SSE-C]; default bucket encryption; **Bucket Keys** reduce KMS calls), **DynamoDB** (always encrypted: AWS owned, AWS managed or customer managed key), **RDS/Aurora** (enable at creation; snapshot-copy to encrypt existing), **Elastic Block Store [EBS]** (encryption by default), **Elastic File System [EFS]** (at creation), **SQS** (Server-Side Encryption with SQS-owned keys [SSE-SQS] or SSE-KMS), **SNS** (SSE-KMS), **Kinesis** (SSE-KMS), **CloudWatch Logs** (KMS key on log group), **Lambda [environment variables](00-developer-basics.md#environment-variables-and-configuration)** (KMS), **Secrets Manager**/**Parameter Store SecureString** (KMS), **ECR**, **Step Functions**, **ElastiCache** (at-rest option).
- **In transit**: **TLS/HTTPS** for all AWS API endpoints (SDK default); enforce HTTPS-only to S3 with bucket policy `aws:SecureTransport=false → Deny`; API Gateway endpoints are HTTPS-only (minimum TLS 1.2 security policy options); **ALB/Network Load Balancer (NLB)** TLS listeners with ACM certs and TLS security policies; **CloudFront** viewer protocol policy redirect/HTTPS only + origin Hypertext Transfer Protocol Secure (HTTPS); **Relational Database Service (RDS)** Secure Sockets Layer (SSL)/TLS (download the RDS CA bundle, `sslmode=verify-full`; force with `rds.force_ssl`/`require_secure_transport`); **ElastiCache** in-transit encryption (TLS); **mutual TLS (mTLS)** with ALB or API Gateway trust stores; Virtual Private Network (VPN)/Internet Protocol Security (IPsec) for networks.
- **Exam triggers:** "ensure all S3 access uses HTTPS" → **`aws:SecureTransport` deny**. "Encrypt existing unencrypted RDS" → **snapshot → encrypted copy → restore**. "Every encrypt/decrypt must be auditable with our own key" → **SSE-KMS with customer managed key**.

### Managing certificates: ACM and AWS Private CA (skill 2.2.2) `[also: SAA]`

- **AWS Certificate Manager (ACM)**: free public TLS certificates (Domain Name System [DNS] or email validation; **DNS validation enables automatic renewal**), import third-party certificates (no auto-renew), deploy to **ALB/NLB, CloudFront (cert in us-east-1), API Gateway custom domains**, App Runner, Amplify, Cognito custom domains. Public ACM certificates generally can't be exported (except the paid **exportable public certificates** option). Expiry events via EventBridge/Health; `DaysToExpiry` metric.
- **AWS Private CA**: managed **private certificate authority** hierarchy (root/subordinate) for **internal** TLS (microservices, service mesh), **mutual TLS client certificates**, IoT devices, code signing; issue private certs through ACM (managed renewal) or directly via the Private CA API (exportable, you manage renewal); **short-lived certificate mode** for cheaper short validity certs; Certificate Revocation List (CRL)/Online Certificate Status Protocol (OCSP) revocation; share CAs with RAM.
- **Exam triggers:** "HTTPS for a custom API Gateway domain" → **ACM cert** (Regional in API's Region; edge-optimized in us-east-1). "Internal services need certificates from our own CA, with mTLS" → **AWS Private CA**.

### Client-side vs server-side encryption (skill 2.2.3) `[also: SAA]`

| | Server-side encryption | Client-side encryption |
|---|---|---|
| Who encrypts | The AWS service after receiving data | Your application before sending |
| Service sees plaintext | Yes (briefly, in memory) | **No** |
| Effort | Configuration only | Code + key handling |
| Tools | SSE-S3, SSE-KMS, DSSE-KMS, SSE-C, DynamoDB/RDS encryption | **AWS Encryption SDK**, **Amazon S3 Encryption Client**, **AWS Database Encryption SDK** (DynamoDB item attribute-level encryption and signing) |
| When | Default; compliance satisfied by encryption at rest | Data must be protected from the storage service/admins, end-to-end, field-level protection |

- **SSE-C**: you supply the key on every request over HTTPS; S3 doesn't store it; you manage keys (lose it = lose data).
- **CloudFront field-level encryption**: encrypt specific form fields at the edge with a public key so only a backend with the private key can read them.
- **Exam triggers:** "data must be encrypted before leaving the app; AWS must never see plaintext" → **client-side (Encryption SDK with KMS)**. "Encrypt specific DynamoDB attributes" → **Database Encryption SDK**.

### Using encryption keys: AWS KMS APIs (skill 2.2.4) `[also: DOP, SAA]`

- **Key concepts**: customer managed vs AWS managed vs AWS owned keys; symmetric (AES-256-GCM) default; asymmetric (RSA/Elliptic Curve Cryptography [ECC]: encrypt/decrypt or **sign/verify**), [Hash-based Message Authentication Code [HMAC]](00-developer-basics.md#hashing-and-hmac) (generate/verify MAC); **key policy** (mandatory) + IAM + **grants**; **aliases** (`alias/app-key`); **automatic rotation** (customer managed symmetric; key ID stays, old material kept); multi-Region keys.
- **APIs**:
  - **`Encrypt`**: encrypt up to **4 KB** of data directly (e.g., a password, a data key).
  - **`Decrypt`**: decrypt ciphertext (symmetric ciphertext includes the key ID, so `KeyId` is optional).
  - **`GenerateDataKey`**: returns a **plaintext data key + encrypted copy** → [envelope encryption](00-developer-basics.md#envelope-encryption): encrypt data locally with plaintext key (AES-GCM), **discard plaintext key** from memory, store encrypted key with the data; to decrypt, call `Decrypt` on the encrypted data key.
  - **`GenerateDataKeyWithoutPlaintext`**: only the encrypted data key (for a component that will encrypt later).
  - **`ReEncrypt`**: change the KMS key protecting ciphertext without exposing plaintext to you.
  - `Sign`/`Verify`, `GenerateMac`/`VerifyMac`, `GenerateRandom`, `CreateGrant`, `DescribeKey`.
  - **Encryption context**: non-secret key-value pairs bound to the ciphertext (**additional authenticated data**); must match on decrypt; logged in CloudTrail; usable in policy conditions (`kms:EncryptionContext:tenant`).
- **AWS Encryption SDK**: client library implementing envelope encryption with best practices (data key caching to reduce KMS calls, multi-key keyrings, encryption context, message format).
- **Quotas**: KMS cryptographic request quotas per account per Region (shared across callers, e.g., 5,500–100,000/s depending on Region and key type) → `ThrottlingException` → **data key caching**, S3 Bucket Keys, request quota increase, [backoff](00-developer-basics.md#retries-exponential-backoff-and-jitter).
- **Exam triggers:** "encrypt a 50 MB file with KMS" → **`GenerateDataKey` envelope encryption** (Encrypt is limited to 4 KB). "Too many KMS calls / throttling" → **data key caching**. "Change the key protecting stored ciphertext" → **`ReEncrypt`**. "Bind ciphertext to a tenant" → **encryption context**.

### Generating certificates and SSH keys for development (skill 2.2.5)

- **SSH keys**: `ssh-keygen -t ed25519` (or Rivest–Shamir–Adleman [RSA] 4096) creates private/public keys; or **EC2 key pairs** (`aws ec2 create-key-pair` → private key returned once; import your own public key with `import-key-pair`; RSA or ED25519). Keep private keys out of repos; `chmod 400`. Prefer **Session Manager** or **EC2 Instance Connect** (pushes a temporary public key for 60 s) instead of long-lived SSH keys.
- **Development certificates**: **self-signed** certificates with OpenSSL (`openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365 -subj "/CN=localhost"`) for local HTTPS testing (browsers warn); create a **Certificate Signing Request [CSR]** (`openssl req -new`) to request a CA-signed certificate; private dev CA via **AWS Private CA** for realistic internal TLS/mTLS; import self-signed certs into ACM only for testing (ALB → target re-encryption accepts self-signed target certs).
- **[Git](00-developer-basics.md#source-control)/CodeCommit-style credentials**: HTTPS Git credentials or SSH public keys uploaded to IAM users (legacy CodeCommit).
- **Exam triggers:** "local HTTPS for testing" → **self-signed cert with OpenSSL**. "Avoid distributing SSH keys" → **EC2 Instance Connect / Session Manager**.

### Sensitive data: identify, mask, limit access (skill 2.2.6)

- **Identify**: **Amazon Macie** (S3 PII discovery), **Amazon Comprehend** PII detection API (`DetectPiiEntities`, redaction), **Bedrock Guardrails** sensitive information filters, **CloudWatch Logs data protection** (managed data identifiers for PII/credentials/financial; custom identifiers).
- **Mask/redact**:
  - **CloudWatch Logs data protection policies**: automatically **mask** sensitive data in log events; only principals with `logs:Unmask` see unmasked data; audit findings.
  - **SNS message data protection**: detect, mask, deny or audit PII in messages.
  - **S3 Object Lambda**: redact PII on the fly when objects are read (via Lambda).
  - **Bedrock Guardrails**: block or **mask** PII in [prompts](00-developer-basics.md#prompt-system-prompt-and-inference-parameters)/responses (Task 2.3).
  - App-level: never log full payloads; mask fields before logging (Powertools data masking utility); tokenization.
- **Limit access**: least-privilege IAM, `dynamodb:Attributes` conditions, separate tables/buckets for sensitive data, KMS key policies restricting decrypt to specific roles (`kms:ViaService`), client-side/attribute-level encryption, **Lake Formation** column/row filters for analytics.
- **PHI/Health Insurance Portability and Accountability Act (HIPAA)**: use HIPAA-eligible services, encrypt, audit (CloudTrail), Business Associate Addendum (BAA) via AWS Artifact.
- **Exam triggers:** "credit card numbers appear in application logs" → **CloudWatch Logs data protection policy** (and fix logging). "Return documents with PII removed to some users" → **S3 Object Lambda**. "Detect PII in free text" → **Comprehend** / Guardrails.

### Secret management (skill 2.2.7) `[also: DOP, SAA]`

- **AWS Secrets Manager**: store DB credentials, API keys, OAuth tokens; **automatic rotation** (managed or Lambda-based; RDS/Aurora/Redshift/DocumentDB templates); versions with staging labels `AWSCURRENT`/`AWSPENDING`/`AWSPREVIOUS`; KMS encryption; resource policies; cross-Region replication; `GetSecretValue` (cache it: **client-side caching libraries**, **Parameters and Secrets Lambda Extension**); **RDS-managed master credentials**; ECS/EKS injection (task definition `secrets` → env vars at start; execution role needs `secretsmanager:GetSecretValue`); CloudFormation dynamic references (`{{resolve:secretsmanager:...}}`); `GenerateRandomPassword`.
- **Systems Manager Parameter Store**: `String`/`StringList`/**`SecureString`** (KMS); standard (free, 4 KB) / advanced (8 KB, parameter policies); hierarchies `/app/prod/db-host` with `GetParametersByPath`; versions/labels; no automatic rotation; also references Secrets Manager secrets.
- **Choosing**: rotation needed / cross-account / replication → **Secrets Manager**; config values and simple secrets, cost-sensitive → **Parameter Store**.
- **Never**: hard-code secrets, put them in plain env vars, Git, Amazon Machine Images (AMIs), logs, or Docker images.
- **Exam triggers:** "database password must rotate automatically without app changes" → **Secrets Manager rotation** + app fetches `AWSCURRENT` (cached). "Reduce latency/cost of fetching secrets in Lambda" → **extension cache**. "ECS task needs DB password as env var" → task definition **`secrets`** from Secrets Manager/Parameter Store.

---

## Task 2.3: AI service security risks

### Requesting and managing access to AI services (skill 2.3.1) `[also: AIP]`

- **Model access**: Bedrock serverless models are invoked under IAM control; some third-party models require accepting the provider's **End User License Agreement (EULA)/terms** (via AWS Marketplace subscription) the first time, and access can be governed centrally (allow-list models with **SCPs/IAM conditions** on model ARNs, e.g., deny `bedrock:InvokeModel` except approved `foundation-model/...` ARNs).
- **IAM**: least-privilege actions (`bedrock:InvokeModel`, `bedrock:InvokeModelWithResponseStream`, `bedrock:Converse`, `bedrock:ApplyGuardrail`, `bedrock:Retrieve`, `bedrock:InvokeAgent`), resource ARNs for **[foundation models](00-developer-basics.md#foundation-model-llm-and-tokens), inference profiles, guardrails, knowledge bases, agents**; condition keys like `bedrock:GuardrailIdentifier` to **require a specific guardrail** on every invocation.
- **Quotas and cost control**: service quotas for tokens/requests per minute per model; **application inference profiles** with tags for cost allocation; Budgets alerts; Provisioned Throughput for guaranteed capacity.
- **AgentCore/agents access**: execution roles for agents/runtimes scoped to the tools they need; **AgentCore Identity** for inbound auth (who can invoke the agent: IAM SigV4 or OAuth/JWT from Cognito/other IdPs) and outbound credentials (OAuth tokens/API keys stored securely for tools, user-delegated access).
- **Kiro**: authenticate via AWS IAM Identity Center or supported identity providers; enterprise governance controls which models/MCP servers are allowed.
- **Exam triggers:** "only approved models may be used across the organization" → **SCP/IAM allow-list on model ARNs**. "Every invocation must use the company guardrail" → **`bedrock:GuardrailIdentifier` condition**.

### Data privacy controls for AI services (skill 2.3.2) `[also: AIP, AIF]`

- **No training on your data**: Amazon Bedrock **doesn't use prompts or completions to train AWS or third-party models** and doesn't share them with model providers; model providers can't access the deployment accounts; data stays in the Region you use (unless you choose cross-Region inference profiles, which route within a geography).
- **Private connectivity**: **interface VPC endpoints (AWS PrivateLink)** for `bedrock-runtime`, `bedrock`, `bedrock-agent-runtime`, AgentCore, so traffic doesn't traverse the internet; **endpoint policies** restrict which models/actions are callable through the endpoint; combine with IAM condition `aws:SourceVpce`.
- **Encryption**: TLS in transit; **customer managed KMS keys** for knowledge bases, agents, custom models, model invocation logs, guardrails.
- **Data minimization**: send only needed fields; strip/redact PII before sending (Comprehend, Guardrails sensitive info filters, app logic); keep sensitive retrieval data access-controlled (knowledge base metadata filtering per user/tenant).
- **Other AI services**: check each service's **opt-out**/data use policy (some AWS AI services may use content for service improvement unless you opt out via **AI services opt-out policies** in AWS Organizations).
- **Exam triggers:** "calls to Bedrock must not leave the AWS network" → **Bedrock interface VPC endpoint**. "Ensure our prompts aren't used to train models" → **Bedrock doesn't train on customer data** (and org AI opt-out policy for other AI services).

### Filtering model inputs and outputs: Amazon Bedrock Guardrails (skill 2.3.3) `[also: AIP]`

- **What it is:** configurable safeguards applied to prompts and/or responses, for Bedrock models (via `guardrailConfig` in `Converse`/`InvokeModel`, Agents, Knowledge Bases) and for **any** text (including other models/self-hosted) via the **`ApplyGuardrail` API**. Versioned; returns `guardrail_intervened` with a configurable blocked message; trace shows which policy triggered.
- **Policies:**
  - **Content filters**: hate, insults, sexual, violence, misconduct with strength levels (none/low/medium/high) for input and output; image content filtering for multimodal.
  - **Prompt attack filter**: detects **prompt injection** and **jailbreak** attempts in user input (mark untrusted input with input tags so only user content is evaluated).
  - **Denied topics**: natural-language topic definitions to refuse (e.g., "investment advice").
  - **Word filters**: custom words/phrases and a managed **profanity** list.
  - **Sensitive information filters**: predefined **PII** types (names, emails, phone, Social Security Number [SSN], credit cards, addresses…) and **custom regex**; action **BLOCK** or **MASK/ANONYMIZE** (replace with `{EMAIL}` etc.) on input and output.
  - **Contextual grounding check**: block responses not grounded in the provided source or not relevant to the query (reduce hallucinations in [Retrieval-Augmented Generation [RAG]](00-developer-basics.md#rag)).
  - **Automated Reasoning checks**: validate responses against formal policy rules.
- **Additional app-level controls**: separate system instructions from user data, escape/tag retrieved content, limit output length, validate structured outputs (JavaScript Object Notation [JSON] schema), allow-list tool calls, rate-limit users (API Gateway/Web Application Firewall [WAF]), monitor guardrail intervention metrics.
- **Exam triggers:** "block questions about competitors' products" → **denied topic**. "Remove customer emails/phone numbers from model responses" → **sensitive information filter (mask)**. "Users try 'ignore previous instructions'" → **prompt attack filter**. "Apply the same safety policies to a model hosted outside Bedrock" → **`ApplyGuardrail` API**.

### Securing AI agent interactions (skill 2.3.4) `[also: AIP]`

- **Tool-use authorization**: give agents only the tools they need; each tool runs with **least-privilege IAM** (Lambda action group execution roles, AgentCore Gateway targets); authorize tool calls **on behalf of the end user** (propagate user identity/OAuth tokens via AgentCore Identity; check permissions in the tool, not just in the prompt); **policy controls** that deterministically allow/deny tool actions; validate tool inputs (don't trust model-generated parameters).
- **Session isolation**: per-user/per-session context so one user's data or memory can't leak to another: **AgentCore Runtime** runs each session in an isolated microVM; use `sessionId` per user conversation in Bedrock Agents; scope **memory** and knowledge-base retrieval by user/tenant (metadata filters); don't share conversation state across tenants.
- **Human-in-the-loop**: require approval before sensitive actions: Bedrock Agents **user confirmation** on action groups and **return of control** (agent returns the planned action to your app, which asks the user before executing); Step Functions **`.waitForTaskToken`** approval steps; [Model Context Protocol [MCP]](00-developer-basics.md#mcp) tool approval prompts in clients (e.g., Kiro asks before running commands).
- **Other**: guardrails on agent inputs/outputs, audit every tool call (CloudTrail, AgentCore Observability [traces](00-developer-basics.md#logs-metrics-and-traces)), limit iterations/timeouts, network isolation (agent tools in VPC), protect against **indirect prompt injection** from tool outputs/web pages (treat as untrusted data).
- **Exam triggers:** "agent must ask the user before placing an order" → **user confirmation / return of control**. "Agent tools must act only with the signed-in user's permissions" → **identity propagation (AgentCore Identity / OAuth)**. "Prevent one customer's conversation context leaking to another" → **session isolation** (AgentCore Runtime sessions, per-user session IDs and memory namespaces).

### Protecting sensitive content in AI monitoring logs (skill 2.3.5) `[also: AIP]`

- **Bedrock model invocation logging** (off by default) captures full **prompts and responses** (text, images, [embeddings](00-developer-basics.md#embeddings)) to **CloudWatch Logs and/or S3** → treat as sensitive: encrypt with **customer managed KMS keys**, restrict log access with IAM/bucket policies, set **retention**, and apply **CloudWatch Logs data protection policies** to **mask PII** (unmask only with `logs:Unmask`).
- **Guardrails** mask PII **before** it reaches the model and logs (input masking) and in outputs.
- **Application logs**: don't log raw prompts/outputs by default; log metadata (model ID, token counts, latency, request ID, guardrail action); use structured logging with explicit field allow-lists; **Powertools data masking**.
- **Traces/observability** (AgentCore Observability, X-Ray/OpenTelemetry): avoid putting PII in span attributes; sample appropriately.
- **CloudTrail**: records Bedrock API calls (not prompt contents for runtime calls by default; data events optional); protect trail buckets.
- **Exam triggers:** "invocation logs contain customer PII; analysts need logs without PII" → **CloudWatch Logs data protection masking** (+ KMS, restricted `logs:Unmask`). "Keep a full audit of prompts for compliance, securely" → **invocation logging to encrypted S3 with retention**.

---

## Gotchas and exam traps

- **User pool** = authentication + JWTs; **identity pool** = AWS credentials.
- API Gateway **API keys aren't authentication**; use IAM, Cognito/JWT or Lambda authorizers.
- **Lambda execution role** = what it can do; **resource policy** = who can invoke it.
- **ECS task role** (app) vs **task execution role** (pull image, logs, secrets).
- Cross-account = **both** sides allow; role chaining max **1 h**.
- Presigned URL lifetime is capped by the **signer's credential expiry**.
- CloudFront: **signed cookies** for many files; signed URL for one.
- KMS **`Encrypt` ≤ 4 KB** → envelope encryption with `GenerateDataKey`.
- **Encryption context** must match on decrypt.
- ACM imported certs don't auto-renew; CloudFront certs in **us-east-1**.
- Secrets Manager rotates; Parameter Store doesn't; cache secrets in Lambda.
- Bedrock **doesn't train on your prompts**; use **PrivateLink** for private access.
- Guardrails: content filters, **prompt attack**, denied topics, word filters, **PII mask/block**, grounding; `ApplyGuardrail` for any model.
- Agents: least-privilege tools, **user confirmation/return of control**, session isolation.
- Invocation logs hold prompts → **encrypt + mask + restrict + retention**.

## Quick-recall summary

- **AuthN/AuthZ**: Cognito user pools (federation, tokens, triggers) and identity pools (temporary credentials, per-user policies), Identity Center, OIDC for CI, API Gateway authorizers, fine-grained IAM conditions, MFA.
- **Programmatic access**: roles first, SSO profiles, SigV4 signing, STS (AssumeRole, web identity, GetSessionToken, GetCallerIdentity), credential errors.
- **Presigned access**: S3 presigned GET/PUT/POST, CloudFront signed URLs/cookies with key groups.
- **Encryption**: at rest per service, in transit (TLS, policies), ACM vs Private CA, client vs server side, KMS APIs (Encrypt/Decrypt/GenerateDataKey/ReEncrypt, encryption context, quotas), dev certs and SSH keys.
- **Sensitive data and secrets**: Macie/Comprehend/Guardrails detection, CloudWatch Logs and SNS data protection, S3 Object Lambda redaction, Secrets Manager vs Parameter Store.
- **AI security**: model access governance, PrivateLink and no-training guarantees, Guardrails policies, agent tool authorization/session isolation/human approval, protecting invocation logs.
