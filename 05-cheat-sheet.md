# DVA-C03 Cheat Sheet

> 🔰 New to a term? Concepts link to plain-English explanations in [the basics page](00-developer-basics.md) on first mention, and every abbreviation is spelled out on first use and listed in the [glossary](00-abbreviations-glossary.md).

> Rapid review: "if the scenario says X → do Y," top traps, numbers. 65 questions (50 scored), 130 minutes, pass 720/1000. Domains: Development 30%, Security 26%, Testing & Deployment 22%, Troubleshooting & Optimization 22%. DVA-C03 is in use from December 1, 2026.

## Keyword → answer decoder

### Development

| If the scenario says… | Do… |
|---|---|
| Decouple producer from slow consumer | **Simple Queue Service (SQS)** |
| One event, many independent consumers | **Simple Notification Service (SNS) → SQS fan-out** (or EventBridge rule with multiple targets) |
| Route events by content, Software as a Service (SaaS) events | **EventBridge** |
| Ordered per entity, exactly-once | **SQS First-In, First-Out [FIFO]** (`MessageGroupId`) |
| Real-time ordered stream, replay, many consumers | **Kinesis Data Streams** |
| Multi-step workflow with retries and approval | **Step Functions** (task token) |
| Messages processed twice | Raise **visibility timeout**; [idempotency](00-developer-basics.md#idempotency) |
| Empty receives cost | **Long polling** (20 s) |
| Payload > SQS limit | **Simple Storage Service (S3) + pointer** (Extended Client) |
| Throttling errors from Amazon Web Services (AWS) Application Programming Interfaces (APIs) | **Exponential [backoff](00-developer-basics.md#retries-exponential-backoff-and-jitter) + jitter** ([Software Development Kit [SDK]](00-developer-basics.md#sdk-and-cli) retries) |
| Reject bad requests before Lambda | **[Application Programming Interface [API]](00-developer-basics.md#api) Gateway request validator + model** |
| Transform request/response | **Mapping templates (VTL)** |
| Browser Cross-Origin Resource Sharing (CORS) error with proxy integration | **Lambda returns CORS headers** |
| Put API data straight into SQS/DynamoDB | **API Gateway AWS service integration** |
| No access keys on Elastic Compute Cloud (EC2) | **Instance profile role** |
| Wait until a resource is ready | **SDK waiter** |
| Large list results | **Paginator** |
| Lambda event `body` won't parse | It's a **JavaScript Object Notation (JSON) string** |
| Kinesis data unreadable | **[Base64](00-developer-basics.md#base64) decode** |
| Thumbnails on upload | **S3 event → Lambda** (different output prefix) |
| [Spec-driven](00-developer-basics.md#ai-assisted-development) Artificial Intelligence (AI) code generation | **Kiro specs** (requirements → design → tasks) |
| Same code for multiple models + tool use | **Bedrock Converse API** |
| Q&A over company docs with citations | **Bedrock Knowledge Bases** |
| Agent calls our API with confirmation | **Bedrock Agents action group** (user confirmation/return of control) |
| Deploy/run agents with session isolation and managed tools | **Bedrock AgentCore** |
| Cheap large offline inference | **Bedrock batch inference** |
| Java cold starts | **SnapStart** / provisioned concurrency |
| Protect downstream DB from Lambda scale | **Reserved concurrency** / Event Source Mapping (ESM) max concurrency |
| Shared libraries | **Layers** (`/opt`, max 5) |
| Lambda reach Relational Database Service (RDS) privately | **Virtual Private Cloud (VPC) config + Security Group (SG) + RDS Proxy** |
| VPC Lambda can't reach S3/internet | **Gateway endpoint** / **Network Address Translation (NAT) gateway** |
| Async failure details | **On-failure destination** |
| Poison record blocks stream | **Bisect + max retries/age + on-failure destination** |
| Retry only failed SQS messages | **Partial batch response** |
| S3/SNS/EventBridge can't invoke Lambda | **Resource-based policy** |
| Auto-delete old items free | **DynamoDB [Time To Live [TTL]](00-developer-basics.md#ttl-and-data-lifecycle)** (epoch seconds) |
| Query by customer + date range | **PK customerId, SK date, Query BETWEEN** |
| Query by another attribute | **Global Secondary Index (GSI)** |
| [Strongly consistent](00-developer-basics.md#read-consistency) alternate sort key | **Local Secondary Index [LSI]** (at table creation) |
| Prevent overwrite / optimistic locking | **Condition expression** (`attribute_not_exists` / version) |
| All-or-nothing multi-item write | **TransactWriteItems** |
| Microsecond DynamoDB reads | **DynamoDB Accelerator (DAX)** |
| Cache Structured Query Language (SQL) query results | **ElastiCache** [lazy loading](00-developer-basics.md#caching) |

### Security

| If the scenario says… | Do… |
|---|---|
| App sign-in with Google/[Security Assertion Markup Language [SAML]](00-developer-basics.md#federation-oauth-20-oidc-saml-and-jwt) | **Cognito user pool** (federation) |
| Users need AWS credentials (upload to S3) | **Cognito identity pool** |
| Users access only their own items | **`dynamodb:LeadingKeys`** / `${cognito-identity.amazonaws.com:sub}` |
| Custom tokens authorize API | **Lambda authorizer** |
| Add claims to tokens | **Pre token generation trigger** |
| CI from GitHub without keys | **OpenID Connect (OIDC) provider + role** |
| Command Line Interface (CLI) without long-term keys | **Identity Center Single Sign-On (SSO) profile** |
| Call IAM-protected API from code | **[Signature Version 4 [SigV4]](00-developer-basics.md#request-signing-sigv4) signing** |
| Temporary download/upload link | **S3 presigned Uniform Resource Locator [URL]** (expires with signer's creds) |
| Many private files via Content Delivery Network (CDN) | **CloudFront signed cookies** |
| Lambda AccessDenied | **Execution role** permissions |
| Elastic Container Service (ECS) can't pull image/secrets | **Task execution role** |
| Cross-account access | **AssumeRole** (trust + permission) / resource policy |
| Third-party access | Role + **external ID** |
| Encrypt > 4 KB with Key Management Service (KMS) | **GenerateDataKey** (envelope) / Encryption SDK |
| KMS throttling | **Data key caching**, S3 Bucket Keys |
| Bind ciphertext to tenant | **Encryption context** |
| AWS must never see plaintext | **[Client-side](00-developer-basics.md#client-side-vs-server-side-encryption) encryption** |
| Internal mutual TLS (mTLS) certs | **AWS Private Certificate Authority (CA)** |
| Local Hypertext Transfer Protocol Secure (HTTPS) dev cert | **Self-signed (OpenSSL)** |
| Avoid Secure Shell (SSH) key distribution | **Session Manager / EC2 Instance Connect** |
| [Personally Identifiable Information [PII]](00-developer-basics.md#pii-and-phi-masking-and-redaction) in logs | **CloudWatch Logs data protection** (mask) |
| Redact PII on S3 reads | **S3 Object Lambda** |
| Rotate DB password | **Secrets Manager** |
| Cheap config | **Parameter Store** |
| Only approved models | **Service Control Policy (SCP)/IAM on model Amazon Resource Names (ARNs)** |
| Bedrock traffic private | **Interface VPC endpoint (PrivateLink)** |
| Prompts used for training? | **No** (Bedrock doesn't train on customer data) |
| Block topics / mask PII / stop [prompt injection](00-developer-basics.md#prompt-injection) | **Bedrock [Guardrails](00-developer-basics.md#guardrails)** (denied topics, sensitive info, prompt attack) |
| Guardrails for non-Bedrock model | **ApplyGuardrail API** |
| Require guardrail on every call | **`bedrock:GuardrailIdentifier` condition** |
| Agent actions only with user's permissions | **AgentCore Identity / Open Authorization (OAuth) propagation** |
| Approve before sensitive agent action | **Human-in-the-loop** (user confirmation, return of control, task token) |
| Invocation logs contain PII | **Encrypt (KMS) + mask + restrict + retention** |

### Testing and deployment

| If the scenario says… | Do… |
|---|---|
| Share internal packages, cache public ones | **CodeArtifact** |
| Prevent overwriting image tags | **Elastic Container Registry (ECR) tag immutability** |
| Old images pile up | **ECR lifecycle policy** |
| Docker build fails in CodeBuild | **Privileged mode** |
| Feature flags with gradual rollout + rollback | **AppConfig** |
| Different Lambda alias per API stage | **Stage variables** (+ Lambda permission per alias) |
| API changes not visible | **Deploy to stage** |
| Test new API version on % of traffic | **Stage [canary](00-developer-basics.md#deployment-strategies)** |
| Private API | **Private endpoint + resource policy** |
| Run Lambda locally with sample event | **`sam local generate-event` + `sam local invoke`** |
| Test Cloud Development Kit (CDK) output | **CDK assertions / snapshot tests** |
| API before backend exists | **API Gateway [mock](00-developer-basics.md#unit-integration-and-end-to-end-tests) integration** |
| Branch preview [environments](00-developer-basics.md#environments-and-stages) | **Amplify [branches](00-developer-basics.md#source-control)/Pull Request (PR) previews** |
| Simple multi-environment ECS deploys | **AWS Copilot** |
| AI-generated tests in pipeline | **Kiro (hooks/specs/headless) + CodeBuild reports** |
| Package > 250 MB | **Container image** (10 GB) |
| ECS zero-downtime update | New **task definition revision** + rolling (healthy %) |
| ECS auto rollback | **Deployment [circuit breaker](00-developer-basics.md#timeouts-and-circuit-breakers)** |
| Gradual Lambda deploy in Serverless Application Model (SAM) | **AutoPublishAlias + DeploymentPreference** |
| EC2 deploy one instance at a time | CodeDeploy **OneAtATime** |
| Validation after install | **ValidateService** hook |
| Lambda canary with pre-traffic tests | **BeforeAllowTraffic** hook |
| Manager approval before prod | **CodePipeline manual approval** |
| Secrets in build | buildspec **`env.secrets-manager`** |
| Build reaches private DB | **CodeBuild VPC config** |
| Beanstalk safest deploy | **Immutable** (or blue/green Canonical Name [CNAME] swap) |
| Beanstalk full capacity, cheap | **Rolling with additional batch** |
| Roll back on alarm | **CodeDeploy alarms / CloudFormation rollback triggers** |
| Instant Lambda rollback | **Move alias** |
| Deploy on release tag | **CodePipeline V2 tag trigger** |
| AI review before approval | **Bedrock/Kiro step + manual approval** |

### Troubleshooting and optimization

| If the scenario says… | Do… |
|---|---|
| API Gateway 502 | Fix **Lambda proxy response format** |
| API Gateway 504 | > **29 s** → async/stream/optimize |
| Cold start delays | `Init Duration` → **provisioned concurrency/SnapStart** |
| Out of memory / slow Central Processing Unit (CPU) | **Raise memory** |
| Find slow downstream call | **X-Ray** subsegments |
| Filter [traces](00-developer-basics.md#logs-metrics-and-traces) by customer | **Annotations** |
| Find requests across services | **Logs Insights + [correlation ID](00-developer-basics.md#correlation-id)** |
| [p99](00-developer-basics.md#percentiles) latency from logs | `stats pct(@duration, 99)` |
| Stack failed | **Stack events** (first failure) |
| ECS `CannotPullContainerError` | **Execution role / ECR endpoints / NAT** |
| Business metrics from Lambda cheaply | **Embedded Metric Format (EMF)** |
| Alert near Lambda concurrency limit | **Service Quotas usage alarm** |
| Notify on pipeline failure | **Notification rule / EventBridge → SNS** |
| Lambda traces missing | **Active tracing + X-Ray permissions** |
| Pods get traffic too early | **Readiness probe** |
| Concurrency needed | **Requests per second (rps) × duration** |
| DB connections exhausted | **RDS Proxy** |
| Cost-optimal memory | **Lambda Power Tuning / Compute Optimizer** |
| Hot DynamoDB keys | **Contributor Insights** |
| Per-function CPU/memory | **Lambda Insights** |
| AI root-cause analysis | **Kiro / DevOps Agent / CloudWatch investigations** |

## Top traps

1. Proxy response needs `statusCode` + string `body`.
2. API Gateway **29 s** / **10 MB**; Lambda sync **6 MB**, async **1 MB**.
3. **Destinations** carry context; Dead-Letter Queue (DLQ) doesn't.
4. Streams block on errors → bisect/max age/destination.
5. VPC Lambda has no internet without **NAT**.
6. **API keys** aren't auth.
7. **Execution role** vs **resource policy**; **task role** vs **task execution role**.
8. Presigned URL dies when signer's **[temporary credentials](00-developer-basics.md#temporary-credentials)** expire.
9. KMS **Encrypt ≤ 4 KB**.
10. **Scan** reads everything; **FilterExpression** doesn't cut Read Capacity Unit (RCU).
11. **GSI** eventual only; **LSI** at creation only.
12. RCU 4 KB / Write Capacity Unit (WCU) 1 KB; transactions ×2; eventual ×0.5.
13. [Representational State Transfer [REST]](00-developer-basics.md#rest-and-http-methods) API changes need **deployment**.
14. **appspec** (CodeDeploy) ≠ **buildspec** (CodeBuild).
15. Native deps must match **Lambda arch/Operating System (OS)**.
16. Annotations indexed; metadata not.
17. Lazy loading → stale; write-through → fresh.
18. Bedrock doesn't train on your data; use **PrivateLink** for private access.

## Numbers

| Item | Value |
|---|---|
| Lambda memory / timeout / `/tmp` | 128 MB–10 GB / 15 min / 512 MB–10 GB |
| Lambda env vars | 4 KB total |
| Lambda package | 50 MB zipped direct, 250 MB unzipped, image 10 GB, 5 layers |
| Lambda payload | sync 6 MB, async 1 MB (formerly 256 KB) |
| Lambda concurrency | 1,000 default per Region; +1,000 per 10 s per function |
| Lambda async retries / event age | 2 / up to 6 h |
| API Gateway | 29 s, 10 MB, 10,000 rps (5,000 burst), cache TTL default 300 s max 3,600 |
| SQS | 1 MiB (formerly 256 KB), visibility 30 s default / 12 h max, retention 4 days / 14 max, long poll 20 s, delay 15 min, batch 10 |
| SQS FIFO | 300 Transactions Per Second [TPS] (3,000 batched); dedup window 5 min |
| SNS | message 256 KB; PublishBatch 10 |
| EventBridge PutEvents | 10 entries per call |
| Kinesis shard | 1 MB/s or 1,000 rec/s in; 2 MB/s out; 5 reads/s; retention 24 h–365 days; PutRecords 500 |
| DynamoDB | item 400 KB; Query/Scan 1 MB pages; BatchGet 100/16 MB; BatchWrite 25; transactions 100 items; 20 Global Secondary Indexes (GSIs), 5 Local Secondary Indexes (LSIs); partition 3,000 RCU/1,000 WCU |
| Step Functions | Standard 1 year; Express 5 min; payload 256 KB |
| Cognito tokens | ID/access 1 h default; refresh 30 days default |
| Security Token Service (STS) | 15 min–12 h; role chaining 1 h |
| S3 presigned URL | up to 7 days (SigV4, Identity and Access Management [IAM] user) |
| KMS | Encrypt 4 KB |
| Parameter Store / Secrets Manager | 4 KB (8 KB adv) / 64 KB |
| CodeBuild timeout | 60 min default, up to 36 h |
| Lambda code storage | 75 GB per Region |
