# Domain 3: Testing and Deployment (22%)

> 🔰 New to a term? Concepts link to plain-English explanations in [the basics page](00-developer-basics.md) on first mention, and every abbreviation is spelled out on first use and listed in the [glossary](00-abbreviations-glossary.md).

> **About 11 of the 50 scored questions.**
>
> Tests whether you can package applications (code, [dependencies](00-developer-basics.md#dependency-management-and-packaging), [containers](00-developer-basics.md#containers-and-images), configuration), test them in Amazon Web Services (AWS) [environments](00-developer-basics.md#environments-and-stages) (SAM/CDK/CloudFormation, mocks, test events, environments, AI-generated tests), and ship them with AWS [Continuous Integration / Continuous Delivery [CI/CD]](00-developer-basics.md#cicd) services and safe deployment strategies with rollback.
>
> Tags like `[also: DOP]` mark material you met in another exam. It is still covered in full here.

## Table of contents

- [Exam tasks and skills](#exam-tasks-and-skills)
- [Task 3.1: Prepare application artifacts](#task-31-prepare-application-artifacts)
- [Task 3.2: Implement and automate testing](#task-32-implement-and-automate-testing)
- [Task 3.3: Deploy code with AWS CI/CD services](#task-33-deploy-code-with-aws-cicd-services)
- [Gotchas and exam traps](#gotchas-and-exam-traps)
- [Quick-recall summary](#quick-recall-summary)

## Exam tasks and skills

> Summarized from the official DVA-C03 exam guide. Exact wording: https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html

### Task 3.1: Prepare application artifacts to be deployed to AWS
- **3.1.1** Manage dependencies for code modules ([environment variables](00-developer-basics.md#environment-variables-and-configuration), configuration files, container images) within packages.
- **3.1.2** Build and manage container images (Elastic Container Registry [ECR], Dockerfiles, image tagging, lifecycle policies).
- **3.1.3** Use code repositories in deployment environments.
- **3.1.4** Manage environment-specific configuration (AppConfig, Parameter Store, Secrets Manager, [Application Programming Interface [API]](00-developer-basics.md#api) Gateway, Lambda).
- **3.1.5** Use API Gateway features (stage-specific URLs, endpoints).

### Task 3.2: Implement and automate application testing
- **3.2.1** Develop and validate tests (Serverless Application Model [SAM], Cloud Development Kit [CDK], CloudFormation).
- **3.2.2** Use [mock](00-developer-basics.md#unit-integration-and-end-to-end-tests) Application Programming Interfaces (APIs) for external dependencies (API Gateway).
- **3.2.3** Create application test events (JavaScript Object Notation [JSON] payloads for Lambda and API Gateway).
- **3.2.4** Create and configure application environments (Lambda aliases, container image tags, Amplify [branches](00-developer-basics.md#source-control), AWS Copilot environments, API Gateway APIs).
- **3.2.5** Use AWS Artificial Intelligence (AI) tools to generate tests and automate testing (execution, result analysis, regression, coverage).

### Task 3.3: Deploy code with AWS CI/CD services
- **3.3.1** Use Lambda packaging methods (.zip, container images, layers).
- **3.3.2** Deploy containerized apps (Elastic Container Service [ECS] task definitions, Elastic Kubernetes Service [EKS] manifests, Fargate, [rolling](00-developer-basics.md#deployment-strategies) updates).
- **3.3.3** Use [Infrastructure as Code [IaC]](00-developer-basics.md#infrastructure-as-code-iac) templates (CloudFormation, SAM).
- **3.3.4** Deploy revisions with CodeDeploy.
- **3.3.5** Orchestrate CI/CD with CodePipeline (linked repositories, multi-environment progression).
- **3.3.6** Implement deployment strategies (blue/green, canary, rolling).
- **3.3.7** Implement automatic or manual rollback (CloudFormation, CodeDeploy).
- **3.3.8** Use source control features for versions and releases (labels, branches).
- **3.3.9** Use AWS AI tools in CI/CD (deployment approvals, environment provisioning, post-deployment validation).

---

## Task 3.1: Prepare application artifacts

### Managing dependencies and packages (skill 3.1.1) `[also: DOP]`

- **Language packaging**: Python (`requirements.txt`/`pyproject.toml`, install into the package directory or a layer; native dependencies must match the Lambda OS/architecture → build in `sam build --use-container` or on Amazon Linux/arm64), Node.js (`package.json` + lock file, `npm ci`, bundle/tree-shake with esbuild), Java (Maven/Gradle fat JAR), .NET, Go (static binary).
- **Lambda**: dependencies inside the zip, in **layers** (shared libraries under `/opt/python`, `/opt/nodejs/node_modules`, `/opt/java/lib`), or in a **container image**. Keep packages small (faster cold starts).
- **Configuration**: environment variables (non-secret settings per environment), config files baked into artifacts (avoid environment-specific values in artifacts → **build once, deploy many**), externalized config (AppConfig, Parameter Store, Secrets Manager).
- **Private packages**: **AWS CodeArtifact** (managed artifact repository for npm, PyPI, Maven, Gradle, NuGet, Cargo, Swift, generic): **domains** and **repositories**, **upstream repositories** and **external connections** to public registries (proxy + cache public packages), package version **status** (published/unlisted/archived/disposed), Identity and Access Management (IAM) + resource policies, cross-account sharing; authenticate with `aws codeartifact login --tool npm|pip|twine` (token valid up to 12 h); use from CodeBuild.
- **Exam triggers:** "share internal libraries across teams and cache public packages" → **CodeArtifact** with upstream/external connection. "Lambda fails importing a compiled library" → build for the **Lambda OS/architecture** (container build). "Same artifact promoted across environments" → externalize config.

### Building and managing container images (skill 3.1.2) `[also: DOP, SAA]`

- **Dockerfile basics**: `FROM` (base image; for Lambda: `public.ecr.aws/lambda/python:3.13` etc.), `WORKDIR`, `COPY`, `RUN` (install), `ENV`, `EXPOSE`, `USER` (non-root), `ENTRYPOINT`/`CMD`; **multi-stage builds** (build in one stage, copy artifacts into a slim runtime stage); `.dockerignore`; layer caching (copy dependency manifests before source); `--platform linux/arm64` for Graviton.
- **Amazon ECR**: create repository → authenticate `aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com` → `docker tag` → `docker push`.
  - **Tagging**: [semantic version](00-developer-basics.md#semantic-versioning) tags (`1.4.2`), Git commit SHA tags (traceable), environment tags (`prod`) — prefer immutable tags; **tag immutability** setting prevents overwriting tags; deploy by **digest** (`@sha256:…`) for exactness; avoid relying on `latest`.
  - **Lifecycle policies**: rules by tag status/prefix and age/count (e.g., expire untagged images after 14 days, keep last 30 `release-*` images); test with preview.
  - **Scanning**: basic (on push) or **enhanced** (Amazon Inspector, continuous, Operating System [OS] + language packages).
  - **Replication** (cross-Region/account), **pull-through cache**, repository policies (cross-account pull), Key Management Service (KMS) encryption, **image signing** (AWS Signer + Notation) and verification.
- **Build services**: **CodeBuild** (docker builds need **privileged mode**; push to ECR with the build role), Elastic Compute Cloud (EC2) Image Builder container recipes, local Docker/Finch.
- **Exam triggers:** "prevent overwriting a released image tag" → **tag immutability**. "Storage cost from many old images" → **lifecycle policy**. "Trace a running container back to source" → **commit SHA tag**. "CodeBuild can't run docker build" → enable **privileged mode**.

### Using code repositories (skill 3.1.3) `[also: DOP]`

- **Git hosting**: GitHub, GitLab, Bitbucket (connected to AWS via **AWS CodeConnections**, formerly CodeStar Connections, an Open Authorization [OAuth]/GitHub App connection that CodePipeline, CodeBuild and other services use), AWS CodeCommit (available to existing customers).
- **Triggers**: CodePipeline source actions start on pushes to a branch, **tags** or **pull requests** (with filters on branches/file paths via pipeline **triggers**); CodeBuild webhooks for Pull Request (PR) builds; CloudFormation **Git sync** deploys stacks from a repo.
- **Branching**: feature branches + pull requests with CI checks; main/trunk deploys; release branches; **tags** for releases (`v1.2.0`).
- **Repo hygiene**: no secrets (use Secrets Manager; scanners), `.gitignore`, buildspec/appspec/templates versioned with code (**everything as code**).
- **Exam triggers:** "start the pipeline when code is pushed to `main` in GitHub" → **CodePipeline source action via CodeConnections**. "Run tests on every pull request" → **CodeBuild PR webhook / pipeline PR trigger**.

### Environment-specific configuration (skill 3.1.4) `[also: DOP]`

| Mechanism | Best for |
|---|---|
| **AWS AppConfig** | Dynamic application configuration and **feature flags** deployed gradually with **validators** (JSON Schema or Lambda), **deployment strategies** (linear/exponential, bake time), automatic **rollback on CloudWatch alarms**; apps fetch via the **AppConfig Agent/Lambda extension** (local cache, polling); config from AppConfig hosted store, Simple Storage Service (S3), Parameter Store, Secrets Manager, CodePipeline |
| **Parameter Store** | Hierarchical config per environment (`/myapp/dev/db-url`, `/myapp/prod/db-url`) with `GetParametersByPath`; SecureString for simple secrets |
| **Secrets Manager** | Credentials with rotation |
| **Lambda environment variables** | Static per-function settings (table names, stage); different per alias isn't possible (env vars belong to the version) → use per-environment functions/stacks or stage variables/AppConfig |
| **API Gateway stage variables** | Per-stage values (e.g., `lambdaAlias=prod`, backend Uniform Resource Locator [URL]) referenced in integrations (`arn:...:function:name:${stageVariables.lambdaAlias}`); the Lambda permission must allow each alias |
| **IaC parameters/mappings** | Environment-specific infrastructure values (instance sizes, domain names) |

- **Exam triggers:** "toggle a feature for 10% of users and roll back automatically if errors rise" → **AppConfig feature flag with deployment strategy + alarm rollback**. "Same API, different Lambda alias per stage" → **stage variables**. "Validate configuration before deploying it" → **AppConfig validators**.

### API Gateway stages and endpoints (skill 3.1.5) `[also: DOP]`

- **Stages**: a **deployment** (snapshot of the API) is published to a **stage** (`dev`, `test`, `prod`) → **stage-specific URL**: `https://{api-id}.execute-api.{region}.amazonaws.com/{stage}/{resource}`. Changes aren't live until you **deploy** ([REST](00-developer-basics.md#rest-and-http-methods)). Hypertext Transfer Protocol (HTTP) APIs can **auto-deploy** (`$default` stage serves at the root).
- **Stage settings**: stage variables, throttling, caching, logging (execution/access logs), X-Ray tracing, Web Application Firewall [WAF] (REST), client [certificates](00-developer-basics.md#certificates-and-ssh-keys), **canary settings** (send a % of stage traffic to a new deployment, with canary stage variable overrides; promote or roll back).
- **Endpoints**: **edge-optimized** (via CloudFront POPs; custom domain cert in us-east-1), **regional** (cert in the API's Region; put your own CloudFront in front if needed), **private** (Virtual Private Cloud [VPC] interface endpoint `execute-api`; **resource policy required**; reach via endpoint-specific Domain Name System [DNS] or private DNS).
- **Custom domain names**: `api.example.com` with an AWS Certificate Manager (ACM) certificate, **base path mappings** (`/v1` → API A stage prod, `/v2` → API B) or routing rules; Route 53 alias to the domain's target; disable the default `execute-api` endpoint to force the custom domain.
- **Versioning APIs**: base paths (`/v1`, `/v2`), separate stages, or separate APIs.
- **Exam triggers:** "test a new API version on 10% of prod traffic" → **stage canary**. "API reachable only from our VPC" → **private API + resource policy**. "Changes not showing" → **deploy the API to the stage**.

---

## Task 3.2: Implement and automate testing

### Developing and validating tests with SAM, CDK and CloudFormation (skill 3.2.1) `[also: DOP]`

- **AWS SAM (Serverless Application Model)**:
  - Template extension (`Transform: AWS::Serverless-2016-10-31`) with resources `AWS::Serverless::Function`, `Api`, `HttpApi`, `SimpleTable`, `LayerVersion`, `StateMachine`, `Application`, `Connector` (permissions between resources); **Globals** section; **policy templates** (`DynamoDBCrudPolicy`, `S3ReadPolicy`, `SQSPollerPolicy`).
  - **SAM Command Line Interface (CLI)**: `sam init`, **`sam build`** (resolve dependencies, `--use-container`), **`sam local invoke`** (run a function locally in Docker with an event file `-e event.json`), **`sam local start-api`** (local API Gateway emulation), `sam local start-lambda`, **`sam local generate-event`** (sample events: `s3 put`, `apigateway aws-proxy`, `sqs receive-message`, `dynamodb update`…), **`sam deploy --guided`** (package to S3/ECR + CloudFormation change set), **`sam sync --watch`** (fast iterative cloud deploys, dev only), **`sam remote invoke`** (invoke deployed function in the cloud), **`sam remote test-event`** (shareable test events stored in the cloud), `sam logs`, `sam traces`, `sam validate` (template linting with cfn-lint), `sam pipeline init` (generate CI/CD pipelines).
  - **Testing approach**: **unit tests** (pytest/Jest with mocked AWS [Software Development Kit [SDK]](00-developer-basics.md#sdk-and-cli) clients: `moto`, `botocore.stub.Stubber`, `aws-sdk-client-mock`), **integration tests in the cloud** against a deployed dev stack (preferred over heavy local emulation), local invoke for quick feedback.
- **AWS CDK testing**: **`assertions` module**: **fine-grained assertions** (`Template.fromStack(stack).hasResourceProperties('AWS::Lambda::Function', {...})`, `resourceCountIs`, `Match.objectLike`), **snapshot tests** (compare synthesized template JSON against a stored snapshot to catch unintended changes), validation with **Aspects**/cdk-nag; `cdk synth` and `cdk diff` in CI.
- **AWS CloudFormation validation**: `aws cloudformation validate-template` (syntax), **cfn-lint** (schema and best-practice linting), **CloudFormation Guard** (`cfn-guard validate` policy-as-code rules, e.g., all buckets encrypted), **change sets** to preview, **TaskCat** (test deploy templates in multiple Regions), stack **drift detection** after deployment, **hooks** for proactive checks.
- **Exam triggers:** "run a Lambda locally with a sample S3 event" → **`sam local generate-event s3 put` + `sam local invoke -e`**. "Test that our CDK stack creates an encrypted bucket" → **CDK assertions** (fine-grained). "Detect unexpected template changes" → **snapshot test**. "Enforce rules on templates in CI" → **cfn-guard / cfn-lint**.

### Mock APIs for external dependencies (skill 3.2.2)

- **API Gateway mock integrations**: an endpoint returns a configured response (integration response with mapping template, status code) **without calling a backend**, so front-end or dependent teams can develop/test before the real service exists, or simulate a third-party API (including error codes via request parameters/templates selecting responses).
- **Other mocking**: unit-test mocks of SDK clients (moto, Stubber, aws-sdk-client-mock), **WireMock**/local stubs, **Lambda functions as fake backends**, Step Functions **mocked service integrations** (Step Functions Local mock config) and `TestState` API, DynamoDB Local, LocalStack (third party), contract tests against OpenAPI definitions; **AWS Fault Injection Service** for testing failure behaviors in integration environments.
- **Exam triggers:** "front-end team needs the API before the backend is built" → **API Gateway mock integration**. "Unit test code that calls DynamoDB without AWS" → **mock the SDK client** (or DynamoDB Local).

### Application test events (skill 3.2.3)

- **Lambda console test events**: JSON payloads (private or **shareable** test events stored in the account via EventBridge schema registry), built from templates per source (API Gateway proxy, S3 put, Simple Queue Service [SQS], Simple Notification Service [SNS], DynamoDB Streams, Kinesis, EventBridge, Cognito triggers).
- **CLI**: `aws lambda invoke --function-name f --payload fileb://event.json out.json` (CLI v2 needs `--cli-binary-format raw-in-base64-out` for raw JSON with `file://`), `--invocation-type Event` for async, `--qualifier prod`, `--log-type Tail` to get the last 4 KB of logs ([base64](00-developer-basics.md#base64)).
- **SAM**: `sam local generate-event`, `sam local invoke -e`, `sam remote invoke --event-file`.
- **API Gateway console "Test"** for a method: supply path/query/headers/body and see the integration request/response and logs without deploying; `test-invoke-method` API.
- **Shape matters**: API Gateway proxy event (`body` string, `pathParameters`, `queryStringParameters`, `headers`, `requestContext`), SQS event (`Records[].body`, `messageId`, `receiptHandle`), S3 event (`Records[].s3.bucket.name`, URL-encoded `object.key`), Kinesis (`Records[].kinesis.data` base64), DynamoDB Streams (`Records[].dynamodb.NewImage` typed JSON), EventBridge (`detail-type`, `source`, `detail`).
- **Exam triggers:** "simulate an SQS-triggered invocation" → test event with the **SQS event structure**. "Test an API method without deploying" → **API Gateway console test**.

### Application environments (skill 3.2.4) `[also: DOP]`

- **Lambda aliases**: `dev`, `test`, `prod` aliases pointing to versions; event sources/API stages target aliases; promote by moving the alias; **weighted aliases** for canary.
- **Container image tags**: environment promotion by retagging the same image digest (`build-123` → `staging` → `prod`) or deploying by digest per environment.
- **AWS Amplify (Hosting)**: Git-based CI/CD for web front ends and full-stack apps; **branches map to environments** (`main` → prod, `dev` → dev), **pull request previews** (temporary environments per PR), **Gen 2 sandboxes** (per-developer cloud backends), password-protected branches, environment variables per branch, custom domains with subdomains per branch, atomic deployments, rollback to previous builds.
- **AWS Copilot** (CLI for ECS/App Runner): **applications** → **environments** (`copilot env init/deploy`: separate VPC/cluster per env such as test and prod) → **services** (load-balanced web service, backend service, worker service, request-driven service) and **jobs**; `copilot svc deploy --env test`; `copilot pipeline init/deploy` builds a CodePipeline across environments; manifests as YAML Ain't Markup Language (YAML); generates CloudFormation.
- **API Gateway**: separate **stages** or separate APIs per environment; stage variables to bind to environment-specific backends.
- **IaC per environment**: same template with parameters per environment (SAM `samconfig.toml` environments, CDK stages/`Environment` objects, CloudFormation stacks per account), ideally **separate AWS accounts** per environment.
- **Exam triggers:** "every feature branch gets its own preview of the web app" → **Amplify branch/PR previews**. "Deploy a containerized service to test then prod environments with a simple CLI" → **AWS Copilot environments**. "Point prod traffic to a validated Lambda version" → **alias**.

### AI tools for testing (skill 3.2.5)

- **What AI assistants do**: **generate unit tests** from code or specs (edge cases, mocks), **generate integration/API tests** from OpenAPI specs, **increase coverage** by identifying untested paths, **analyze test failures** (explain stack [traces](00-developer-basics.md#logs-metrics-and-traces), suggest fixes), **automate regression suites** (re-run and triage on each change), produce test data, keep tests updated as code changes.
- **Kiro**: [spec-driven](00-developer-basics.md#ai-assisted-development) development produces **acceptance criteria** that become tests; **agent hooks** can automatically create/update tests when files are saved or run tests on events; **headless mode in CI** to run agent tasks (e.g., fix failing tests, write missing tests) inside pipelines; steering files enforce testing conventions; [Model Context Protocol [MCP]](00-developer-basics.md#mcp) to connect test tools.
- **In CI/CD**: run AI-generated tests like any others in **CodeBuild** with **test reports** (JUnit/Cucumber/NUnit reports, code coverage reports) visible in the console; gate merges on pass rate and coverage.
- **[Guardrails](00-developer-basics.md#guardrails)**: review AI-generated tests for meaningful assertions (not just passing), avoid secrets/[Personally Identifiable Information [PII]](00-developer-basics.md#pii-and-phi-masking-and-redaction) in test data, keep humans approving merges.
- **Exam triggers:** "automatically generate unit tests for new code and run them in the pipeline" → **AI tool (Kiro hooks/specs) + CodeBuild test reports**. "Explain why tests fail after a change" → AI-assisted test result analysis.

---

## Task 3.3: Deploy code with AWS CI/CD services

### Lambda packaging methods (skill 3.3.1) `[also: DOP]`

| Method | Limits | Use when |
|---|---|---|
| **.zip archive** | **50 MB zipped** direct upload; **250 MB unzipped** (including layers) via S3 | Most functions; fastest cold starts for small packages |
| **Layers** | Up to **5 layers**, counted in 250 MB unzipped | Share dependencies/runtimes across functions; separate dependency updates from code |
| **Container image** | Up to **10 GB**, stored in **ECR** (same account, or cross-account with repository policy); must implement Lambda Runtime API (AWS base images or runtime interface client) | Large dependencies (Machine Learning [ML] libraries), existing container tooling, custom OS packages |

- **Deploy commands**: `aws lambda update-function-code --zip-file fileb://function.zip` or `--s3-bucket/--s3-key` or `--image-uri`; `publish-version`; `update-alias`. Through IaC: SAM `CodeUri` (packaged to S3 by `sam deploy`), CDK `Code.fromAsset`, CloudFormation `Code.S3Bucket/S3Key/ImageUri`.
- **Optimization**: exclude dev dependencies/tests, minify/bundle (esbuild), use layers for big shared libs, arm64 builds, **SnapStart** for Java/Python/.NET.
- **Exam triggers:** "deployment package exceeds 250 MB" → **container image**. "Several functions share a 40 MB library" → **layer**.

### Deploying containerized applications (skill 3.3.2) `[also: DOP, SAA]`

- **Amazon ECS**:
  - **Task definition** (versioned **revisions**): `family`, container definitions (image, cpu/memory, port mappings, environment, **secrets**, log configuration `awslogs`/FireLens, health check, `essential`), `networkMode` (`awsvpc` required for Fargate: each task gets an Elastic Network Interface [ENI] + Security Group [SG]), **task role** and **task execution role**, `requiresCompatibilities` (`FARGATE`/`EC2`), task-level Central Processing Unit (CPU)/memory for Fargate.
  - **Service**: desired count, launch type/capacity provider (**Fargate**, Fargate Spot, EC2 Auto Scaling group [ASG] capacity providers, ECS Managed Instances), load balancer (target group; `ip` target type for awsvpc), **service auto scaling**, service discovery/Service Connect.
  - **Deployment types**: **rolling update** (ECS controller; `minimumHealthyPercent` and `maximumPercent` control how many tasks are replaced at once; **deployment [circuit breaker](00-developer-basics.md#timeouts-and-circuit-breakers)** with **rollback** when tasks fail to reach steady state; CloudWatch alarm-based rollback), ECS native **blue/green** (and linear/canary) with lifecycle hooks, or **CodeDeploy blue/green** (`AppSpec` with `TaskDefinition`, container name/port; test listener; traffic shifting `ECSCanary10Percent5Minutes`, `ECSLinear10PercentEvery1Minutes`, `ECSAllAtOnce`).
  - Deploy a new image: register a new task definition revision → `update-service --task-definition family:rev` (or `--force-new-deployment` when using a mutable tag).
- **Amazon EKS**: Kubernetes **manifests** (YAML) for `Deployment` (replicas, pod template, **rolling update** strategy with `maxSurge`/`maxUnavailable`, [readiness](00-developer-basics.md#health-checks-liveness-and-readiness)/liveness probes), `Service`, `Ingress` (AWS Load Balancer Controller → Application Load Balancer [ALB]), `ConfigMap`/`Secret`, `HorizontalPodAutoscaler`; apply with `kubectl apply -f`, **Helm** charts, GitOps (Argo CD/Flux); `kubectl rollout status/undo` for rollback; IAM for pods via **EKS Pod Identity**/IAM Roles for Service Accounts (IRSA); **Fargate profiles** run pods without nodes.
- **AWS Fargate launch type**: serverless containers for ECS/EKS: specify CPU/memory per task/pod, no instances to manage, `awsvpc` networking, ephemeral storage (20 GB default, up to 200 GB), platform versions.
- **Exam triggers:** "update ECS service to a new image with zero downtime" → new **task definition revision** + rolling update with healthy percent settings. "Automatically roll back failing ECS deployments" → **deployment circuit breaker**. "Canary traffic shifting for ECS" → **CodeDeploy blue/green** (or ECS native canary). "Kubernetes rollout with no unavailable pods" → `maxUnavailable: 0`.

### Infrastructure as code templates (skill 3.3.3) `[also: DOP]`

- **CloudFormation**: template sections (`Parameters`, `Mappings`, `Conditions`, `Resources`, `Outputs`/exports), intrinsic functions (`Ref`, `Fn::GetAtt`, `Fn::Sub`, `Fn::ImportValue`, `Fn::If`), pseudo parameters, **change sets**, **stack updates** (replacement vs no interruption), **DeletionPolicy**, nested stacks, cross-stack references, **dynamic references** for secrets, `aws cloudformation deploy` (create/update with change set) and `package` (upload local artifacts to S3 and rewrite paths), capabilities (`CAPABILITY_IAM`, `CAPABILITY_NAMED_IAM`, **`CAPABILITY_AUTO_EXPAND`** for SAM transforms/macros).
- **SAM templates**: shorthand serverless resources expanded by the transform into CloudFormation; `Events` on functions (Api, HttpApi, S3, SQS, Kinesis, DynamoDB, Schedule, EventBridgeRule, SNS), **`AutoPublishAlias`** (publishes a version and updates the alias on each deploy) + **`DeploymentPreference`** (`Type: Canary10Percent5Minutes | Linear10PercentEvery1Minute | AllAtOnce…`, `Alarms`, `Hooks: PreTraffic/PostTraffic`) → **CodeDeploy** gradual Lambda deployments; `Globals`; `Layers`; `Policies`.
- **CDK**: constructs → `cdk synth` → CloudFormation; `cdk deploy`; **CDK Pipelines**.
- **Exam triggers:** "gradually shift traffic to new Lambda code with automatic rollback on alarms in a SAM app" → **`AutoPublishAlias` + `DeploymentPreference`**. "Package local Lambda code for a CloudFormation deploy" → **`cloudformation package`** / `sam deploy`. "Deploy fails: requires CAPABILITY_AUTO_EXPAND" → acknowledge it for transforms.

### AWS CodeDeploy (skill 3.3.4) `[also: DOP]`

- **Components**: **application** → **deployment group** (targets: EC2 tags/Auto Scaling groups [ASGs], on-prem instances, Lambda functions, ECS services; service role; load balancer; rollback and alarm settings) → **deployment configuration** → **revision** (S3 bundle or GitHub for EC2; AppSpec for Lambda/ECS).
- **EC2/on-premises**: **CodeDeploy agent** required on instances; **`appspec.yml`** at the bundle root with `files` (source → destination), `permissions`, and **lifecycle event hooks** in order: `ApplicationStop` → `DownloadBundle` → `BeforeInstall` → `Install` → `AfterInstall` → `ApplicationStart` → `ValidateService` (plus `BeforeBlockTraffic`/`BlockTraffic`/`AfterBlockTraffic`, `BeforeAllowTraffic`/`AllowTraffic`/`AfterAllowTraffic` with load balancers); scripts with `timeout` and `runas`.
  - **In-place** deployments with configs **`CodeDeployDefault.AllAtOnce`**, **`HalfAtATime`**, **`OneAtATime`** or custom minimum healthy hosts.
  - **Blue/green** for EC2: new instances (copy ASG) behind the LB, then reroute; terminate or keep the original instances.
- **Lambda**: AppSpec specifies function name, alias, current and target versions; hooks `BeforeAllowTraffic`/`AfterAllowTraffic` (Lambda validation functions that call `PutLifecycleEventHookExecutionStatus`); configs **`LambdaCanary10Percent5Minutes`** (10% then the rest after 5 min; also 10/15/30 min variants), **`LambdaLinear10PercentEvery1Minute`** (also 2/3/10 min), **`LambdaAllAtOnce`**.
- **ECS**: blue/green with replacement task set, production and **test listeners**, hooks (`BeforeInstall`, `AfterInstall`, `AfterAllowTestTraffic`, `BeforeAllowTraffic`, `AfterAllowTraffic`), canary/linear/all-at-once configs.
- **Troubleshooting**: logs on instances (`/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log`, `/var/log/aws/codedeploy-agent/`), deployment events in console show the failing hook; common issues: agent not running/not installed, instance profile can't read the S3 revision, hook script exits non-zero or times out, `ApplicationStop` fails because the previous revision's script is broken (deploy with `--ignore-application-stop-failures`), wrong file permissions, health check failures behind the LB.
- **Exam triggers:** "deploy to 100 EC2 instances one at a time" → **`OneAtATime`** in-place. "Run a validation script after install" → **`ValidateService`** hook. "Lambda canary with automated tests before shifting" → **`BeforeAllowTraffic` hook** + canary config. "Deployment fails at ApplicationStop" → previous revision's script; **ignore application stop failures**.

### AWS CodePipeline and CodeBuild (skill 3.3.5) `[also: DOP]`

- **CodePipeline**:
  - **Pipeline type V2** (current: pipeline-level **variables**, **triggers** with Git tag/branch/file-path filters and PR events, execution modes **QUEUED**/**SUPERSEDED**/**PARALLEL**), **stages** → **actions** (source, build, test, deploy, approval, invoke) running sequentially or in parallel (`runOrder`), **artifacts** passed between actions via the **S3 artifact store** (encrypted with KMS; cross-Region actions need artifact buckets per Region).
  - **Source providers**: GitHub/GitLab/Bitbucket via **CodeConnections**, CodeCommit, **S3** (versioned object), **ECR** (image push).
  - **Build/test**: **CodeBuild**, Jenkins. **Deploy**: CloudFormation (create/update stack, change sets), CodeDeploy, **ECS** (standard rolling using `imagedefinitions.json`, or **CodeDeploy blue/green** using `imageDetail.json` + `appspec.yaml` + `taskdef.json`), Elastic Beanstalk, S3, Lambda, AppConfig, Service Catalog, Amazon ECR/EKS deploy actions, commands action.
  - **Manual approval** action (SNS notification; approver needs `codepipeline:PutApprovalResult`), **cross-account deployments** (CloudFormation actions with roles in target accounts; KMS customer managed key for artifacts shared to those accounts), **stage conditions/rollback** (automatically roll back a stage on failure to the last successful execution; entry/exit conditions using CloudWatch alarms), **notifications** (EventBridge/notification rules → SNS/chat).
  - Multi-environment progression: Source → Build → Deploy-Dev → Test → Approval → Deploy-Prod (often to separate accounts).
- **CodeBuild**:
  - Managed build containers; **`buildspec.yml`**: `version: 0.2`, `env` (`variables`, **`parameter-store`**, **`secrets-manager`**, `exported-variables`), `phases` (`install` with `runtime-versions`, `pre_build`, `build`, `post_build`; `on-failure`), `artifacts` (files, `discard-paths`, `base-directory`, secondary artifacts), `reports` (test and coverage reports), `cache` (S3 or local caching: Docker layer, source, custom paths).
  - **Environment**: managed images (Amazon Linux, Ubuntu, Windows, arm), custom images from ECR, compute types, **Lambda compute** for fast small builds, **privileged mode** for Docker builds, **VPC configuration** for private resources (needs Network Address Translation [NAT]/endpoints for internet/AWS APIs), **service role** permissions, build **timeout** (default 60 min, up to 36 h), batch builds, webhook triggers, **CloudWatch Logs/S3 logs**, local testing with the CodeBuild agent.
- **Exam triggers:** "require manager approval before production deploy" → **manual approval action**. "Pass secrets to the build" → buildspec **`env.secrets-manager`/`parameter-store`**. "Build needs to reach a private Relational Database Service (RDS)" → **CodeBuild VPC config**. "Speed up builds downloading dependencies" → **CodeBuild cache**. "Deploy to prod account from tools account" → **cross-account roles + KMS key for artifacts**.

### Deployment strategies (skill 3.3.6) `[also: DOP]`

| Strategy | AWS implementations |
|---|---|
| **All at once / in place** | CodeDeploy `AllAtOnce`, Beanstalk all at once, Lambda `AllAtOnce` |
| **Rolling** | CodeDeploy `HalfAtATime`/`OneAtATime`, ECS rolling update, EKS Deployments, Beanstalk rolling (+ additional batch), ASG instance refresh |
| **Immutable** | Beanstalk immutable (new ASG), new task sets/instances |
| **Blue/green** | CodeDeploy blue/green (EC2, ECS), ECS native blue/green, Beanstalk **Canonical Name (CNAME) swap**, Route 53/ALB weighted switch, API Gateway stages, Lambda alias switch |
| **Canary** | CodeDeploy `LambdaCanary…`/`ECSCanary…`, API Gateway **stage canary**, Lambda **weighted alias**, Beanstalk **traffic splitting**, AppConfig gradual rollout, CloudFront continuous deployment |
| **Linear** | CodeDeploy `…Linear…`, AppConfig linear strategy |

- **AWS Elastic Beanstalk deployment policies**: **all at once** (fastest, downtime), **rolling** (reduced capacity), **rolling with additional batch** (full capacity), **immutable** (new instances in a temporary ASG; safest, quick rollback by terminating), **traffic splitting** (canary to new instances for an evaluation period), **blue/green** (clone environment, then **swap environment URLs (CNAME)**). Configuration via `.ebextensions/*.config`, `.platform` hooks, `Procfile`, saved configurations; worker environments use SQS (`cron.yaml` for periodic tasks); EB CLI (`eb init`, `eb create`, `eb deploy`, `eb swap`).
- **Exam triggers:** "Beanstalk deploy with zero downtime and full capacity, cheapest" → **rolling with additional batch**. "Safest Beanstalk deploy with fastest rollback" → **immutable** (or blue/green swap). "Test a new Beanstalk version on 10% traffic" → **traffic splitting**.

### Rollback (skill 3.3.7) `[also: DOP]`

- **CloudFormation**: automatic rollback on stack create/update failure; **rollback triggers** (CloudWatch alarms watched during and for up to 180 min after the update); `--disable-rollback` for debugging; `UPDATE_ROLLBACK_FAILED` → `continue-update-rollback` (skip resources); manual rollback = redeploy previous template/parameters.
- **CodeDeploy**: **automatic rollback** when a deployment fails or when **CloudWatch alarms** go into ALARM (configured on the deployment group); manual rollback = redeploy the last known good revision (CodeDeploy creates a new deployment of the previous revision); `stop-deployment --auto-rollback-enabled`.
- **Lambda**: move the alias back to the previous version (instant); SAM `DeploymentPreference` rolls back on alarms/hook failures.
- **ECS**: circuit breaker rollback; redeploy previous task definition revision; CodeDeploy blue/green reroutes to original task set.
- **CodePipeline**: **stage rollback** to the last successful pipeline execution's artifacts; retry failed actions.
- **API Gateway**: redeploy a previous deployment to the stage (deployment history) or roll back canary.
- **Beanstalk**: deploy a previous application version; swap CNAMEs back; immutable rollback = terminate new instances.
- **AppConfig**: automatic rollback on alarm during deployment.
- **Exam triggers:** "automatically roll back when the error-rate alarm fires after deploy" → **CodeDeploy/SAM alarms** or CloudFormation rollback triggers. "Instantly revert a bad Lambda release" → **point alias to previous version**.

### Source control for versions and releases (skill 3.3.8) `[also: DOP]`

- **Branches**: `main`/trunk (deployable), feature branches, release branches, hotfix branches; branch protection (required reviews and status checks).
- **Tags/labels**: annotated Git tags for releases (`v2.3.0`, semantic versioning) trigger release pipelines (CodePipeline V2 tag triggers); container image tags and Lambda **version descriptions** link deployments to commits; GitHub releases/changelogs.
- **Merging**: pull requests with CI checks; squash/merge commits; revert commits for rollbacks in GitOps.
- **Traceability**: pass the commit SHA through the pipeline (CodePipeline variables `#{SourceVariables.CommitId}`) into image tags, Lambda version descriptions, CloudFormation parameters, AppConfig versions.
- **Exam triggers:** "only deploy to production when a release tag is pushed" → **pipeline trigger on Git tags**. "Identify which commit is running in prod" → tag artifacts with **commit SHA**.

### AI tools in CI/CD (skill 3.3.9)

- **Uses**: **automated deployment approvals** (AI reviews change sets/diffs, test results and risk, then recommends or gates approval, with humans in the loop for production), **environment provisioning** (generate IaC from natural language/specs; create ephemeral test environments), **post-deployment validation** (analyze metrics, logs and traces after release, detect anomalies, recommend rollback), automated code review on pull requests, release notes generation, pipeline failure analysis.
- **AWS tools**: **Kiro** (headless mode in CI to run agent tasks such as generating IaC, fixing builds or writing tests; hooks; MCP integrations with AWS services), **AWS DevOps Agent** (release management: release-readiness reviews of code changes, dependency mapping across repos, autonomous release testing; production incident investigation and rollback recommendations), **Amazon Bedrock** called from CodeBuild/Lambda/Step Functions to summarize diffs or analyze test output, **CloudWatch** anomaly detection and AI-assisted investigations for post-deploy checks.
- **Pattern**: CodePipeline → CodeBuild step invokes an AI review (Bedrock/Kiro headless) → writes a report/score → **manual approval** action (or automated gate on score) → deploy with **CodeDeploy canary + CloudWatch alarm rollback** → post-deployment validation step.
- **Exam triggers:** "use AI to review each change set and summarize risk before approval" → **AI step (Bedrock/Kiro) + manual approval**. "Automatically validate after deployment and roll back on anomalies" → **post-deploy validation + alarm-based rollback**.

---

## Gotchas and exam traps

- **Build once, deploy many**: externalize environment config (AppConfig, Parameter Store, stage variables).
- Native Python/Node dependencies must match **Lambda OS/arch** (`sam build --use-container`).
- ECR **tag immutability** prevents overwrites; **lifecycle policies** expire images; CodeBuild Docker builds need **privileged mode**.
- API Gateway REST changes need a **deployment to a stage**; stage URL includes the stage name.
- Stage variables pointing to Lambda aliases need **Lambda permissions per alias**.
- **`sam local invoke`** + `sam local generate-event` for local tests; CDK **assertions/snapshots** for IaC tests.
- **Mock integration** = API without backend.
- Lambda zip **50 MB / 250 MB**, image **10 GB**, **5 layers**.
- ECS rolling: `minimumHealthyPercent`/`maximumPercent`; **circuit breaker** rollback; new image = new **task definition revision**.
- **appspec.yml** (CodeDeploy) vs **buildspec.yml** (CodeBuild); hooks order ApplicationStop → BeforeInstall → AfterInstall → ApplicationStart → ValidateService.
- CodeDeploy EC2 needs the **agent** + instance profile S3 access.
- SAM gradual deploys: **AutoPublishAlias + DeploymentPreference**.
- Beanstalk: immutable = safest; rolling with additional batch = full capacity; blue/green = **CNAME swap**.
- CodePipeline cross-account needs **customer managed KMS key** for artifacts + roles.

## Quick-recall summary

- **Artifacts**: dependencies, layers, CodeArtifact, Dockerfiles, ECR (tags, immutability, lifecycle, scanning), repos via CodeConnections, AppConfig/Parameter Store/Secrets Manager/stage variables, API Gateway stages/endpoints/custom domains.
- **Testing**: SAM CLI (build, local invoke/start-api, generate-event, sync, remote invoke), CDK assertions and snapshots, cfn-lint/Guard, API Gateway mocks, test events, environments (aliases, image tags, Amplify branches/previews, Copilot environments, stages), AI-generated tests.
- **Deploying**: Lambda packaging, ECS/EKS/Fargate deployments, CloudFormation/SAM (AutoPublishAlias, DeploymentPreference), CodeDeploy (appspec, hooks, configs), CodePipeline V2 (triggers, approvals, cross-account) + CodeBuild (buildspec, env secrets, VPC, cache, reports), strategies, rollback everywhere, Git tags/branches, AI in CI/CD.
