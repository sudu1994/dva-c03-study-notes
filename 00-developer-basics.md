# Developer Basics (the 101 layer)

> Plain-English explanations of the software, Application Programming Interface (API), data, security and Artificial Intelligence (AI) ideas the DVA-C03 notes rely on. Cloud Practitioner basics are left out. The first mention of each concept in the domain files links back here. Abbreviations are in [00-abbreviations-glossary.md](00-abbreviations-glossary.md).

## How DVA questions think

DVA asks *what code, configuration or service feature a developer uses* to make an app work, secure, deployable and fast: which Software Development Kit (SDK) call, which Lambda setting, which DynamoDB key design, which header, which deployment configuration, which log tells you why it broke.

| Constraint word | Usually means |
|---|---|
| "least operational overhead" / "minimal code changes" | Managed feature/configuration (Lambda destinations, Simple Queue Service [SQS] Dead-Letter Queue [DLQ], API Gateway validation) over custom code |
| "most secure" | Temporary credentials (roles), least privilege, no secrets in code, encryption with Key Management Service (KMS) |
| "decouple" / "asynchronous" | SQS, Simple Notification Service (SNS), EventBridge, Step Functions |
| "real time" | Kinesis/streams, WebSockets, DynamoDB Streams |
| "without impacting production" | Aliases, stages, canary, separate environments |

---

## Software and API concepts

### API
An **Application Programming Interface**: a defined way for programs to call each other. On Amazon Web Services (AWS), every action (create a bucket, put an item) is an API call; the console, Command Line Interface (CLI) and Software Development Kits (SDKs) all call the same Application Programming Interfaces (APIs).

### REST and HTTP methods
**Representational State Transfer (REST)** APIs use Hypertext Transfer Protocol (HTTP): `GET` (read), `POST` (create), `PUT` (replace), `PATCH` (partial update), `DELETE`. Resources are addressed by URLs (`/orders/123`). Responses carry **status codes** (200 OK, 201 Created, 400 bad request, 401 unauthenticated, 403 forbidden, 404 not found, 409 conflict, 429 throttled, 500 server error, 502 bad gateway, 503 unavailable, 504 timeout).

### GraphQL and WebSocket
**Graph Query Language (GraphQL)**: clients ask for exactly the fields they need in one query (AWS AppSync). **WebSocket**: a persistent two-way connection so the server can push messages (chat, live updates).

### SDK and CLI
An **SDK** (software development kit) is a library for a programming language (boto3 for Python, AWS SDK for JavaScript v3, Java v2) that signs requests, retries, paginates and handles errors. The **AWS CLI** is a command-line tool over the same APIs.

### Request signing (SigV4)
Every AWS API request is signed with **Signature Version 4** using the caller's secret key, proving identity and integrity. SDKs/CLI do it automatically; you only do it by hand for raw HTTP calls (e.g., to an IAM-authorized API Gateway).

### Pagination
APIs return large result sets in **pages** with a `NextToken`/`LastEvaluatedKey`; keep calling until no token is returned (SDK **paginators** do this).

### Synchronous vs asynchronous
**Synchronous**: the caller waits for the result (API Gateway → Lambda). **Asynchronous**: the caller hands work to a queue/event system and continues; results come later (Simple Storage Service [S3] event → Lambda, SQS).

### Tightly vs loosely coupled
**Tightly coupled** components call each other directly and fail together. **Loosely coupled** components talk through queues/topics/events, so one can be slow or down without breaking the other.

### Event-driven architecture
Components react to **events** ("order placed") published by others instead of being called directly.

### Idempotency
Doing the same request twice has the same effect as once. Needed because retries and at-least-once delivery cause duplicates. Implement with an **idempotency key** stored in DynamoDB (e.g., Powertools for AWS Lambda idempotency utility) or conditional writes.

### Retries, exponential backoff and jitter
On throttling (429/`ThrottlingException`) or transient 5xx errors, retry after waiting 1, 2, 4, 8 s…(**exponential backoff**) plus randomness (**jitter**). SDKs do this automatically (configurable max attempts, retry mode `standard`/`adaptive`). Don't retry 4xx validation errors.

### Timeouts and circuit breakers
Always set timeouts on outbound calls. A **circuit breaker** stops calling a failing dependency for a while to let it recover.

### Graceful degradation
When a dependency fails, keep serving a reduced experience (cached data, default response) instead of failing completely.

### Serialization
Converting objects to a transmittable format (**serialize**: object → JavaScript Object Notation [JSON] string/bytes) and back (**deserialize**). AWS APIs use JSON; Lambda events arrive as JSON; Kinesis/SQS payloads are strings/bytes (often JSON or **base64**-encoded binary); DynamoDB has its own typed JSON (`{"S": "text"}`, `{"N": "5"}`) that SDK document clients convert for you. **Avro/Protobuf** are compact binary formats with schemas.

### Base64
Encoding binary data as text. Kinesis records delivered to Lambda and API Gateway binary payloads are base64-encoded; decode before use.

### Environment variables and configuration
Values given to the app at runtime (table name, stage, feature flags) instead of hard-coding. Secrets go in Secrets Manager/Parameter Store, not plain env vars.

### Dependency management and packaging
Your code plus its libraries (`requirements.txt`, `package.json`, `pom.xml`) bundled into a **deployment artifact** (zip, JAR, container image). Lock versions for repeatable builds.

### Containers and images
An **image** packages app + runtime + libraries (built from a **Dockerfile**); a **container** is a running image. Images live in registries (ECR) and are identified by **tags** (`v1.4.2`, `latest`) and immutable **digests** (`sha256:…`).

---

## Data concepts

### Relational vs NoSQL
**Relational** (Relational Database Service [RDS], Aurora): tables, Structured Query Language (SQL), joins, transactions. **Not only SQL (NoSQL) key-value/document** (DynamoDB): access by key at any scale, flexible attributes, design tables around queries.

### Partition key and cardinality
DynamoDB spreads data across partitions by hashing the **partition key**. **High cardinality** (many distinct values like `userId`) spreads load; **low cardinality** (like `status`) creates **hot partitions** and throttling.

### Read consistency
**Eventually consistent** reads may return slightly stale data (cheaper, default). **Strongly consistent** reads return the latest committed write (cost double, not available on Global Secondary Indexes [GSIs]).

### Caching
Storing frequently read data in fast memory or at the edge. Strategies: **lazy loading / cache-aside** (fill on miss), **write-through** (update cache on every write), **Time To Live (TTL)** expiry. Trade-off: speed vs staleness.

### TTL and data lifecycle
**Time to live**: automatic expiry of data (DynamoDB TTL, cache TTL). **Lifecycle policies** move or delete data as it ages (S3 Lifecycle, CloudWatch Logs retention).

### Streaming data
Continuous flows of records (clicks, IoT) processed in order per key, with retention so consumers can replay (Kinesis Data Streams, DynamoDB Streams, Kafka).

---

## Security concepts

### Authentication vs authorization
**Authentication**: who you are (password, token, keys). **Authorization**: what you can do (Identity and Access Management [IAM] policies, scopes, groups).

### Principal, policy and least privilege
A **principal** is who calls (user, role, service). A **policy** (JSON) allows/denies actions on resources. **Least privilege** = only what's needed.

### Temporary credentials
Short-lived access key + secret + **session token** from Security Token Service [STS] (roles). Safer than long-term access keys; SDKs load them automatically from the environment, instance metadata, container credentials or Single Sign-On (SSO).

### Federation, OAuth 2.0, OIDC, SAML and JWT
**Federation**: use an existing identity provider (Google, corporate directory). **Open Authorization (OAuth) 2.0**: authorization framework issuing **access tokens** with **scopes**. **OpenID Connect (OIDC)**: identity layer on OAuth 2.0 issuing **ID tokens**. **Security Assertion Markup Language (SAML) 2.0**: XML-based enterprise federation. **JWT (JSON Web Token)**: signed token `header.payload.signature` with claims (`sub`, `iss`, `aud`, `exp`); verify signature with the issuer's public keys (JWKS) and check expiry/audience.

### MFA
Multi-factor authentication: something you know + something you have (authenticator code, passkey).

### Encryption at rest and in transit, symmetric and asymmetric
**At rest**: stored data encrypted (KMS keys). **In transit**: TLS/Hypertext Transfer Protocol Secure (HTTPS). **Symmetric**: same key encrypts/decrypts (AES). **Asymmetric**: public/private key pair (Rivest–Shamir–Adleman [RSA]/Elliptic Curve Cryptography [ECC]) for encryption or signing.

### Envelope encryption
Encrypt data with a **data key**; encrypt that data key with a **KMS key**; store the encrypted data key with the data. KMS only handles small keys, so you can encrypt any size locally.

### Client-side vs server-side encryption
**Server-side**: the service encrypts after receiving the data (S3 SSE). **Client-side**: you encrypt before sending, so the service only sees ciphertext (AWS Encryption SDK, S3 Encryption Client).

### Certificates and SSH keys
A **Transport Layer Security (TLS) certificate** binds a domain to a public key, signed by a certificate authority (public Certificate Authority [CA] or your private CA). A **Secure Shell (SSH) key pair** (private key kept by you, public key on the server) authenticates SSH logins; generate with `ssh-keygen` or Elastic Compute Cloud (EC2) key pairs.

### Hashing and HMAC
**Hashing** (SHA-256) = one-way fingerprint for integrity. **Hash-based Message Authentication Code (HMAC)** = hash with a secret key, proving a message came from someone with the key (used in Signature Version 4 [SigV4] and webhooks).

### PII and PHI, masking and redaction
**Personally Identifiable Information (PII)**: data identifying a person; **Protected Health Information (PHI)**: health data. **Masking** hides part (`***-**-1234`); **redaction** removes it; **tokenization** replaces it with a reference.

---

## Delivery concepts

### CI/CD
**Continuous integration**: every change is built and tested automatically. **Continuous delivery/deployment**: every passing change is released to environments automatically (with or without a manual approval to production).

### Environments and stages
Separate copies of the app (dev, test, staging, prod) with their own configuration and data so changes are verified before reaching users.

### Unit, integration and end-to-end tests
**Unit**: one function in isolation (dependencies mocked). **Integration**: components together with real services. **End-to-end**: the whole user flow. **Regression**: re-running tests to catch old bugs returning. **Mock**: a fake dependency returning canned responses.

### Deployment strategies
**All-at-once**, **rolling** (batches), **immutable** (new instances), **blue/green** (switch traffic between two full environments), **canary** (small % first), **linear** (equal steps). Rollback = return traffic/version to the previous good state.

### Infrastructure as Code (IaC)
Defining infrastructure in templates/code (CloudFormation, Serverless Application Model [SAM], Cloud Development Kit [CDK]) so environments are repeatable and versioned.

### Source control
**Git**: commits, **branches** (feature work), **tags** (release labels like `v2.0.1`), **pull requests** (review before merge), merge strategies.

### Semantic versioning
`MAJOR.MINOR.PATCH`: major = breaking change, minor = new feature, patch = bug fix.

---

## Observability concepts

### Logs, metrics and traces
**Logs**: event records (prefer **structured** JSON with request IDs). **Metrics**: numbers over time (latency, errors). **Traces**: a request's path across services with timing (X-Ray, OpenTelemetry). Together they answer "what broke, where and why."

### Correlation ID
A unique ID passed through all services for one request so its logs and traces can be joined.

### Health checks, liveness and readiness
**Liveness**: is the process alive (restart if not)? **Readiness**: is it ready to receive traffic (e.g., finished loading, DB reachable)? Load balancers and Kubernetes use these probes.

### Percentiles
p50/p90/p99 latency show typical and worst-case user experience better than averages.

### Concurrency, thread pools and connection pools
**Concurrency** = how many requests are handled at once. A **thread pool** limits and reuses worker threads; a **connection pool** reuses database connections instead of opening new ones per request.

---

## AI concepts for developers

### Foundation model, LLM and tokens
A **foundation model** is a large pre-trained model; a **Large Language Model (LLM)** is a foundation model for text. Input and output are measured in **tokens** (word pieces); pricing, limits and latency depend on token counts.

### Prompt, system prompt and inference parameters
The **prompt** is the input; a **system prompt** sets behavior rules. Parameters: **temperature** (randomness), **top-p**, **max tokens**, **stop sequences**.

### Streaming responses
Returning model output token by token as it's generated (Bedrock `ConverseStream`/`InvokeModelWithResponseStream`) for responsive UIs.

### RAG
**Retrieval-Augmented Generation**: retrieve relevant documents (via embeddings/vector search) and add them to the prompt so the model answers from your data.

### Embeddings
Number vectors representing meaning; similar texts have nearby vectors. Used for semantic search and Retrieval-Augmented Generation (RAG).

### AI agents, tools and human-in-the-loop
An **agent** uses a model to plan and call **tools** (APIs, functions) over several steps. **Tool-use authorization** limits which tools/actions it may call with which permissions; **session isolation** keeps one user's context and data separate from another's; **human-in-the-loop** requires a person to approve sensitive actions.

### Guardrails
Policies that filter model inputs/outputs: harmful content categories, **denied topics**, **sensitive information (PII) detection/redaction**, word filters, **prompt attack (injection/jailbreak) detection**, grounding checks.

### Prompt injection
An attack where input text (from a user or a retrieved document) tries to override the model's instructions ("ignore previous instructions and …"). Mitigate with guardrails' prompt attack filters, separating instructions from data, least-privilege tools and human approval.

### AI-assisted development
Using AI tools to generate code from specifications (**spec-driven development**), complete code, review it, suggest refactors, scan for security issues, write tests, analyze errors and logs. **Kiro** is AWS's agentic development environment for this.

### MCP
**Model Context Protocol**: open standard for connecting AI agents/assistants to tools and data sources.
