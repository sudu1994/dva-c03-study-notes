# Domain 1: Development with AWS Services (30%)

> 🔰 New to a term? Concepts link to plain-English explanations in [the basics page](00-developer-basics.md) on first mention, and every abbreviation is spelled out on first use and listed in the [glossary](00-abbreviations-glossary.md).

> **About 15 of the 50 scored questions. The biggest domain.**
>
> Tests whether you can write application code that uses Amazon Web Services (AWS) well: decoupled and resilient designs, Application Programming Interfaces (APIs), [Software Development Kit [SDK]](00-developer-basics.md#sdk-and-cli) calls, [streams](00-developer-basics.md#streaming-data) and events, Artificial Intelligence (AI) services and [AI-assisted](00-developer-basics.md#ai-assisted-development) coding, Lambda functions, and data stores (especially DynamoDB) with caching.
>
> Tags like `[also: DOP]` mark material you met in another exam. It is still covered in full here.

## Table of contents

- [Exam tasks and skills](#exam-tasks-and-skills)
- [Task 1.1: Develop code for applications hosted on AWS](#task-11-develop-code-for-applications-hosted-on-aws)
- [Task 1.2: Develop code for AWS Lambda](#task-12-develop-code-for-aws-lambda)
- [Task 1.3: Use data stores in application development](#task-13-use-data-stores-in-application-development)
- [Gotchas and exam traps](#gotchas-and-exam-traps)
- [Quick-recall summary](#quick-recall-summary)

## Exam tasks and skills

> Summarized from the official AWS Certified Developer – Associate (DVA-C03) exam guide (in use from December 1, 2026; DVA-C02 ends November 30, 2026). Exact wording: https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html

### Task 1.1: Develop code for applications hosted on AWS
- **1.1.1** Identify tightly and [loosely coupled](00-developer-basics.md#tightly-vs-loosely-coupled) systems; decouple components with AWS services.
- **1.1.2** Choose synchronous vs [asynchronous](00-developer-basics.md#synchronous-vs-asynchronous) patterns ([Application Programming Interface [API]](00-developer-basics.md#api) Gateway sync, Simple Queue Service [SQS] async, EventBridge [event-driven](00-developer-basics.md#event-driven-architecture)).
- **1.1.3** Build fault-tolerant, resilient apps (retry with [backoff](00-developer-basics.md#retries-exponential-backoff-and-jitter), [graceful degradation](00-developer-basics.md#graceful-degradation)).
- **1.1.4** Create, extend and maintain APIs (request/response transformations, validation rules).
- **1.1.5** Write code that calls AWS services with APIs and Software Development Kits (SDKs).
- **1.1.6** Manage [streaming](00-developer-basics.md#streaming-responses) data with AWS services.
- **1.1.7** Serialize and [deserialize](00-developer-basics.md#serialization) data to integrate with AWS services.
- **1.1.8** Implement event-driven patterns (EventBridge, Simple Storage Service [S3] Event Notifications, SQS, Simple Notification Service [SNS]).
- **1.1.9** Use AI-assisted development tools (spec-driven generation, automated reviews, code completion, refactoring, security scanning).
- **1.1.10** Integrate managed AWS AI services into applications.

### Task 1.2: Develop code for AWS Lambda
- **1.2.1** Give Lambda access to private resources in Virtual Private Clouds (VPCs).
- **1.2.2** Configure [environment variables](00-developer-basics.md#environment-variables-and-configuration) and parameters (memory, concurrency, timeout, runtime, handler, layers, extensions).
- **1.2.3** Handle event lifecycles and errors (destinations, dead-letter queues).
- **1.2.4** Integrate Lambda with services via code, event source mappings and destinations (API Gateway, DynamoDB).

### Task 1.3: Use data stores during application development
- **1.3.1** Manage data lifecycles (S3 Lifecycle, [Time To Live [TTL]](00-developer-basics.md#ttl-and-data-lifecycle), retention periods).
- **1.3.2** Choose data stores (ElastiCache, DynamoDB, S3, Relational Database Service [RDS]) by access pattern.
- **1.3.3** Choose caching for the chosen data store.
- **1.3.4** Configure DynamoDB ([high-cardinality](00-developer-basics.md#partition-key-and-cardinality) partition keys, consistency, query vs scan, keys, attributes, indexes).

---

## Task 1.1: Develop code for applications hosted on AWS

### Decoupling services at a glance (skills 1.1.1, 1.1.2, 1.1.8) `[also: SAA, DOP]`

| Service | Model | Delivery | Ordering | Use when |
|---|---|---|---|---|
| **SQS standard** | Queue (pull) | At least once | Best effort | Buffer work, absorb spikes, one consumer per message |
| **SQS First-In, First-Out (FIFO)** | Queue (pull) | Exactly-once processing (dedup 5 min) | Per message group | Ordered processing per entity |
| **SNS** | Pub/sub (push) | At least once, retries per protocol | FIFO topics only | Fan-out to many subscribers |
| **EventBridge** | Event bus + rules (push) | At least once, retries 24 h | No | Content-based routing, AWS/SaaS events, schedules |
| **Kinesis Data Streams** | Stream (pull, replay) | At least once | Per shard (partition key) | Real-time ordered streams, multiple consumers, replay |
| **Step Functions** | Orchestration (stateful) | Exactly once (Standard) | Defined by workflow | Multi-step processes with retries, branching, approvals |
| **API Gateway → Lambda** | Synchronous request/response | n/a | n/a | Client needs immediate answer |

- **Sync vs async decision**: if the client must wait for the result and it's fast (< 29 s), synchronous API. If it's slow, bursty or the client doesn't need the result now → accept, enqueue (SQS) or emit an event (EventBridge), return 202 + an ID; client polls or gets a [WebSocket](00-developer-basics.md#graphql-and-websocket)/notification.
- **Fan-out**: SNS topic → several SQS queues (each consumer processes independently); or EventBridge rule with multiple targets.
- **Choreography vs orchestration**: services reacting to each other's events (EventBridge) vs a central coordinator (Step Functions).

### Amazon SQS for developers (skills 1.1.1, 1.1.2) `[also: SAA, DOP]`

- **API**: `SendMessage`/`SendMessageBatch` (up to 10), `ReceiveMessage` (up to 10 per call; `WaitTimeSeconds` up to 20 = **long polling**; `VisibilityTimeout` override; `MessageAttributeNames`), `DeleteMessage`/`DeleteMessageBatch` (with **receipt handle**), `ChangeMessageVisibility` (extend while still working), `PurgeQueue`.
- **Settings**: visibility timeout (default 30 s, max 12 h; set ≥ processing time, for Lambda ≥ 6× function timeout), retention 1 min–14 days (default 4), **delay** 0–15 min (queue or per message), max message size **1 MiB** (formerly 256 KB; larger payloads → **Extended Client Library** storing body in S3), **message attributes** (metadata, up to 10), **Dead-Letter Queue (DLQ)** with `maxReceiveCount` and redrive, **Server-Side Encryption with SQS-owned keys (SSE-SQS)/Server-Side Encryption with KMS keys (SSE-KMS)**, access policy.
- **FIFO**: `MessageGroupId` (ordering scope; parallelism across groups), `MessageDeduplicationId` or **content-based deduplication**; 300 Transactions Per Second (TPS) per API action (3,000 messages with batching), **high-throughput mode** for much more.
- **Patterns**: [idempotent](00-developer-basics.md#idempotency) consumers (at-least-once), batch for cost, long polling for cost, DLQ + alarm on `ApproximateNumberOfMessagesVisible` of the DLQ, temporary queues for request/response.
- **Exam triggers:** "messages processed twice" → visibility timeout too short / not deleted / make idempotent. "Reduce empty receives and cost" → **long polling**. "Delay processing of each message 5 minutes" → **delay queue/message timer**. "Payload 5 MB" → **S3 + pointer** (Extended Client).

### Amazon SNS for developers (skill 1.1.8) `[also: SAA]`

- `Publish` (to topic Amazon Resource Name [ARN], or to phone number/endpoint), `PublishBatch` (10), `Subscribe` (protocols: SQS, Lambda, HTTP/S, email, SMS, mobile push, Firehose), **subscription filter policies** on message attributes or **message body** (exact, prefix, numeric, anything-but, exists), **raw message delivery** (SQS/HTTP get the body without the SNS JavaScript Object Notation [JSON] envelope), **FIFO topics** (ordered fan-out to SQS FIFO), DLQ per subscription, **message data protection** (detect/mask [Personally Identifiable Information [PII]](00-developer-basics.md#pii-and-phi-masking-and-redaction) in messages), delivery status logging.
- **Exam triggers:** "only send `order_type = premium` messages to this queue" → **filter policy**. "Consumers get JSON envelope they don't want" → **raw message delivery**.

### Amazon EventBridge for developers (skills 1.1.2, 1.1.8) `[also: DOP]`

- `PutEvents` (up to 10 entries per call; check `FailedEntryCount` in the response and retry failed entries), event structure (`source`, `detail-type`, `detail`, `resources`, `time`), custom buses, **rules** with **event patterns** (prefix, suffix, numeric, anything-but, exists, wildcard, `$or`), up to 5 targets per rule, **input transformers**, **archive and replay**, **schema registry** (download code bindings for events), **API destinations** (call external Hypertext Transfer Protocol [HTTP] APIs with connections holding auth), **EventBridge Pipes** (source → filter → enrichment → target, e.g., SQS → Lambda enrichment → Step Functions), **EventBridge Scheduler** (one-time/recurring invocations of 270+ targets).
- **Exam triggers:** "route events by content to different services without code" → **EventBridge rules**. "React to a Software as a Service (SaaS) partner's events" → partner event bus. "Re-process last week's events after a bug fix" → **archive + replay**.

### AWS Step Functions for developers (skill 1.1.2) `[also: SAA, DOP]`

- Amazon States Language (JSON/YAML Ain't Markup Language [YAML]) or Workflow Studio; states: Task, Choice, Parallel, **Map** (inline or **Distributed** for millions of items), Wait, Pass, Succeed, Fail.
- **Standard** (up to 1 year, exactly-once, history 90 days) vs **Express** (up to 5 min, at-least-once, high volume; sync or async invocation).
- **Error handling**: `Retry` (`ErrorEquals`, `IntervalSeconds`, `MaxAttempts`, `BackoffRate`, `JitterStrategy`) and `Catch` (fallback state); built-in errors `States.ALL`, `States.Timeout`, `States.TaskFailed`.
- **Integration patterns**: request-response, **`.sync`** (wait for job completion), **`.waitForTaskToken`** (pause until `SendTaskSuccess`/`SendTaskFailure` with the token: human approvals, external systems). **JSONata** or JSONPath (`InputPath`, `Parameters`, `ResultSelector`, `ResultPath`, `OutputPath`) to shape data. Direct SDK integrations to 200+ services (no Lambda glue).
- **Testing**: `TestState` API to test a single state; Step Functions Local (legacy).
- **Exam triggers:** "coordinate several Lambdas with retries and a manual approval step" → **Step Functions** (task token). "High-volume short workflows, lowest cost" → **Express**.

### Fault tolerance and resilience in code (skill 1.1.3) `[also: DOP]`

- **SDK retries**: automatic retries with exponential backoff and jitter for throttling and transient errors; configure `max_attempts` and `retry_mode` (`standard`, `adaptive` which adds [client-side](00-developer-basics.md#client-side-vs-server-side-encryption) rate limiting).
- **Throttling errors**: `ThrottlingException`, `ProvisionedThroughputExceededException` (DynamoDB), `TooManyRequestsException` (Lambda, 429), `SlowDown` (S3 503) → backoff; reduce rate; batch; request quota increase; spread keys.
- **[Timeouts](00-developer-basics.md#timeouts-and-circuit-breakers)**: set SDK connect/read timeouts shorter than the caller's timeout (e.g., Lambda behind API Gateway must finish < 29 s).
- **Idempotency**: idempotency tokens on APIs (`ClientToken` in Elastic Compute Cloud [EC2], `ClientRequestToken` in DynamoDB transactions/Step Functions), conditional writes, Powertools idempotency.
- **Graceful degradation**: fallback to cached/static data (CloudFront stale-if-error, ElastiCache), feature flags (AppConfig) to disable non-critical features.
- **Dead-letter queues and destinations**: capture failed events for later reprocessing (SQS DLQ, Lambda on-failure destination, EventBridge target DLQ, SNS subscription DLQ).
- **Bulkheads and limits**: Lambda reserved concurrency per function; SQS to smooth load.
- **Multi-AZ by default** services (Lambda, DynamoDB, SQS) and RDS Proxy for DB failover.
- **Exam triggers:** "intermittent `ThrottlingException`" → **exponential backoff with jitter**. "Keep app partially working when recommendation service is down" → **graceful degradation** (fallback/cached response).

### Amazon API Gateway: building APIs (skill 1.1.4) `[also: DOP, SAA]`

- **API types**: **[Representational State Transfer [REST]](00-developer-basics.md#rest-and-http-methods) API** (full features: request validation, mapping templates, usage plans/API keys, caching, Web Application Firewall [WAF], resource policies, private endpoints, [canary](00-developer-basics.md#deployment-strategies)), **HTTP API** (cheaper, faster, [JSON Web Token [JWT]](00-developer-basics.md#federation-oauth-20-oidc-saml-and-jwt) authorizers, simpler; parameter mapping but no Velocity Template Language [VTL]), **WebSocket API** (`$connect`, `$disconnect`, `$default`, custom routes by `routeSelectionExpression`; push to clients via `@connections` API `POST /@connections/{connectionId}`).
- **Endpoint types**: edge-optimized, regional, private (Virtual Private Cloud [VPC] interface endpoint).
- **Integrations**:
  - **Lambda proxy** (`AWS_PROXY`): whole request passed as event; function must return `{"statusCode": 200, "headers": {...}, "body": "string", "isBase64Encoded": false}` or the client gets **502**.
  - **Lambda non-proxy (custom)** / **AWS service** (`AWS`): use **mapping templates** to transform requests/responses; e.g., API Gateway → SQS `SendMessage` or DynamoDB `PutItem` directly.
  - **HTTP proxy / HTTP**: forward to an HTTP backend; **VPC Link** for private ALB/Network Load Balancer (NLB).
  - **[Mock](00-developer-basics.md#unit-integration-and-end-to-end-tests)**: return a response without a backend (prototypes, Cross-Origin Resource Sharing [CORS] preflight, contract testing).
- **Request/response transformations (REST)**: **mapping templates** in **VTL (Velocity Template Language)** with `$input.json('$.field')`, `$input.params('id')`, `$context.requestId`, `$util.escapeJavaScript()`; per content type; **gateway responses** customize error responses (e.g., add CORS headers to 4XX/5XX); **integration response** mapping by status regex. HTTP API uses **parameter mapping** (headers, query strings, paths).
- **Validation rules**: **request validators** (validate body against a **model** = JSON Schema, and/or required query strings/headers) → invalid requests get **400** without invoking the backend; models also generate SDKs.
- **CORS**: browser calls from another origin need `Access-Control-Allow-Origin` etc. REST: "Enable CORS" adds an `OPTIONS` mock method; with **proxy integrations, your Lambda must return the CORS headers**. HTTP API: built-in CORS config.
- **Other**: **binary media types**, payload limit **10 MB**, integration timeout **29 s** default (REST regional/private can request longer), **throttling** (10,000 requests per second [rps] / 5,000 burst account default; stage/method limits; usage plans per API key), **caching** (REST stage cache 0.5–237 GB, TTL default 300 s, max 3,600; per-method; clients can invalidate with `Cache-Control: max-age=0` if authorized), **custom domain names** + base path mappings, **OpenAPI** import/export, **SDK generation**.
- **Exam triggers:** "reject requests missing required fields before Lambda runs" → **request validator + model**. "Transform legacy XML backend response to JSON" → **mapping template**. "Browser gets CORS error with Lambda proxy" → **return CORS headers from Lambda** (and OPTIONS). "Write to SQS without Lambda" → **AWS service integration**.

### Calling AWS services with APIs and SDKs (skill 1.1.5) `[also: DOP]`

- **Credential provider chain** (SDK default order, roughly): explicit code credentials (avoid) → **environment variables** (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) → **web identity token** (Elastic Kubernetes Service [EKS] IAM Roles for Service Accounts [IRSA]/OpenID Connect [OIDC]) → shared **config/credentials files** (`~/.aws`, profiles, **Identity and Access Management [IAM] Identity Center/SSO** profiles, `credential_process`) → **container credentials** (Elastic Container Service [ECS] task role endpoint, EKS Pod Identity) → **EC2 instance metadata (IMDSv2)** role credentials. In Lambda, credentials come from the **execution role** via environment variables.
- **Region**: from client config, `AWS_REGION`/`AWS_DEFAULT_REGION`, profile.
- **SDK features**: **clients** (low-level, 1:1 with API) vs **resources/document clients** (higher-level, e.g., DynamoDB `DocumentClient`/`Table` that convert native types), **[paginators](00-developer-basics.md#pagination)**, **waiters** (poll until a state, e.g., `bucket_exists`, `instance_running`), **retries/backoff**, **error handling** (service error codes like `ConditionalCheckFailedException`, `NoSuchKey`, `AccessDenied`; HTTP status; request ID for support), **presigners** (S3/CloudFront URLs), **endpoint configuration** (VPC endpoints, Federal Information Processing Standards [FIPS], dual-stack), **S3 Transfer Manager** (multipart, parallel).
- **Initialize clients outside the Lambda handler** to reuse connections across invocations.
- **AWS Command Line Interface (CLI)**: profiles (`--profile`), `--query` (JSON Matching Expression Path [JMESPath] filtering), `--output json|table|text`, `--dry-run` (EC2 permission check), `--debug`, pagination flags, `aws configure sso`, **AWS CloudShell** (browser shell with CLI preinstalled and your console credentials; persistent 1 GB home directory per Region; can run in a VPC).
- **Exam triggers:** "app on EC2 should not store access keys" → **instance profile role** (SDK picks it up). "Local dev with Single Sign-On (SSO) credentials" → `aws configure sso` profile. "Wait until a resource is ready" → **waiter**. "List more than 1,000 S3 objects" → **paginator**.

### Managing streaming data (skill 1.1.6) `[also: DEA, SAA]`

- **Kinesis Data Streams (KDS)**:
  - **Shards**: each **1 MB/s or 1,000 records/s write**, **2 MB/s read** (shared by standard consumers, **5 GetRecords calls/s per shard**), or **on-demand** capacity mode.
  - **Producer**: `PutRecord`/`PutRecords` (up to 500 records, 5 MB per call; partial failures → check `FailedRecordCount` and retry failed records), **partition key** → shard (ordering per key; high-cardinality keys avoid hot shards), **sequence numbers**; **Kinesis Producer Library [KPL]** (batching, aggregation, retries; async), Kinesis Agent.
  - **Consumer**: **KCL** (Kinesis Client Library: shard leases and **checkpoints in DynamoDB**, one worker per shard lease), **Lambda event source mapping** (batch size, batching window, **parallelization factor** up to 10 per shard, starting position `TRIM_HORIZON`/`LATEST`/`AT_TIMESTAMP`, bisect on error, max retries, max record age, on-failure destination, **tumbling windows** for aggregation), **enhanced fan-out** (dedicated 2 MB/s per consumer via `SubscribeToShard` push; lower latency, up to 20 consumers).
  - **Retention** 24 h (default) to 365 days; **resharding** (split/merge) in provisioned mode; **records** base64-encoded in Lambda events.
  - Errors: `ProvisionedThroughputExceededException` → more shards/on-demand, better partition keys, backoff; consumer lag → `IteratorAgeMilliseconds` high → more shards/parallelization/EFO.
- **Amazon Data Firehose**: managed delivery (S3, Redshift, OpenSearch, Splunk, HTTP endpoints, Iceberg); buffering by size/time; **Lambda transformation** (records returned with `recordId`, `result` = `Ok`/`Dropped`/`ProcessingFailed`, `data` [base64](00-developer-basics.md#base64)); format conversion to Parquet/Optimized Row Columnar (ORC); dynamic partitioning; `PutRecord`/`PutRecordBatch` (500 records).
- **DynamoDB Streams**: ordered item-level change log (24 h) → Lambda Event Source Mapping (ESM); **stream view types** `KEYS_ONLY`, `NEW_IMAGE`, `OLD_IMAGE`, `NEW_AND_OLD_IMAGES`; use for triggers, replication, aggregations, audit. **Kinesis Data Streams for DynamoDB** for longer retention/more consumers.
- **Amazon OpenSearch Service** as a stream destination for search/log analytics (via Firehose or OpenSearch Ingestion).
- **Amazon Athena** to query streamed data landed in S3 (Structured Query Language [SQL], pay per scan).
- **Exam triggers:** "ordered processing per customer in real time with multiple consumers and replay" → **KDS with customerId partition key**. "Deliver clickstream to S3 in batches with least code" → **Firehose**. "React to DynamoDB item changes" → **DynamoDB Streams + Lambda**. "Consumer falling behind" → `IteratorAge` → more shards/parallelization factor/EFO.

### Serialization and deserialization (skill 1.1.7)

- **JSON everywhere**: Lambda event objects (API Gateway proxy event: `body` is a **string** → `json.loads(event["body"])`; response `body` must be a **string** → `json.dumps(...)`), SQS message `Body` (string; SNS-wrapped messages contain an inner JSON `Message` unless raw delivery), SNS `Message`, EventBridge `detail` (object), Step Functions state input/output (JSON; payload limit 256 KB → store large data in S3 and pass references).
- **Base64**: Kinesis `data` in Lambda events, Firehose transformation records, API Gateway binary bodies (`isBase64Encoded`), Key Management Service (KMS) `Encrypt` output (`CiphertextBlob` bytes, base64 in CLI).
- **DynamoDB typed JSON** (low-level API): `{"S": "a"}`, `{"N": "1"}` (numbers sent as strings), `{"BOOL": true}`, `{"M": {...}}`, `{"L": [...]}`, `{"SS": [...]}`, `{"B": ...}`; **document clients** (`boto3.resource('dynamodb').Table`, `DynamoDBDocumentClient` in JS v3) marshall/unmarshall native types (Python returns numbers as `Decimal`). DynamoDB Streams records arrive in typed JSON.
- **Dates/numbers**: use ISO 8601 strings or epoch seconds (DynamoDB **TTL requires epoch seconds Number**).
- **Schemas**: EventBridge schema registry code bindings; Avro/Protobuf with schema registries for Kafka; JSON Schema models in API Gateway.
- **Exam triggers:** "Lambda behind API Gateway fails parsing input" → `event.body` is a **JSON string**. "TTL not deleting items" → attribute not **epoch seconds Number**. "Kinesis data unreadable in Lambda" → **base64 decode**.

### Event-driven patterns (skill 1.1.8) `[also: DOP]`

- **S3 Event Notifications**: object created/removed/restored/replication/lifecycle/tagging events with **prefix/suffix filters** → **Lambda, SQS (standard), SNS**; or enable **EventBridge** for advanced filters, multiple targets, archive/replay. Destinations need resource policies allowing `s3.amazonaws.com`; no overlapping filter rules for the same event type; **avoid recursion** (writing outputs back to the triggering prefix); event delivery usually seconds, at least once; object key is **URL-encoded** in the event (decode `+` and `%xx`).
- **SQS-driven**: producers enqueue, Lambda ESM consumes in batches.
- **SNS fan-out** to SQS queues/Lambdas with filter policies.
- **EventBridge** content routing and scheduled events.
- **DynamoDB Streams / Kinesis** change-data and stream processing.
- **Transactional outbox** (write business data + event record in one DynamoDB transaction, stream publishes event) to avoid lost/phantom events.
- **Exam triggers:** "generate thumbnails when images are uploaded" → **S3 event → Lambda** (write to another bucket/prefix). "Several services must react to uploads with different filters" → **S3 → EventBridge**.

### AI-assisted development tools (skill 1.1.9)

- **Kiro** (AWS's agentic AI development environment: Integrated Development Environment [IDE], CLI, web):
  - **Specs** (spec-driven development): turn a prompt into **requirements** (user stories with acceptance criteria) → **design** (architecture, data models, interfaces) → **tasks** (implementation checklist), then generate code task by task with review; **bugfix specs** for defects.
  - **Steering** files (project rules: coding standards, tech stack, conventions in `.kiro/steering`), **hooks** (agent actions triggered by events such as file save, e.g., update tests/docs, run security scan), **[Model Context Protocol [MCP]](00-developer-basics.md#mcp)** servers (tool/data access), custom agents and sub-agents, checkpoints/rewind, **headless mode for CI**, cloud sessions; models include Claude and others; enterprise SSO/IAM controls.
- **What AI tools do for developers (exam vocabulary)**: **intelligent code completion**, **code generation from natural language or specs**, **automated code reviews** (bugs, style, best practices on pull requests), **refactoring suggestions**, **security scanning** (vulnerabilities, hard-coded secrets, injection risks), **explaining code**, **generating unit tests**, **upgrading/transforming code** (e.g., language/runtime version upgrades), **troubleshooting from errors/logs**, generating **[Infrastructure as Code [IaC]](00-developer-basics.md#infrastructure-as-code-iac)**.
- **Responsible use**: review generated code (correctness, security, licenses), keep secrets out of [prompts](00-developer-basics.md#prompt-system-prompt-and-inference-parameters), enforce steering rules/standards, run tests and scanners in CI, [least-privilege](00-developer-basics.md#principal-policy-and-least-privilege) tool permissions for agents.
- **Exam triggers:** "generate code from written requirements with a reviewable design and task list" → **Kiro specs**. "Automatically run a check whenever files are saved" → **Kiro hooks**. "Enforce team conventions in AI-generated code" → **steering**.

### Integrating managed AWS AI services (skill 1.1.10) `[also: AIP, AIF]`

- **Amazon Bedrock** (serverless access to [foundation models](00-developer-basics.md#foundation-model-llm-and-tokens) from Amazon (Nova), Anthropic (Claude), Meta (Llama), Mistral, Cohere, AI21, DeepSeek, OpenAI open-weight models and others):
  - **Model access**: enable/request access to models (some require agreeing to provider terms; IAM permissions `bedrock:InvokeModel` scoped to model Amazon Resource Names [ARNs]); **inference profiles** (cross-Region inference to route requests across Regions for throughput; application inference profiles for cost tracking with tags).
  - **APIs** (runtime): **`Converse`** / **`ConverseStream`** (unified message format across models, system prompts, multi-turn, **tool use/function calling**, [guardrails](00-developer-basics.md#guardrails), document/image inputs) and **`InvokeModel`** / **`InvokeModelWithResponseStream`** (model-specific JSON body). Parameters: `temperature`, `topP`, `maxTokens`, `stopSequences`. Response includes `usage` (input/output tokens) and `stopReason` (`end_turn`, `max_tokens`, `tool_use`, `guardrail_intervened`).
  - **Throughput**: on-demand (quotas on requests and tokens per minute → `ThrottlingException` → backoff/cross-Region inference/quota increase), **Provisioned Throughput** (reserved model units, required for some custom models), **batch inference** (asynchronous jobs on S3 data at lower cost), latency-optimized inference for some models, **prompt caching** (reuse long prompt prefixes to cut latency/cost).
  - **Bedrock Knowledge Bases**: managed **[Retrieval-Augmented Generation [RAG]](00-developer-basics.md#rag)**: ingest documents from S3 (and other sources) → chunk → embed → store in a vector store (OpenSearch Serverless, Aurora PostgreSQL pgvector, S3 Vectors, others) → `Retrieve` or `RetrieveAndGenerate` APIs with citations.
  - **Bedrock Agents**: an agent with instructions, **action groups** (Lambda functions or **return of control** to your app, defined by OpenAPI/function schemas), knowledge bases, memory, **user confirmation** before actions; `InvokeAgent`.
  - **Bedrock Guardrails** (Domain 2), **Prompt management** (versioned prompt templates), **Flows** (visual workflows linking prompts, KBs, Lambda), **model evaluation**, **custom models** (fine-tuning, continued pre-training, distillation; imported models).
- **Amazon Bedrock AgentCore** (platform to deploy and operate [AI agents](00-developer-basics.md#ai-agents-tools-and-human-in-the-loop) built with any framework, e.g., Strands, LangGraph, CrewAI, and any model):
  - **Runtime** (serverless, session-isolated hosting for agents/tools, long-running sessions), **Gateway** (turn APIs and Lambda functions into **MCP**-compatible tools for agents), **Identity** (agent identity and **Open Authorization [OAuth]/credential management** for calling tools on behalf of users, inbound auth with IdPs like Cognito), **Memory** (short- and long-term memory), **Code Interpreter** and **Browser** built-in tools (sandboxed), **Observability** ([traces](00-developer-basics.md#logs-metrics-and-traces)/metrics in CloudWatch via OpenTelemetry), plus **policy controls** on what tools an agent may call.
  - Exam angle: "deploy an agent securely at scale with per-session isolation, managed tool access and observability" → **AgentCore**.
- **Amazon Quick** (Amazon Quick Suite): AWS's AI-powered business workspace: chat agents over company data and connected apps, research, automated workflows (flows/automations), and Business Intelligence [BI] (Quick Sight; BI-only is out of scope for DVA). Developers integrate data sources/actions so business users can use agents on them.
- **Calling AI services from apps**: use the SDK `bedrock-runtime` client; stream responses to the User Interface [UI] (WebSocket API Gateway or Lambda response streaming / function URLs with `RESPONSE_STREAM`); handle throttling with backoff; set timeouts long enough for generation (Lambda up to 15 min, but API Gateway 29 s → streaming or async pattern); log token usage; keep prompts and outputs out of logs if sensitive (Domain 2).
- **Exam triggers:** "single code path that works across multiple models with tool use" → **Converse API**. "Answer questions over company documents in S3 with citations" → **Knowledge Bases (`RetrieveAndGenerate`)**. "Agent must call our order API, with user confirmation" → **Bedrock Agents action group** (or AgentCore Gateway). "Classify 1 million documents cheaply overnight" → **batch inference**. "Show tokens as they're generated" → **ConverseStream**.

---

## Task 1.2: Develop code for AWS Lambda

### Lambda configuration (skill 1.2.2) `[also: DOP, SAA]`

- **Runtime** (Node.js, Python, Java, .NET, Ruby, Go via `provided.al2023`, **custom runtimes** via Runtime API/bootstrap, **OS-only runtimes**), architecture **x86_64 or arm64 (Graviton, ~20% cheaper)**.
- **Handler**: `file.function` (Python `app.lambda_handler(event, context)`), Node `index.handler` (async/await), Java `package.Class::method`. The **context** object gives `aws_request_id`, `get_remaining_time_in_millis()`, `function_name`, memory limit, log group.
- **Init code** (outside handler) runs once per execution environment (cold start): create SDK clients, DB connections, load config → reused on warm invocations.
- **Memory**: 128 MB–10,240 MB (Central Processing Unit [CPU] scales proportionally; ~1 virtual CPU [vCPU] at 1,769 MB, max 6 virtual CPUs [vCPUs]). **Timeout**: 1 s–**15 min** (default 3 s). **Ephemeral storage** `/tmp` 512 MB–10 GB.
- **Environment variables**: up to **4 KB total**, encrypted at rest with AWS managed key (or your Customer Master Key (now 'customer managed key') [CMK]; **encryption helpers** for client-side encryption in console); reserved names like `AWS_REGION`, `AWS_LAMBDA_FUNCTION_NAME`, `_HANDLER`. For secrets use **Secrets Manager/Parameter Store** with the **AWS Parameters and Secrets Lambda Extension** (local caching HTTP endpoint).
- **Concurrency**:
  - **Account limit** default 1,000 concurrent executions per Region (raise via quota); scaling rate up to **1,000 additional concurrent executions every 10 seconds per function**.
  - **Reserved concurrency**: guarantees and **caps** a function (also used to throttle to protect a DB; 0 = disable function).
  - **Provisioned concurrency**: pre-initialized [environments](00-developer-basics.md#environments-and-stages) (no cold starts), on a version/alias; can auto scale (Application Auto Scaling); billed while configured.
  - **SnapStart** (Java, Python, .NET): snapshot of the initialized environment restored on scale-out; published versions only; use runtime hooks to refresh uniqueness/credentials/connections after restore.
- **Layers**: zip archives of libraries/custom runtimes/data extracted to **`/opt`**; up to **5 layers**; total unzipped function + layers ≤ **250 MB**; versioned and shareable (layer permissions). Not for [container image](00-developer-basics.md#containers-and-images) functions (build [dependencies](00-developer-basics.md#dependency-management-and-packaging) into the image).
- **Extensions**: run alongside the function in the execution environment (**internal**: in-process via language wrappers/env vars; **external**: separate process in `/opt/extensions` with its own lifecycle via the **Extensions API**, e.g., monitoring agents, secrets caching, log shipping via **Telemetry API**). They share memory/CPU and can extend duration (shutdown phase).
- **Versions and aliases**: **publish version** = immutable snapshot (code + config) with its own ARN (`:3`); `$LATEST` is mutable; **alias** (`prod`) points to a version and can **split traffic** between two versions with weights (canary); event sources and API Gateway should point to aliases.
- **Packaging** (Domain 3): zip (≤ 50 MB direct upload, ≤ 250 MB unzipped via S3) or **container image** (≤ 10 GB, from Elastic Container Registry [ECR], must implement the Lambda Runtime API, e.g., AWS base images or runtime interface client).
- **Other**: **function URLs** (dedicated Hypertext Transfer Protocol Secure [HTTPS] endpoint; auth `AWS_IAM` or `NONE`; CORS; response streaming), **response streaming** (Node.js managed runtime and custom runtimes; sends the response progressively, up to 20 MB by default), **recursive loop detection**, **logging config** (JSON vs text format, application/system log levels, custom log group), **code signing** (only allow signed code from approved signing profiles).
- **Exam triggers:** "Java cold starts too slow" → **SnapStart** (or provisioned concurrency). "Predictable low latency at peak" → **provisioned concurrency** on alias. "Function overwhelms downstream DB" → **reserved concurrency**. "Share common library across functions" → **layer**. "Function times out at 3 s" → raise **timeout**. "CPU-bound function slow" → raise **memory**.

### Lambda access to private VPC resources (skill 1.2.1) `[also: SAA]`

- **Configure** the function with **private subnets** (in ≥ 2 Availability Zones [AZs]) and **security groups**; Lambda creates **Hyperplane Elastic Network Interfaces (ENIs)** shared per subnet/SG combination (created at configuration time, not per invocation). Execution role needs `AWSLambdaVPCAccessExecutionRole` (`ec2:CreateNetworkInterface`, `DescribeNetworkInterfaces`, `DeleteNetworkInterface`…).
- **Reach**: RDS/Aurora (use **RDS Proxy** for [connection pooling](00-developer-basics.md#concurrency-thread-pools-and-connection-pools)), ElastiCache, private Application Load Balancers (ALBs), EC2, Elastic File System [EFS] (mount **EFS access point** at `/mnt/...`).
- **Internet/AWS services from a VPC function**: no public Internet Protocol (IP) ever → needs **Network Address Translation (NAT) gateway** (in a public subnet) for internet/AWS public endpoints, or **VPC endpoints** (gateway for S3/DynamoDB; interface for SQS, Secrets Manager, KMS, Security Token Service [STS], Bedrock…). Placing the function in a public subnet doesn't give internet access.
- **Security groups**: DB Security Group (SG) must allow inbound from the Lambda SG on the DB port.
- **Troubleshooting**: function times out calling AWS APIs from VPC → missing NAT/endpoint; can't connect to DB → SG rules; Elastic Network Interface (ENI) creation errors → subnet IP exhaustion or missing permissions; Internet Protocol version 6 (IPv6) dual-stack option available.
- **Exam triggers:** "Lambda must query RDS in a private subnet" → **VPC config + SG rule + RDS Proxy**. "VPC Lambda can't reach S3" → **S3 gateway endpoint** (or NAT). "VPC Lambda can't call a third-party API" → **NAT gateway**.

### Invocation models, event lifecycles and errors (skill 1.2.3) `[also: DOP]`

| Invocation type | Sources | Retries | Failure handling |
|---|---|---|---|
| **Synchronous** (`RequestResponse`) | API Gateway, Application Load Balancer (ALB), function Uniform Resource Locator (URL), Cognito triggers, SDK `Invoke`, Step Functions | None by Lambda (caller decides) | Error returned to caller |
| **Asynchronous** (`Event`) | S3, SNS, EventBridge, CloudWatch Logs, CodeCommit, Simple Email Service (SES), IoT, SDK with `InvocationType=Event` | **2 retries** (configurable 0–2) with backoff; event kept up to **6 hours** (configurable `MaximumEventAgeInSeconds` 60 s–6 h) | **Destinations** (on-success / **on-failure**: SQS, SNS, Lambda, EventBridge, S3 for on-failure) or **DLQ** (SQS/SNS, failures only, event payload only) |
| **Event source mapping** (Lambda polls) | **SQS**, **Kinesis**, **DynamoDB Streams**, MSK/Kafka, Amazon MQ, DocumentDB | Streams: retry the batch until success/expiry (**blocks the shard**); SQS: message returns after visibility timeout | Streams: `BisectBatchOnFunctionError`, `MaximumRetryAttempts`, `MaximumRecordAgeInSeconds`, **on-failure destination** (SQS/SNS/S3); SQS: **queue's DLQ** (redrive policy), **partial batch response** |

- **Destinations vs DLQ**: destinations include **request and response/error context** and support success too; DLQ only failed events. Prefer destinations.
- **Partial batch responses**: enable `ReportBatchItemFailures` and return `{"batchItemFailures": [{"itemIdentifier": "<messageId or sequenceNumber>"}]}` so only failed messages are retried (SQS, Kinesis, DynamoDB Streams).
- **SQS ESM settings**: batch size (up to 10,000 standard; 10 FIFO), **batching window** (up to 5 min), **maximum concurrency** for the ESM (2–1,000) to cap Lambda scaling; scaling up to 1,250 concurrent; FIFO processes groups in order.
- **Filtering**: **event filtering** on Event Source Mappings [ESMs] (SQS, Kinesis, DynamoDB, Managed Streaming for Apache Kafka [MSK]) with patterns so non-matching records don't invoke (and are deleted/skipped).
- **Error types**: function errors (unhandled exceptions → `FunctionError` header, error object), invocation errors (throttling `TooManyRequestsException` 429, permissions, payload too large: sync **6 MB**, async **1 MB**; older material says 256 KB for async), timeouts (`Task timed out after X seconds`).
- **Idempotency** is required for async and ESM processing (at-least-once).
- **Exam triggers:** "keep failed S3-triggered events for analysis with error details" → **on-failure destination**. "One bad Kinesis record blocks the shard" → **bisect + max retries + on-failure destination**. "Only failed SQS messages should be retried" → **partial batch response**. "Lambda scales too much and overwhelms DB from SQS" → **ESM maximum concurrency** (or reserved concurrency).

### Integrating Lambda with other services (skill 1.2.4) `[also: DOP]`

- **API Gateway** (proxy integration event `httpMethod`, `path`, `headers`, `queryStringParameters`, `pathParameters`, `body`, `requestContext` with authorizer claims) → return proxy response; **Lambda authorizers** (token or request type; return an IAM policy + context; cached by TTL).
- **ALB** target (event has `requestContext.elb`; multi-value headers option; response needs `statusCode`, `statusDescription`, headers, body).
- **DynamoDB**: SDK reads/writes from the function; **DynamoDB Streams ESM** for triggers (batch of `Records` with `eventName` `INSERT`/`MODIFY`/`REMOVE` and `dynamodb.NewImage`/`OldImage`).
- **S3**: event notification triggers; read/write objects with the SDK; S3 Object Lambda (transform data on GET).
- **SQS/Kinesis/MSK** ESMs; **SNS** subscriptions; **EventBridge** rules/Scheduler; **Step Functions** tasks; **Cognito** user pool triggers; **CloudWatch Logs** subscriptions; **CloudFront** Lambda@Edge.
- **Permissions**: **execution role** (what the function may call) vs **resource-based policy** (who may invoke the function: `lambda:InvokeFunction` for `s3.amazonaws.com`, `sns.amazonaws.com`, `events.amazonaws.com`, `apigateway.amazonaws.com`, other accounts). ESMs use the execution role to **read** from the source (e.g., `sqs:ReceiveMessage`, `kinesis:GetRecords`).
- **Invoking Lambda from code**: `Invoke` with `InvocationType` `RequestResponse` (sync), `Event` (async), `DryRun` (permission check); qualifier for version/alias.
- **Exam triggers:** "S3 can't trigger the function" → missing **resource-based policy**. "Function can't read from the SQS queue" → **execution role** permissions. "Return 401/403 based on a custom token" → **Lambda authorizer**.

---

## Task 1.3: Use data stores in application development

### Data lifecycles (skill 1.3.1) `[also: SAA]`

- **S3 Lifecycle**: transition (Standard → Standard-Infrequent Access [Standard-IA] ≥ 30 days → Glacier Instant/Flexible → Deep Archive), **expiration** of current versions, **noncurrent version** expiration/transition, delete expired **delete markers**, **abort incomplete multipart uploads**; filters by prefix, tags, object size. Minimum storage durations (Infrequent Access [IA] 30, Glacier Instant Retrieval [Glacier IR]/Flexible 90, Deep Archive 180 days) and minimum object sizes affect cost.
- **DynamoDB TTL**: set a Number attribute with **epoch seconds** expiry; items deleted in the background (typically within a few days of expiry, not instantly) at **no Write Capacity Unit (WCU) cost**; expired items may still appear in reads until deleted → filter by expiry in queries; TTL deletes appear in **DynamoDB Streams** (identifiable by `userIdentity` `dynamodb.amazonaws.com`) for archiving to S3.
- **ElastiCache TTL**: `EXPIRE`/`SET ... EX` per key; eviction policies (`allkeys-lru`, `volatile-lru`…) when memory is full.
- **Retention periods**: **CloudWatch Logs** retention per log group (never expire by default), **Kinesis** retention 24 h–365 days, **SQS** 1 min–14 days, **DynamoDB Streams** 24 h, **RDS** backup retention 0–35 days, **S3 Object Lock** retention, **ECR lifecycle policies** for images, Step Functions history 90 days.
- **Exam triggers:** "delete session records 24 hours after creation automatically, no cost" → **DynamoDB TTL**. "Move logs to cheaper storage after 30 days, delete after 1 year" → **S3 Lifecycle**. "Log costs growing forever" → **log group retention**.

### Choosing data stores by access pattern (skill 1.3.2) `[also: SAA]`

| Access pattern | Store |
|---|---|
| Key-value lookups, massive scale, single-digit ms, serverless, flexible schema | **DynamoDB** |
| Relational data, joins, ad-hoc SQL, transactions across tables | **RDS / Aurora** |
| Sub-millisecond reads, sessions, leaderboards, counters, pub/sub, rate limiting | **ElastiCache (Valkey/Redis OSS)** |
| Simple distributed cache, multi-threaded, no persistence | **ElastiCache Memcached** |
| Files/objects, media, static assets, data lake, backups, large payloads | **S3** |
| Archive | **S3 Glacier** classes |
| Shared file system for containers/Lambda | **EFS** |
| Block storage for one EC2 instance | **Elastic Block Store (EBS)** |
| Full-text search, log analytics | **OpenSearch** |
| SQL over files in S3 | **Athena** |

- Common combos: DynamoDB item with **S3 pointer** for large objects (> 400 KB); RDS + ElastiCache; S3 + DynamoDB metadata index.

### Caching solutions by data store (skill 1.3.3) `[also: SAA]`

| Data store | Caching choice |
|---|---|
| **DynamoDB** | **DynamoDB Accelerator [DAX]** (API-compatible, microsecond, [write-through](00-developer-basics.md#caching) item cache + query cache; [eventually consistent](00-developer-basics.md#read-consistency) reads only; in VPC) or ElastiCache for computed results |
| **RDS/Aurora** | **ElastiCache** (lazy loading/write-through in app code); Aurora replicas for read scaling; RDS Proxy is not a cache |
| **S3 / static content** | **CloudFront** (edge cache) |
| **API responses** | **API Gateway caching** (REST) or CloudFront |
| **Computed/session data** | ElastiCache (Valkey/Redis OSS) |
| **Lambda** | In-memory variables in init scope (per environment), `/tmp`, Parameters and Secrets extension cache |

- **Strategy trade-offs**: lazy loading (only requested data cached; stale until TTL; cache miss penalty), write-through (always fresh; writes cost more; unused data cached), TTL for all; cache invalidation on writes.
- **Exam triggers:** "DynamoDB reads need microseconds with no code changes" → **DAX**. "Cache results of expensive SQL queries" → **ElastiCache lazy loading**. "Leaderboard" → **Redis sorted sets**.

### Amazon DynamoDB for developers (skill 1.3.4) `[also: SAA]`

- **Keys**: **partition key** (simple primary key) or **partition key + sort key** (composite). Choose **high-cardinality**, evenly accessed partition keys (userId, orderId); avoid low-cardinality keys (status, date) → hot partitions. **Write sharding**: add a random/calculated suffix (`2026-11-19#7`) for hot keys and query all suffixes. Partition limits: **3,000 Read Capacity Unit (RCU) and 1,000 WCU per partition**.
- **Items and attributes**: item max **400 KB**; schemaless attributes (only key attributes required); types: scalar (S, N, B, BOOL, NULL), document (M, L), sets (SS, Name Server [NS], BS).
- **Read consistency**: **eventually consistent** (default; 0.5 RCU per 4 KB) vs **strongly consistent** (`ConsistentRead=true`; 1 RCU per 4 KB; not on Global Secondary Indexes [GSIs] or cross-Region global table reads). **Transactional** reads/writes cost 2×.
- **Capacity math**: RCU = ceil(item size / 4 KB) per strongly consistent read/s (half for eventual); WCU = ceil(item size / 1 KB) per write/s. Example: 50 writes/s of 2.5 KB items → 3 WCU each → **150 WCU**. 80 eventually consistent reads/s of 6 KB → 2 RCU each × 0.5 → **80 RCU**.
- **Operations**:
  - `GetItem` (by full key), `PutItem` (replace), `UpdateItem` (**update expressions**: `SET`, `REMOVE`, `ADD`, `DELETE`; **atomic counters** `SET n = n + :inc`), `DeleteItem`.
  - **`Query`**: by **partition key** (required, equality) + optional **sort key condition** (`=`, `<`, `BETWEEN`, `begins_with`); efficient; returns up to **1 MB per call** (paginate with `LastEvaluatedKey`); `ScanIndexForward=false` for descending; `Limit`; **`FilterExpression` is applied after reading** (still consumes RCU for all read items).
  - **`Scan`**: reads the **whole table** (expensive, slow); **parallel scan** (`Segment`/`TotalSegments`) to speed up; use sparingly (exports for analytics).
  - **`ProjectionExpression`** (return only some attributes), **`ExpressionAttributeNames`** (`#n` for reserved words like `name`, `status`), **`ExpressionAttributeValues`** (`:v`).
  - **Conditional writes**: `ConditionExpression` (`attribute_not_exists(pk)` to prevent overwrite; version number check for **optimistic locking**) → `ConditionalCheckFailedException`.
  - **Batch**: `BatchGetItem` (up to **100 items / 16 MB**), `BatchWriteItem` (up to **25 put/delete**, no update, no conditions); handle **`UnprocessedItems`/`UnprocessedKeys`** with backoff retry.
  - **Transactions**: `TransactWriteItems`/`TransactGetItems` (up to **100 actions**, all-or-nothing, across tables, with conditions; idempotency with `ClientRequestToken`).
  - **PartiQL** (SQL-compatible `SELECT/INSERT/UPDATE/DELETE`; a `SELECT` without a key condition becomes a scan).
- **Indexes**:
  - **GSI** (global secondary index): different partition (and optional sort) key; created any time; **own capacity** (if it throttles, base-table writes throttle); **eventually consistent only**; **projections** (`KEYS_ONLY`, `INCLUDE`, `ALL`); up to 20 per table. **Sparse index**: only items having the index key attribute appear (efficient filtering, e.g., `isOpen`).
  - **LSI** (local secondary index): same partition key, different sort key; **must be created with the table**; shares table capacity; strongly consistent option; **10 GB per partition key value** limit; up to 5.
- **Single-table design**: multiple entity types in one table with generic keys (`PK=CUSTOMER#123`, `SK=ORDER#2026-11-19`), GSIs overloaded for alternate access patterns; design from access patterns first.
- **Other features**: **on-demand vs provisioned** (+ auto scaling), **DynamoDB Streams**, **TTL**, **Point-In-Time Recovery (PITR)**, **global tables**, **DAX**, **export to S3** (no RCU use), **Standard-IA table class**, **deletion protection**, **DynamoDB Local** for development/testing, fine-grained access control with `dynamodb:LeadingKeys` (users access only their own partition).
- **Exam triggers:** "query orders by customer and date range" → **PK = customerId, SK = orderDate, `Query` with BETWEEN**. "Look up by email instead of userId" → **GSI on email**. "Need strongly consistent query on alternate sort key" → **LSI** (only at table creation). "Prevent overwriting an existing item" → **`attribute_not_exists` condition**. "Two users updating the same item" → **optimistic locking with version attribute**. "Scan is slow and expensive" → redesign for **Query**/GSI. "BatchWriteItem returns unprocessed items" → **retry with backoff**.

---

## Gotchas and exam traps

- Lambda **proxy** responses must include `statusCode` and a **string** `body`, or API Gateway returns **502**.
- With proxy integrations, **CORS headers come from your Lambda**.
- API Gateway timeout **29 s** default, payload **10 MB**; Lambda sync payload **6 MB**, async **1 MB** (formerly 256 KB).
- **Destinations** > DLQ (include context, support success).
- Stream ESMs retry until success → **bisect/max retries/max age/on-failure destination**; SQS → **partial batch response** + queue DLQ.
- SQS visibility timeout ≥ 6× Lambda timeout for ESMs.
- VPC Lambda has **no internet** without NAT; use endpoints for AWS services.
- **Reserved concurrency caps**; **provisioned concurrency** removes cold starts; **SnapStart** for Java/Python/.NET.
- Layers: max 5, 250 MB total unzipped, extracted to `/opt`; env vars 4 KB total.
- **FilterExpression** doesn't reduce RCU; **Scan** reads everything.
- **GSI = eventually consistent**; **LSI only at creation**.
- RCU = 4 KB, WCU = 1 KB, transactional ×2, eventual ×0.5.
- **TTL** needs epoch seconds Number; deletion isn't immediate.
- SDK credential chain: env vars → files/SSO → container → instance metadata; never hard-code keys.
- `PutRecords`/`PutEvents`/`BatchWriteItem` can **partially fail**: check per-entry results.
- Bedrock `ThrottlingException` → backoff, cross-Region inference profile, quota increase.

## Quick-recall summary

- **Decouple**: SQS (visibility, long polling, DLQ, FIFO groups), SNS (filters, raw delivery, fan-out), EventBridge (patterns, PutEvents, Pipes, Scheduler, replay), Step Functions (Retry/Catch, .sync, task tokens, Standard vs Express).
- **Resilience**: SDK retries with backoff/jitter, timeouts, idempotency, graceful degradation, Dead-Letter Queues (DLQs)/destinations.
- **APIs**: API Gateway REST/HTTP/WebSocket, proxy vs non-proxy, mapping templates (VTL), validators + models, CORS, throttling, caching.
- **SDK**: credential chain, paginators, waiters, error handling, CLI/CloudShell.
- **Streams**: KDS (shards, partition keys, KCL, Enhanced Fan-Out [EFO], ESM settings), Firehose (transform, convert), DynamoDB Streams.
- **AI**: Kiro (specs, steering, hooks), Bedrock (Converse, streaming, inference profiles, KBs, Agents, batch), AgentCore (Runtime, Gateway, Identity, Memory, tools, Observability), Amazon Quick.
- **Lambda**: config (memory, timeout, env vars, concurrency types, SnapStart, layers, extensions, versions/aliases), VPC access, invocation types and error handling, integrations and permissions.
- **Data**: lifecycle (S3 Lifecycle, TTL, retention), store choice, caching per store, DynamoDB keys/consistency/capacity math/Query vs Scan/conditions/transactions/indexes.
