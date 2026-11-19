# Domain 4: Troubleshooting and Optimization (22%)

> 🔰 New to a term? Concepts link to plain-English explanations in [the basics page](00-developer-basics.md) on first mention, and every abbreviation is spelled out on first use and listed in the [glossary](00-abbreviations-glossary.md).

> **About 11 of the 50 scored questions.**
>
> Tests whether you can find the root cause of defects using logs, metrics and [traces](00-developer-basics.md#logs-metrics-and-traces), build good observability into code (structured logs, custom metrics, alerts, tracing, [health checks](00-developer-basics.md#health-checks-liveness-and-readiness)), and make applications faster and cheaper (concurrency, memory/compute, messaging, caching, bottleneck detection), including with Artificial Intelligence (AI) tools.
>
> Tags like `[also: DOP]` mark material you met in another exam. It is still covered in full here.

## Table of contents

- [Exam tasks and skills](#exam-tasks-and-skills)
- [Task 4.1: Assist in root cause analysis](#task-41-assist-in-root-cause-analysis)
- [Task 4.2: Logging, monitoring and observability](#task-42-logging-monitoring-and-observability)
- [Task 4.3: Optimize applications](#task-43-optimize-applications)
- [Gotchas and exam traps](#gotchas-and-exam-traps)
- [Quick-recall summary](#quick-recall-summary)

## Exam tasks and skills

> Summarized from the official DVA-C03 exam guide. Exact wording: https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html

### Task 4.1: Assist in a root cause analysis
- **4.1.1** Debug code and service integration issues.
- **4.1.2** Interpret application metrics, logs and traces.
- **4.1.3** Query logs to find relevant data.
- **4.1.4** Troubleshoot deployment failures using output logs.
- **4.1.5** Use Amazon Web Services (AWS) AI tools to analyze errors and suggest fixes.

### Task 4.2: Apply logging, monitoring and observability best practices
- **4.2.1** Implement effective logging (structured logging of events and user actions).
- **4.2.2** Emit custom metrics from code.
- **4.2.3** Implement notification alerts (quota limits, deployment completions).
- **4.2.4** Implement tracing with AWS services and tools.
- **4.2.5** Monitor workload health (application health checks, liveness and readiness probes).

### Task 4.3: Optimize applications using AWS services and features
- **4.3.1** Apply concurrency concepts (Lambda provisioned concurrency, Elastic Compute Cloud [EC2] Auto Scaling, thread pools, [connection pooling](00-developer-basics.md#concurrency-thread-pools-and-connection-pools)).
- **4.3.2** Optimize memory and compute.
- **4.3.3** Optimize messaging (Simple Queue Service [SQS] queues, Simple Notification Service [SNS] fan-out).
- **4.3.4** Implement caching (ElastiCache, DynamoDB Accelerator [DAX], CloudFront).
- **4.3.5** Detect performance bottlenecks (CloudWatch).
- **4.3.6** Use AWS AI tools to find optimization opportunities.

---

## Task 4.1: Assist in root cause analysis

### Debugging code and service integrations (skill 4.1.1) `[also: DOP]`

**Common integration failures and where they show up:**

| Symptom | Likely cause | Where to look / fix |
|---|---|---|
| [Application Programming Interface [API]](00-developer-basics.md#api) Gateway **502** | Lambda proxy response malformed (no `statusCode`, `body` not a string), function error/exception, unhandled promise | Lambda logs, API Gateway execution logs; return proper format |
| API Gateway **504** | Integration exceeded **29 s** timeout | Optimize, go async (SQS/Step Functions), stream, or raise timeout ([Representational State Transfer [REST]](00-developer-basics.md#rest-and-http-methods) regional) |
| API Gateway **403** | Missing auth/wrong authorizer, resource policy, Web Application Firewall (WAF) block, calling wrong stage/path (`Missing Authentication Token` for non-existent route), private API without endpoint | Access logs, authorizer config |
| API Gateway **429** | Throttling (account/stage/usage plan) | Raise limits, [backoff](00-developer-basics.md#retries-exponential-backoff-and-jitter), caching |
| API Gateway **500** with Lambda authorizer | Authorizer error or bad policy document | Authorizer logs |
| Lambda `Task timed out after N seconds` | Timeout too low, slow downstream call, Virtual Private Cloud (VPC) function without NAT/endpoint hanging on AWS API calls | Raise timeout, set [Software Development Kit [SDK]](00-developer-basics.md#sdk-and-cli) timeouts, add VPC endpoint/NAT |
| Lambda `Runtime exited` / out of memory (`Max Memory Used` = configured) | Memory too low, leak | Raise memory, fix code |
| Lambda `TooManyRequestsException` (429) | Concurrency limit (account or reserved) | Quota increase, reserved/provisioned concurrency, smooth with SQS |
| Lambda `AccessDeniedException` calling a service | **Execution role** missing permission or Key Management Service (KMS) key policy | Identity and Access Management (IAM), CloudTrail `errorCode` |
| Event source doesn't trigger function | Missing **resource-based policy** (S3/SNS/EventBridge), disabled Event Source Mapping (ESM), filter criteria mismatch, ESM role can't read source | Lambda permissions, ESM state |
| SQS messages reprocessed / in Dead-Letter Queue (DLQ) | Visibility timeout < processing time, function errors, batch partially failed without partial batch response | Visibility ≥ 6× timeout, `ReportBatchItemFailures`, [idempotency](00-developer-basics.md#idempotency) |
| Kinesis/DynamoDB stream stuck | Poison record retried forever | Bisect, max retries/age, on-failure destination; `IteratorAge` |
| DynamoDB `ProvisionedThroughputExceededException` | Hot partition, low capacity, Global Secondary Index (GSI) throttling | On-demand/auto scaling, better keys, backoff |
| DynamoDB `ValidationException` | Reserved words without `ExpressionAttributeNames`, wrong key types, missing key attributes, item > 400 KB | Fix expressions/types |
| `ConditionalCheckFailedException` | Condition false (expected in optimistic locking) | Handle, re-read, retry |
| Simple Storage Service (S3) `AccessDenied` on GetObject | IAM/bucket policy, KMS key permission, object owned by another account, presigned Uniform Resource Locator (URL) expired/creds expired | Policies, Object Ownership |
| S3 `SignatureDoesNotMatch` | Wrong secret, clock skew, modified URL/headers | Sync clock, re-sign |
| Cross-Origin Resource Sharing (CORS) error in browser | Missing `Access-Control-Allow-*` headers (Lambda proxy must add them), OPTIONS not handled, S3 bucket CORS config | Configure CORS |
| Step Functions `States.TaskFailed` / `States.Timeout` | Task error, missing IAM permission for the state machine role, timeout | Execution history (input/output per state), Retry/Catch |
| Cognito `NotAuthorizedException` | Wrong password/expired refresh token/app client settings | Token handling |
| Bedrock `ThrottlingException` / `ValidationException` / `AccessDeniedException` | Token/request quotas; wrong request body or model ID; model access not granted or IAM missing | Backoff, inference profiles; check model ID/region; enable access |

- **Debugging techniques**: reproduce with the same **test event**; run locally (`sam local invoke`, debugger attach with `-d` port, Integrated Development Environment [IDE] toolkits); add structured logs with request IDs; check **CloudTrail** for denied API calls; use **X-Ray** service map/trace timelines; test IAM with **policy simulator**; API Gateway **console test**; Step Functions **execution event history** and **redrive** (restart failed executions from the failed state); EventBridge **sandbox** for patterns; SQS DLQ inspection.

### Interpreting metrics, logs and traces (skill 4.1.2) `[also: DOP]`

- **Lambda `REPORT` log line**: `Duration`, `Billed Duration`, `Memory Size`, **`Max Memory Used`** (close to Memory Size → raise memory), **`Init Duration`** (present only on **cold starts**), `XRAY TraceId`. `START`/`END` lines with **RequestId**.
- **Lambda metrics**: `Invocations`, `Errors` (function errors), `Throttles`, `Duration` ([p99](00-developer-basics.md#percentiles)), `ConcurrentExecutions`, `UnreservedConcurrentExecutions`, `ProvisionedConcurrencySpilloverInvocations` (provisioned capacity exceeded → cold starts), `IteratorAge` (stream lag), `AsyncEventAge`, `AsyncEventsDropped`, `DeadLetterErrors`, `DestinationDeliveryFailures`, `RecursiveInvocationsDropped`.
- **API Gateway metrics**: `Count`, `4XXError`, `5XXError`, **`Latency`** (end-to-end) vs **`IntegrationLatency`** (backend only: if Latency ≫ IntegrationLatency, overhead is in API Gateway: authorizers, mapping, cold start of authorizer), `CacheHitCount`/`CacheMissCount`.
- **SQS**: `ApproximateAgeOfOldestMessage` (consumers falling behind), `ApproximateNumberOfMessagesVisible`/`NotVisible` (in flight), `NumberOfEmptyReceives` (use long polling).
- **DynamoDB**: `ConsumedRead/WriteCapacityUnits`, `ThrottledRequests`, `ReadThrottleEvents`/`WriteThrottleEvents`, `SuccessfulRequestLatency`, `SystemErrors`, `UserErrors`; Contributor Insights for hot keys.
- **Application Load Balancer (ALB)**: `TargetResponseTime`, `HTTPCode_Target_5XX_Count` (app errors) vs `HTTPCode_ELB_5XX_Count` (LB/no healthy targets), `UnHealthyHostCount`.
- **X-Ray traces**: **segments** (one per service), **subsegments** (downstream calls: DynamoDB, Hypertext Transfer Protocol [HTTP], Structured Query Language [SQL]), **annotations** (indexed key-values for filtering traces, e.g., `customer_tier`), **metadata** (non-indexed details), **errors** (4xx), **faults** (5xx), **throttles** (429); the **service map** shows latency and error rates per node; **trace timeline** shows where time is spent; **filter expressions** (`service("api") AND responsetime > 2`, `annotation.customer_tier = "gold"`); **insights** detect anomalies.
- **Correlation**: same request ID/trace ID in logs and traces; CloudWatch **ServiceLens**/**Application Signals** combine metrics, logs and traces per service with **Service Level Objectives (SLOs)**.
- **Exam triggers:** "Lambda sometimes slow on first call" → `Init Duration` = **cold start** → provisioned concurrency/SnapStart/smaller package. "Which downstream call is slow?" → **X-Ray subsegments/timeline**. "API slow but Lambda fast" → compare `Latency` vs `IntegrationLatency`. "Filter traces by customer ID" → **annotation**.

### Querying logs (skill 4.1.3) `[also: DOP]`

- **CloudWatch Logs Insights** query language:
  ```
  fields @timestamp, @message, @requestId
  | filter @message like /ERROR/
  | sort @timestamp desc
  | limit 50
  ```
  - Commands: `fields`, `filter` (`like`, `=`, regex, `in`), `stats` (`count()`, `avg()`, `max()`, `pct(@duration, 99)`, `by bin(5m)`), `sort`, `limit`, `parse` (extract fields from text with glob or regex), `dedup`, `display`, `unmask` (with permission).
  - JavaScript Object Notation (JSON) log fields are **auto-discovered** (`filter level = "ERROR" and orderId = "123"`).
  - Lambda: `filter @type = "REPORT" | stats max(@maxMemoryUsed), avg(@duration), count(@initDuration) as coldStarts by bin(1h)`.
  - Query multiple log groups (or by prefix), save queries, add results to dashboards; natural-language query generation; also **OpenSearch Piped Processing Language (PPL)/SQL** query languages supported.
- **Filter patterns** (`aws logs filter-log-events --filter-pattern`): terms (`ERROR`), JSON (`{ $.level = "ERROR" && $.latency > 500 }`), space-delimited; also used by **metric filters** and **subscription filters**.
- **Live Tail** to watch logs in real time; **`sam logs --tail`**; **CloudTrail Lake** SQL / **Athena** for logs in S3 (ALB access logs, CloudTrail, Flow Logs).
- **Exam triggers:** "find all requests for order 123 across services" → Logs Insights with **structured JSON field filter** (multiple log groups). "Count errors per 5 minutes" → `stats count() by bin(5m)`. "p99 duration of Lambda" → `pct(@duration, 99)`.

### Troubleshooting deployment failures from output logs (skill 4.1.4) `[also: DOP]`

| Tool | Where the logs are | Typical failures |
|---|---|---|
| **CloudFormation / Serverless Application Model (SAM) / Cloud Development Kit (CDK)** | **Stack events** (first `CREATE_FAILED`/`UPDATE_FAILED` reason), `sam deploy`/`cdk deploy` output, CloudTrail | Missing capabilities (`CAPABILITY_IAM/NAMED_IAM/AUTO_EXPAND`), IAM permissions/`iam:PassRole`, resource name collisions, invalid property values, limits, custom resource timeouts, `UPDATE_ROLLBACK_FAILED` (→ continue rollback), CDK not **bootstrapped**, S3 artifact bucket access |
| **CodeBuild** | Build logs in **CloudWatch Logs**/S3, phase details in console (`DOWNLOAD_SOURCE`, `INSTALL`, `PRE_BUILD`, `BUILD`, `POST_BUILD`, `UPLOAD_ARTIFACTS`) | Command non-zero exit, missing permissions (Elastic Container Registry [ECR] push, S3, Secrets Manager), Docker without **privileged mode**, VPC build without Network Address Translation (NAT), wrong `runtime-versions`, buildspec YAML Ain't Markup Language (YAML) errors, timeouts |
| **CodeDeploy** | Deployment events per lifecycle hook in console; instance logs `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log`; Lambda hook function logs | Agent not running, instance profile can't read revision, hook script error/timeout, health checks failing, `ApplicationStop` failure from old revision, alarms triggering rollback |
| **CodePipeline** | Action execution details and links to provider logs | Source connection permissions, artifact bucket/KMS access, cross-account role issues |
| **Elastic Container Service (ECS)** | Service **events** tab (e.g., "unable to place a task", "task failed Elastic Load Balancing [ELB] health checks"), stopped task **stopped reason** (`CannotPullContainerError`, `ResourceInitializationError` for secrets/logs, `Essential container exited`, OutOfMemory), container logs | ECR pull permissions/network (execution role, endpoints/NAT), insufficient CPU/memory, health check failures, [circuit breaker](00-developer-basics.md#timeouts-and-circuit-breakers) rollback |
| **Elastic Beanstalk** | Environment **events**, `eb logs` (instance logs: `/var/log/eb-engine.log`, `web.stdout.log`), health dashboard | `.ebextensions` command failures, dependency install failures, health check path, deployment timeouts |
| **Lambda deploy** | Command Line Interface (CLI)/API errors | Package too large (`RequestEntityTooLargeException` / unzipped > 250 MB), wrong handler (`Runtime.HandlerNotFound` at invoke), image not in ECR/arch mismatch, `CodeStorageExceededException` (75 GB per Region code storage → delete old versions) |
| **Amplify** | Build logs per phase in Amplify console | `amplify.yml` errors, env vars missing |

- **Exam triggers:** "CloudFormation stack rolled back; find why" → **stack events (first failure)**. "ECS tasks keep stopping with `CannotPullContainerError`" → **execution role/ECR endpoints/NAT**. "CodeBuild fails at docker build" → **privileged mode**. "Lambda deploy fails: code storage limit" → delete old **versions**.

### AI tools for error analysis (skill 4.1.5)

- **Kiro**: paste stack traces/logs or let the agent read files and terminal output; it explains errors, traces them to code, proposes fixes as diffs or **bugfix specs**, runs tests to verify; **[Model Context Protocol [MCP]](00-developer-basics.md#mcp)** servers (e.g., AWS documentation, CloudWatch) give it context; **hooks** can trigger analysis on test failures.
- **AWS DevOps Agent**: investigates **production incidents** automatically from alarms/tickets, correlates metrics, logs, traces and recent deployments to find **root causes**, proposes mitigations (including rollback), and can hand off code fix specs to coding agents.
- **CloudWatch [AI-assisted](00-developer-basics.md#ai-assisted-development) investigations and features**: AI-powered **investigations** of alarms/anomalies (hypotheses with supporting metrics/logs/traces), **natural-language query generation** for Logs Insights/Metrics Insights, **log anomaly detection** and **pattern analysis**, Application Signals Service Level Objective (SLO) breach analysis.
- **Amazon Bedrock** in your own tooling: summarize error logs, classify exceptions, suggest fixes (e.g., a Lambda that analyzes CodeBuild failure logs and posts suggestions to the Pull Request [PR]).
- **Exam triggers:** "developers need AI help understanding an exception and generating a fix with tests" → **Kiro**. "Speed up root cause analysis of production incidents" → **DevOps Agent / CloudWatch investigations**. "Write a Logs Insights query from a plain-English question" → **query generation**.

---

## Task 4.2: Logging, monitoring and observability

### Effective logging (skill 4.2.1) `[also: DOP]`

- **Structured logging**: emit **JSON** with consistent fields: `timestamp`, `level`, `message`, `service`, **`request_id`/`correlation_id`**, `trace_id`, `user_id` (not [Personally Identifiable Information [PII]](00-developer-basics.md#pii-and-phi-masking-and-redaction)), `action`, `duration_ms`, `error` details. Benefits: Logs Insights auto-discovers fields, metric filters on JSON, cross-service correlation.
- **Lambda**: logs to CloudWatch Logs automatically (`/aws/lambda/<function>`); **JSON log format** setting makes the runtime emit structured logs (including request ID), **application log level** and **system log level** controls (filter DEBUG in prod without code changes), custom log group; **Powertools for AWS Lambda** (Python, TypeScript, Java, .NET): `Logger` (structured logs, [correlation IDs](00-developer-basics.md#correlation-id), sampling debug logs, log event), `Metrics` (EMF), `Tracer` (X-Ray), plus idempotency, batch processing, parameters, data masking utilities.
- **Containers**: log to **stdout/stderr**; ECS `awslogs` driver or FireLens; Elastic Kubernetes Service (EKS) Fluent Bit.
- **Log user actions/audit events** (who did what) separately from debug logs; never log secrets/PII (mask); use **log levels** appropriately; **sampling** high-volume debug logs; set **retention**; consider **CloudWatch Logs Infrequent Access** class for cost.
- **Exam triggers:** "search logs by order ID across services" → **structured JSON with correlation ID**. "Turn on debug logs in prod temporarily without redeploying" → **Lambda log level setting** (or Powertools env var). "Logs contain PII" → mask/remove; data protection policy.

### Emitting custom metrics (skill 4.2.2) `[also: DOP]`

- **`PutMetricData`** API: namespace, metric name, dimensions (up to 30), value/values + counts (statistic sets), unit, timestamp, `StorageResolution` (60 or **1 for high resolution**); batch up to 1,000 metrics per call; adds latency and API cost per call (and can throttle).
- **Embedded Metric Format (EMF)**: write a JSON log line with an `_aws` metadata block (`CloudWatchMetrics` namespace, dimensions, metrics) → CloudWatch extracts metrics **asynchronously** from logs: **no extra API calls or latency**; ideal for Lambda/containers. Libraries: Powertools `Metrics`, `aws-embedded-metrics`.
- **Metric filters** on existing logs (count `ERROR`, extract latency) for metrics without code changes (not retroactive).
- **CloudWatch agent** StatsD/collectd for custom metrics on EC2/containers; **OpenTelemetry** metrics via AWS Distro for OpenTelemetry (ADOT).
- **Design**: business Key Performance Indicators [KPIs] (orders placed, payment failures), latency percentiles, avoid [high-cardinality](00-developer-basics.md#partition-key-and-cardinality) dimensions (each combination is a billable metric).
- **Exam triggers:** "emit business metrics from Lambda without slowing it down" → **EMF**. "Need 1-second granularity" → **high-resolution** metric. "Metric from existing log lines without code changes" → **metric filter**.

### Notification alerts (skill 4.2.3) `[also: DOP]`

- **CloudWatch alarms** on metrics/metric math/anomaly detection → **SNS** (email, SMS, chat via Amazon Q Developer in chat applications, Lambda), with M-of-N datapoints and missing-data handling; **composite alarms** to reduce noise.
- **Quota limit alerts**: **Service Quotas** console → create a **CloudWatch alarm on usage** (AWS/Usage namespace metrics, e.g., Lambda `ConcurrentExecutions` vs quota with `SERVICE_QUOTA()` metric math); Trusted Advisor service limit checks; alarm on `UnreservedConcurrentExecutions`, `Throttles`, API Gateway `429`s, DynamoDB throttles.
- **Deployment completion/failure alerts**: **CodePipeline/CodeBuild/CodeDeploy notification rules** (via AWS CodeStar Notifications) → SNS/Slack, or **EventBridge rules** on `CodePipeline Pipeline Execution State Change`, `CodeBuild Build State Change`, `CodeDeploy Deployment State-change Notification`, `ECS Deployment State Change`, CloudFormation stack status events → SNS/Lambda.
- **Other events**: AWS Health (service events, scheduled changes), AWS Certificate Manager (ACM) [certificate](00-developer-basics.md#certificates-and-ssh-keys) expiry, GuardDuty findings, budget alerts (AWS Budgets → SNS), Bedrock throttling alarms.
- **Exam triggers:** "notify the team when a pipeline fails" → **notification rule** or **EventBridge rule → SNS**. "Alert before hitting the Lambda concurrency limit" → **Service Quotas usage alarm**.

### Tracing (skill 4.2.4) `[also: DOP]`

- **AWS X-Ray**:
  - Enable: **Lambda active tracing** (toggle; execution role needs `xray:PutTraceSegments`/`PutTelemetryRecords` via `AWSXRayDaemonWriteAccess`), **API Gateway** stage tracing, **Step Functions** tracing, App Runner, Elastic Beanstalk option; on **EC2/ECS** run the **X-Ray daemon** (User Datagram Protocol [UDP] port 2000) or **CloudWatch agent/ADOT collector** as a sidecar/daemon.
  - **Instrument code**: X-Ray SDK (patch AWS SDK and HTTP clients to create subsegments automatically; `xray_recorder.capture`, `begin_subsegment`; add **annotations** and **metadata**) or **AWS Distro for OpenTelemetry (ADOT)** / OpenTelemetry Software Development Kits [SDKs] (recommended going forward; send to X-Ray/CloudWatch); Powertools `Tracer`.
  - **Sampling rules**: default 1 request/s + 5% of additional requests (reservoir + rate); custom rules by service/URL/method to control cost.
  - **Trace header** `X-Amzn-Trace-Id` propagates context across services (API Gateway, ALB add it); SQS/SNS/EventBridge propagate for linked traces.
  - Views: **service map**, traces, analytics; **CloudWatch Application Signals** (automatic service-level dashboards, SLOs from OpenTelemetry instrumentation), **Transaction Search** (all spans indexed in CloudWatch Logs).
- **Exam triggers:** "find which service in a chain adds latency" → **X-Ray**. "Lambda traces missing" → active tracing + **execution role X-Ray permissions**. "Traces from ECS [containers](00-developer-basics.md#containers-and-images)" → **X-Ray daemon/ADOT sidecar** + task role permissions. "Group traces by tenant" → **annotation**.

### Monitoring workload health (skill 4.2.5) `[also: DOP]`

- **Application health check endpoints**: `/health` (liveness: process up) and `/ready` (readiness: [dependencies](00-developer-basics.md#dependency-management-and-packaging) reachable, warm-up complete); keep liveness checks shallow (don't fail on a dependency outage, or every instance restarts).
- **Where they're used**:
  - **ELB target group health checks** (path, interval, thresholds, success codes) → route only to healthy targets; Auto Scaling group (ASG) with ELB health type replaces unhealthy instances.
  - **ECS container health checks** (`healthCheck` command in the task definition with interval/retries/`startPeriod`) and service health check grace period; unhealthy essential containers stop tasks.
  - **Kubernetes probes** on EKS: **liveness** (restart container), **readiness** (remove from Service endpoints), **startup** probes (protect slow starters) via HTTP/Transmission Control Protocol (TCP)/exec.
  - **Route 53 health checks** for Domain Name System (DNS) failover; **API Gateway** health via canaries.
  - **CloudWatch Synthetics canaries** (scripted checks of Application Programming Interfaces [APIs] and User Interface [UI] flows on a schedule; alarms on `SuccessPercent`), **CloudWatch RUM** (real users), **Application Signals SLOs**, **Elastic Beanstalk enhanced health**.
  - **Lambda**: no health endpoint; monitor `Errors`, `Throttles`, `Duration`, DLQ/destination failures.
- **Exam triggers:** "Kubernetes pods receive traffic before they're ready" → **readiness probe**. "Container hangs but process is alive" → **liveness probe/ECS health check**. "Continuously test the login flow every 5 minutes" → **Synthetics [canary](00-developer-basics.md#deployment-strategies)**.

---

## Task 4.3: Optimize applications

### Concurrency (skill 4.3.1) `[also: DOP]`

- **Lambda concurrency** = requests per second × average duration (seconds). Example: 200 requests per second (rps) × 0.5 s = **100 concurrent executions**.
  - **Provisioned concurrency** (pre-initialized, no cold starts; on alias/version; schedule or auto scale it), **reserved concurrency** (guarantee + cap), **account limit** (default 1,000; burst scaling 1,000 per 10 s per function), **SQS ESM maximum concurrency**, **SnapStart**.
  - Lambda handles **one request per execution environment at a time**: in-function threads help only for parallel I/O within one invocation (e.g., fetching multiple S3 objects concurrently with async/threads).
- **EC2 Auto Scaling** (target tracking on CPU/`RequestCountPerTarget`, step, scheduled, predictive, warm pools) and **ECS service auto scaling** for container concurrency.
- **Thread pool management**: size pools to downstream limits (DB connections, API rate limits); bounded queues and back-pressure; async I/O (Node.js event loop, Python asyncio) for I/O-bound work; avoid blocking calls.
- **Connection pooling**: reuse SDK HTTP clients (keep-alive, create once outside the handler), DB connection pools in long-running apps (HikariCP etc.), **Relational Database Service (RDS) Proxy** for Lambda/serverless (pooling + multiplexing, IAM auth, faster failover), ElastiCache connection reuse.
- **Exam triggers:** "compute required concurrency" → rps × duration. "Many Lambdas exhaust DB connections" → **RDS Proxy**. "Initialize DB connection once per environment" → **init outside handler**. "Prevent cold starts at 9 AM peak" → **scheduled provisioned concurrency**.

### Optimizing memory and compute (skill 4.3.2) `[also: DOP]`

- **Lambda**: memory 128 MB–10 GB with **proportional Central Processing Unit [CPU]** (≈1 virtual CPU [vCPU] at 1,769 MB, up to 6 vCPU) and network; CPU-bound functions often get **cheaper** with more memory (shorter duration). Tune with **AWS Lambda Power Tuning** (Step Functions-based open-source tool testing memory settings for cost/speed) or **Compute Optimizer** Lambda recommendations. Watch `Max Memory Used`. Use **arm64 (Graviton)** for ~20% lower cost and often better performance. Reduce package size and init work; lazy-load rarely used modules; reuse connections; avoid unnecessary SDK calls.
- **EC2/containers**: right-size instance families (compute/memory optimized), Graviton, **Compute Optimizer**, Fargate task CPU/memory sizes, ECS task placement/bin packing.
- **Code-level**: efficient algorithms, batch API calls (`BatchGetItem`, `SendMessageBatch`, `PutRecords`), parallelize I/O, compress payloads, avoid N+1 queries, select only needed attributes (`ProjectionExpression`), use `Query` not `Scan`.
- **Exam triggers:** "CPU-heavy Lambda is slow" → **increase memory**. "Find the cost-optimal Lambda memory" → **Lambda Power Tuning / Compute Optimizer**. "Reduce Lambda cost with no code change" → **arm64** (if dependencies support it).

### Optimizing messaging (skill 4.3.3) `[also: SAA]`

- **SQS**: **long polling** (`WaitTimeSeconds=20`: fewer empty receives, lower cost), **batching** (`SendMessageBatch`/`DeleteMessageBatch` up to 10; Lambda ESM batch size + batching window), right **visibility timeout**, **DLQ** to stop poison messages looping, **First-In, First-Out (FIFO)** only when ordering is needed (high-throughput mode for scale; more message groups = more parallelism), **delay queues** for scheduling, **message size** (use S3 for large payloads), **SQS as buffer** to smooth spikes to slower consumers, **ESM maximum concurrency** to protect downstream, **partial batch responses**.
- **SNS fan-out**: one publish → many SQS queues/Lambdas (parallel independent processing, no repeated publishing); **filter policies** so subscribers receive only relevant messages (less wasted processing); **raw message delivery** to avoid envelope parsing; **FIFO topics** + FIFO queues when order matters; `PublishBatch`.
- **EventBridge vs SNS**: EventBridge for rich content-based routing/many sources; SNS for high-throughput fan-out with lower latency.
- **Kinesis**: batch `PutRecords`, Kinesis Producer Library (KPL) aggregation, enhanced fan-out for many consumers, parallelization factor for Lambda.
- **Exam triggers:** "too many empty ReceiveMessage calls" → **long polling**. "Reduce API calls sending many messages" → **batching**. "Three services each need every order event" → **SNS fan-out to three SQS queues**. "Each consumer only needs some messages" → **filter policies**.

### Implementing caching (skill 4.3.4) `[also: SAA]`

- **Amazon ElastiCache** (Valkey, Redis OSS, Memcached; serverless or node-based):
  - **[Lazy loading](00-developer-basics.md#caching) (cache-aside)**: `get` → miss → read DB → `set` with [Time To Live [TTL]](00-developer-basics.md#ttl-and-data-lifecycle). Pros: only requested data cached, cache failure isn't fatal. Cons: miss penalty (3 trips), stale data until TTL.
  - **Write-through**: write DB and cache together. Pros: fresh data. Cons: write penalty, caches unread data (use TTL).
  - **TTL** on all keys; **eviction policies** when memory full; **Redis data structures** (hashes for objects, **sorted sets** for leaderboards, `INCR` for counters/rate limits, pub/sub, [streams](00-developer-basics.md#streaming-data)), cluster mode for scale, replicas for reads/High Availability (HA); session store for stateless apps.
- **DynamoDB Accelerator (DAX)**: in-memory, **microsecond** reads for DynamoDB; **API-compatible** (swap client); **item cache** (GetItem/BatchGetItem) and **query cache** (Query/Scan results); write-through; **[eventually consistent](00-developer-basics.md#read-consistency) only** (strongly consistent reads pass through to DynamoDB); runs in your VPC; TTL configurable; not ideal for write-heavy or rarely repeated reads.
- **Amazon CloudFront**: edge caching of static and dynamic content; **cache policies** (TTL min/default/max; cache key headers/cookies/query strings: keep minimal for a high hit ratio), **origin request policies**, **invalidations** (or versioned file names), **Origin Shield**, compression; caching API GET responses; signed URLs/cookies for private content.
- **API Gateway caching** (REST): per stage, TTL 0–3,600 s (default 300), cache keys from parameters, per-method overrides, encryption; clients can bypass with `Cache-Control: max-age=0` only if allowed (require [authorization](00-developer-basics.md#authentication-vs-authorization) for invalidation).
- **In-function caching (Lambda)**: global variables/`/tmp` reused across warm invocations (config, reference data, tokens); Parameters and Secrets extension cache.
- **Exam triggers:** "DynamoDB read-heavy, microsecond latency, minimal code change" → **DAX**. "Reduce RDS load for repeated queries" → **ElastiCache lazy loading**. "Data must never be stale after writes" → **write-through**. "Global users fetch the same API responses" → **CloudFront/API Gateway caching**.

### Detecting performance bottlenecks with CloudWatch (skill 4.3.5) `[also: DOP]`

- **Metrics**: Lambda `Duration` p99, `Throttles`, `ConcurrentExecutions`, `Init Duration` (Logs Insights), `IteratorAge`; API Gateway `Latency` vs `IntegrationLatency`; DynamoDB throttles and `SuccessfulRequestLatency`; SQS `ApproximateAgeOfOldestMessage`; RDS CPU/connections/`ReadLatency`; ALB `TargetResponseTime`.
- **Tools**: **Lambda Insights** (CPU, memory, network, cold starts per function via extension), **Container Insights** (ECS/EKS CPU/memory per task/pod), **Application Signals** (service latency/error SLOs), **X-Ray/ServiceLens** traces, **Contributor Insights** (top talkers/hot keys), **Database Insights/Performance Insights** (slow SQL), **CloudWatch RUM** (front-end performance), **Synthetics**, **anomaly detection**, **Metrics Insights** (SQL over metrics to find top-N slow functions), **CodeGuru Profiler** (runtime profiling of hot code paths, where available).
- **Method**: start from user-facing latency/error metric → find the slowest service in traces → drill into its metrics/logs → fix (capacity, caching, query/index, concurrency, code) → verify with metrics.
- **Exam triggers:** "identify which DynamoDB keys are hot" → **Contributor Insights**. "Per-function memory/CPU detail" → **Lambda Insights**. "Queue backlog growing" → `ApproximateAgeOfOldestMessage` → scale consumers.

### AI tools for optimization (skill 4.3.6)

- **Kiro**: analyze code for inefficiencies (N+1 calls, unbounded scans, [synchronous](00-developer-basics.md#synchronous-vs-asynchronous) chains), propose refactors (batching, caching, async I/O), right-size [Infrastructure as Code [IaC]](00-developer-basics.md#infrastructure-as-code-iac) (Lambda memory, arm64), write benchmarks/tests to validate improvements; steering files capture performance standards.
- **AWS DevOps Agent**: recommendations in **infrastructure optimization**, observability, deployment pipeline and resilience based on your environment and incidents.
- **AWS Compute Optimizer** (ML-based rightsizing for Lambda memory, EC2, Elastic Block Store [EBS], ECS on Fargate), **CloudWatch** anomaly detection and AI-assisted investigations for performance regressions, **Amazon Bedrock** to summarize profiling data/logs in custom tooling.
- **Exam triggers:** "AI suggestions to make a slow function faster and cheaper" → **Kiro analysis + Compute Optimizer**. "Detect performance anomalies automatically" → **CloudWatch anomaly detection/investigations**.

---

## Gotchas and exam traps

- API Gateway **502** = bad Lambda response format/error; **504** = > 29 s; **403 Missing Authentication Token** = wrong path/stage.
- `Init Duration` only on cold starts; `Max Memory Used` ≈ configured → raise memory.
- `Latency` − `IntegrationLatency` = API Gateway overhead.
- X-Ray **annotations** are indexed/filterable; **metadata** isn't.
- Lambda tracing needs **active tracing + X-Ray permissions**; EC2/ECS need the **daemon/ADOT**.
- **EMF** = metrics from logs, no API latency; `PutMetricData` adds calls.
- Metric filters aren't retroactive.
- Concurrency = rps × duration; **RDS Proxy** for connection exhaustion.
- More Lambda memory = more CPU (can lower cost).
- **Long polling** + batching cut SQS cost; **filter policies** cut wasted fan-out work.
- DAX = DynamoDB only, eventually consistent reads only.
- Lazy loading → stale/miss penalty; write-through → fresh but write cost.
- Stack events show the **first** failure; ECS stopped reason explains task failures.

## Quick-recall summary

- **Root cause**: integration error table (API Gateway 4xx/5xx, Lambda timeouts/OOM/throttles/permissions, SQS/stream retries, DynamoDB errors, S3 auth, CORS, Bedrock errors), Lambda REPORT lines, metrics per service, X-Ray segments/annotations/service map, Logs Insights queries and filter patterns, deployment logs per tool, AI-assisted analysis (Kiro, DevOps Agent, CloudWatch investigations).
- **Observability**: structured JSON logs with correlation IDs and log levels (Powertools), custom metrics (EMF, PutMetricData, metric filters, high resolution), alarms + quota/deployment notifications, X-Ray/ADOT tracing, health checks/probes/canaries.
- **Optimization**: concurrency (provisioned/reserved, auto scaling, pools, RDS Proxy), memory/compute (power tuning, arm64, efficient calls), messaging (long polling, batching, fan-out, filters), caching (ElastiCache strategies, DAX, CloudFront, API Gateway, in-function), bottleneck detection (Lambda/Container Insights, Contributor Insights, Application Signals), AI optimization tools.
