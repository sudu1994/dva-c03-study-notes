# DVA-C03 Cheat Sheet

> 🔰 <b>N</b>ew <b>t</b>o <b>a</b> <b>te</b>rm? <b>Conc</b>epts <b>li</b>nk <b>t</b>o <b>pla</b>in-<b>Engl</b>ish <b>explan</b>ations <b>i</b>n [<b>t</b>he <b>bas</b>ics <b>pa</b>ge](00-developer-basics.md) <b>o</b>n <b>fir</b>st <b>ment</b>ion, <b>a</b>nd <b>eve</b>ry <b>abbrev</b>iation <b>i</b>s <b>spel</b>led <b>o</b>ut <b>o</b>n <b>fir</b>st <b>u</b>se <b>a</b>nd <b>lis</b>ted <b>i</b>n <b>t</b>he [<b>glos</b>sary](00-abbreviations-glossary.md).

> <b>Rap</b>id <b>rev</b>iew: "<b>i</b>f <b>t</b>he <b>scen</b>ario <b>sa</b>ys <b>X</b> → <b>d</b>o <b>Y</b>," <b>t</b>op <b>tra</b>ps, <b>numb</b>ers. 65 <b>quest</b>ions (50 <b>sco</b>red), 130 <b>minu</b>tes, <b>pa</b>ss 720/1000. <b>Doma</b>ins: <b>Develo</b>pment 30%, <b>Secu</b>rity 26%, <b>Test</b>ing & <b>Deplo</b>yment 22%, <b>Troubles</b>hooting & <b>Optimi</b>zation 22%. DVA-C03 <b>i</b>s <b>i</b>n <b>u</b>se <b>fr</b>om <b>Dece</b>mber 1, 2026.

## Keyword → answer decoder

### Development

| <b>I</b>f <b>t</b>he <b>scen</b>ario <b>sa</b>ys… | <b>D</b>o… |
|---|---|
| <b>Deco</b>uple <b>prod</b>ucer <b>fr</b>om <b>sl</b>ow <b>cons</b>umer | **Simple Queue Service (SQS)** |
| <b>O</b>ne <b>eve</b>nt, <b>ma</b>ny <b>indepe</b>ndent <b>consu</b>mers | **Simple Notification Service (SNS) → SQS fan-out** (<b>o</b>r <b>EventB</b>ridge <b>ru</b>le <b>wi</b>th <b>mult</b>iple <b>targ</b>ets) |
| <b>Rou</b>te <b>eve</b>nts <b>b</b>y <b>cont</b>ent, <b>Soft</b>ware <b>a</b>s <b>a</b> <b>Serv</b>ice (<b>Sa</b>aS) <b>eve</b>nts | **EventBridge** |
| <b>Orde</b>red <b>p</b>er <b>ent</b>ity, <b>exac</b>tly-<b>on</b>ce | **SQS First-In, First-Out [FIFO]** (`MessageGroupId`) |
| <b>Re</b>al-<b>ti</b>me <b>orde</b>red <b>str</b>eam, <b>rep</b>lay, <b>ma</b>ny <b>consu</b>mers | **Kinesis Data Streams** |
| <b>Mul</b>ti-<b>st</b>ep <b>work</b>flow <b>wi</b>th <b>retr</b>ies <b>a</b>nd <b>appr</b>oval | **Step Functions** (<b>ta</b>sk <b>tok</b>en) |
| <b>Mess</b>ages <b>proce</b>ssed <b>twi</b>ce | <b>Rai</b>se **visibility timeout**; [<b>idempo</b>tency](00-developer-basics.md#idempotency) |
| <b>Emp</b>ty <b>rece</b>ives <b>co</b>st | **Long polling** (20 <b>s</b>) |
| <b>Payl</b>oad > SQS <b>lim</b>it | **Simple Storage Service (S3) + pointer** (<b>Exte</b>nded <b>Cli</b>ent) |
| <b>Throt</b>tling <b>err</b>ors <b>fr</b>om <b>Ama</b>zon <b>W</b>eb <b>Serv</b>ices (AWS) <b>Applic</b>ation <b>Progra</b>mming <b>Inter</b>faces (APIs) | **Exponential [backoff](00-developer-basics.md#retries-exponential-backoff-and-jitter) + jitter** ([<b>Soft</b>ware <b>Develo</b>pment <b>K</b>it [SDK]](00-developer-basics.md#sdk-and-cli) <b>retr</b>ies) |
| <b>Rej</b>ect <b>b</b>ad <b>requ</b>ests <b>bef</b>ore <b>Lam</b>bda | **[Application Programming Interface [API]](00-developer-basics.md#api) Gateway request validator + model** |
| <b>Trans</b>form <b>requ</b>est/<b>resp</b>onse | **Mapping templates (VTL)** |
| <b>Brow</b>ser <b>Cro</b>ss-<b>Ori</b>gin <b>Reso</b>urce <b>Shar</b>ing (CORS) <b>err</b>or <b>wi</b>th <b>pro</b>xy <b>integr</b>ation | **Lambda returns CORS headers** |
| <b>P</b>ut API <b>da</b>ta <b>stra</b>ight <b>in</b>to SQS/<b>Dyna</b>moDB | **API Gateway AWS service integration** |
| <b>N</b>o <b>acc</b>ess <b>ke</b>ys <b>o</b>n <b>Elas</b>tic <b>Comp</b>ute <b>Clo</b>ud (EC2) | **Instance profile role** |
| <b>Wa</b>it <b>unt</b>il <b>a</b> <b>reso</b>urce <b>i</b>s <b>rea</b>dy | **SDK waiter** |
| <b>Lar</b>ge <b>li</b>st <b>resu</b>lts | **Paginator** |
| <b>Lam</b>bda <b>eve</b>nt `body` <b>won</b>'t <b>par</b>se | <b>It</b>'s <b>a</b> **JavaScript Object Notation (JSON) string** |
| <b>Kine</b>sis <b>da</b>ta <b>unrea</b>dable | **[Base64](00-developer-basics.md#base64) decode** |
| <b>Thumb</b>nails <b>o</b>n <b>upl</b>oad | **S3 event → Lambda** (<b>diffe</b>rent <b>out</b>put <b>pre</b>fix) |
| [<b>Sp</b>ec-<b>dri</b>ven](00-developer-basics.md#ai-assisted-development) <b>Artif</b>icial <b>Intell</b>igence (AI) <b>co</b>de <b>gener</b>ation | **Kiro specs** (<b>requir</b>ements → <b>des</b>ign → <b>tas</b>ks) |
| <b>Sa</b>me <b>co</b>de <b>f</b>or <b>mult</b>iple <b>mod</b>els + <b>to</b>ol <b>u</b>se | **Bedrock Converse API** |
| <b>Q</b>&A <b>ov</b>er <b>comp</b>any <b>do</b>cs <b>wi</b>th <b>citat</b>ions | **Bedrock Knowledge Bases** |
| <b>Age</b>nt <b>cal</b>ls <b>o</b>ur API <b>wi</b>th <b>confir</b>mation | **Bedrock Agents action group** (<b>us</b>er <b>confir</b>mation/<b>ret</b>urn <b>o</b>f <b>cont</b>rol) |
| <b>Dep</b>loy/<b>r</b>un <b>age</b>nts <b>wi</b>th <b>sess</b>ion <b>isola</b>tion <b>a</b>nd <b>mana</b>ged <b>too</b>ls | **Bedrock AgentCore** |
| <b>Che</b>ap <b>lar</b>ge <b>offl</b>ine <b>infer</b>ence | **Bedrock batch inference** |
| <b>Ja</b>va <b>co</b>ld <b>sta</b>rts | **SnapStart** / <b>provis</b>ioned <b>concur</b>rency |
| <b>Prot</b>ect <b>downs</b>tream DB <b>fr</b>om <b>Lam</b>bda <b>sca</b>le | **Reserved concurrency** / <b>Eve</b>nt <b>Sou</b>rce <b>Mapp</b>ing (ESM) <b>m</b>ax <b>concur</b>rency |
| <b>Sha</b>red <b>libra</b>ries | **Layers** (`/opt`, <b>m</b>ax 5) |
| <b>Lam</b>bda <b>rea</b>ch <b>Relat</b>ional <b>Data</b>base <b>Serv</b>ice (RDS) <b>priva</b>tely | **Virtual Private Cloud (VPC) config + Security Group (SG) + RDS Proxy** |
| VPC <b>Lam</b>bda <b>can</b>'t <b>rea</b>ch S3/<b>inte</b>rnet | **Gateway endpoint** / **Network Address Translation (NAT) gateway** |
| <b>Asy</b>nc <b>fail</b>ure <b>deta</b>ils | **On-failure destination** |
| <b>Poi</b>son <b>rec</b>ord <b>blo</b>cks <b>str</b>eam | **Bisect + max retries/age + on-failure destination** |
| <b>Ret</b>ry <b>on</b>ly <b>fai</b>led SQS <b>mess</b>ages | **Partial batch response** |
| S3/SNS/<b>EventB</b>ridge <b>can</b>'t <b>inv</b>oke <b>Lam</b>bda | **Resource-based policy** |
| <b>Au</b>to-<b>del</b>ete <b>o</b>ld <b>ite</b>ms <b>fr</b>ee | **DynamoDB [Time To Live [TTL]](00-developer-basics.md#ttl-and-data-lifecycle)** (<b>epo</b>ch <b>seco</b>nds) |
| <b>Que</b>ry <b>b</b>y <b>cust</b>omer + <b>da</b>te <b>ran</b>ge | **PK customerId, SK date, Query BETWEEN** |
| <b>Que</b>ry <b>b</b>y <b>anot</b>her <b>attri</b>bute | **Global Secondary Index (GSI)** |
| [<b>Stro</b>ngly <b>consi</b>stent](00-developer-basics.md#read-consistency) <b>alter</b>nate <b>so</b>rt <b>k</b>ey | **Local Secondary Index [LSI]** (<b>a</b>t <b>tab</b>le <b>crea</b>tion) |
| <b>Prev</b>ent <b>overw</b>rite / <b>optim</b>istic <b>lock</b>ing | **Condition expression** (`attribute_not_exists` / <b>vers</b>ion) |
| <b>A</b>ll-<b>o</b>r-<b>noth</b>ing <b>mul</b>ti-<b>it</b>em <b>wri</b>te | **TransactWriteItems** |
| <b>Micros</b>econd <b>Dyna</b>moDB <b>rea</b>ds | **DynamoDB Accelerator (DAX)** |
| <b>Cac</b>he <b>Struc</b>tured <b>Que</b>ry <b>Lang</b>uage (SQL) <b>que</b>ry <b>resu</b>lts | **ElastiCache** [<b>la</b>zy <b>load</b>ing](00-developer-basics.md#caching) |

### Security

| <b>I</b>f <b>t</b>he <b>scen</b>ario <b>sa</b>ys… | <b>D</b>o… |
|---|---|
| <b>A</b>pp <b>si</b>gn-<b>i</b>n <b>wi</b>th <b>Goo</b>gle/[<b>Secu</b>rity <b>Asser</b>tion <b>Mar</b>kup <b>Lang</b>uage [SAML]](00-developer-basics.md#federation-oauth-20-oidc-saml-and-jwt) | **Cognito user pool** (<b>feder</b>ation) |
| <b>Use</b>rs <b>ne</b>ed AWS <b>creden</b>tials (<b>upl</b>oad <b>t</b>o S3) | **Cognito identity pool** |
| <b>Use</b>rs <b>acc</b>ess <b>on</b>ly <b>the</b>ir <b>o</b>wn <b>ite</b>ms | **`dynamodb:LeadingKeys`** / `${cognito-identity.amazonaws.com:sub}` |
| <b>Cus</b>tom <b>tok</b>ens <b>autho</b>rize API | **Lambda authorizer** |
| <b>A</b>dd <b>cla</b>ims <b>t</b>o <b>tok</b>ens | **Pre token generation trigger** |
| CI <b>fr</b>om <b>Git</b>Hub <b>with</b>out <b>ke</b>ys | **OpenID Connect (OIDC) provider + role** |
| <b>Comm</b>and <b>Li</b>ne <b>Inter</b>face (CLI) <b>with</b>out <b>lo</b>ng-<b>te</b>rm <b>ke</b>ys | **Identity Center Single Sign-On (SSO) profile** |
| <b>Ca</b>ll IAM-<b>prote</b>cted API <b>fr</b>om <b>co</b>de | **[Signature Version 4 [SigV4]](00-developer-basics.md#request-signing-sigv4) signing** |
| <b>Tempo</b>rary <b>down</b>load/<b>upl</b>oad <b>li</b>nk | **S3 presigned Uniform Resource Locator [URL]** (<b>expi</b>res <b>wi</b>th <b>sign</b>er's <b>cre</b>ds) |
| <b>Ma</b>ny <b>priv</b>ate <b>fil</b>es <b>v</b>ia <b>Cont</b>ent <b>Deli</b>very <b>Netw</b>ork (CDN) | **CloudFront signed cookies** |
| <b>Lam</b>bda <b>Access</b>Denied | **Execution role** <b>permis</b>sions |
| <b>Elas</b>tic <b>Conta</b>iner <b>Serv</b>ice (ECS) <b>can</b>'t <b>pu</b>ll <b>ima</b>ge/<b>secr</b>ets | **Task execution role** |
| <b>Cro</b>ss-<b>acco</b>unt <b>acc</b>ess | **AssumeRole** (<b>tru</b>st + <b>permi</b>ssion) / <b>reso</b>urce <b>pol</b>icy |
| <b>Thi</b>rd-<b>par</b>ty <b>acc</b>ess | <b>Ro</b>le + **external ID** |
| <b>Encr</b>ypt > 4 KB <b>wi</b>th <b>K</b>ey <b>Manag</b>ement <b>Serv</b>ice (KMS) | **GenerateDataKey** (<b>enve</b>lope) / <b>Encry</b>ption SDK |
| KMS <b>throt</b>tling | **Data key caching**, S3 <b>Buc</b>ket <b>Ke</b>ys |
| <b>Bi</b>nd <b>ciphe</b>rtext <b>t</b>o <b>ten</b>ant | **Encryption context** |
| AWS <b>mu</b>st <b>nev</b>er <b>s</b>ee <b>plain</b>text | **[Client-side](00-developer-basics.md#client-side-vs-server-side-encryption) encryption** |
| <b>Inte</b>rnal <b>mut</b>ual TLS (<b>mT</b>LS) <b>cer</b>ts | **AWS Private Certificate Authority (CA)** |
| <b>Loc</b>al <b>Hyper</b>text <b>Tran</b>sfer <b>Prot</b>ocol <b>Sec</b>ure (HTTPS) <b>d</b>ev <b>ce</b>rt | **Self-signed (OpenSSL)** |
| <b>Avo</b>id <b>Sec</b>ure <b>She</b>ll (SSH) <b>k</b>ey <b>distri</b>bution | **Session Manager / EC2 Instance Connect** |
| [<b>Perso</b>nally <b>Identi</b>fiable <b>Inform</b>ation [PII]](00-developer-basics.md#pii-and-phi-masking-and-redaction) <b>i</b>n <b>lo</b>gs | **CloudWatch Logs data protection** (<b>ma</b>sk) |
| <b>Red</b>act PII <b>o</b>n S3 <b>rea</b>ds | **S3 Object Lambda** |
| <b>Rot</b>ate DB <b>pass</b>word | **Secrets Manager** |
| <b>Che</b>ap <b>con</b>fig | **Parameter Store** |
| <b>On</b>ly <b>appr</b>oved <b>mod</b>els | **Service Control Policy (SCP)/IAM on model Amazon Resource Names (ARNs)** |
| <b>Bedr</b>ock <b>traf</b>fic <b>priv</b>ate | **Interface VPC endpoint (PrivateLink)** |
| <b>Prom</b>pts <b>us</b>ed <b>f</b>or <b>trai</b>ning? | **No** (<b>Bedr</b>ock <b>does</b>n't <b>tra</b>in <b>o</b>n <b>cust</b>omer <b>da</b>ta) |
| <b>Blo</b>ck <b>top</b>ics / <b>ma</b>sk PII / <b>st</b>op [<b>pro</b>mpt <b>injec</b>tion](00-developer-basics.md#prompt-injection) | **Bedrock [Guardrails](00-developer-basics.md#guardrails)** (<b>den</b>ied <b>top</b>ics, <b>sensi</b>tive <b>in</b>fo, <b>pro</b>mpt <b>att</b>ack) |
| <b>Guard</b>rails <b>f</b>or <b>n</b>on-<b>Bedr</b>ock <b>mod</b>el | **ApplyGuardrail API** |
| <b>Requ</b>ire <b>guard</b>rail <b>o</b>n <b>eve</b>ry <b>ca</b>ll | **`bedrock:GuardrailIdentifier` condition** |
| <b>Age</b>nt <b>acti</b>ons <b>on</b>ly <b>wi</b>th <b>use</b>r's <b>permis</b>sions | **AgentCore Identity / Open Authorization (OAuth) propagation** |
| <b>Appr</b>ove <b>bef</b>ore <b>sensi</b>tive <b>age</b>nt <b>act</b>ion | **Human-in-the-loop** (<b>us</b>er <b>confir</b>mation, <b>ret</b>urn <b>o</b>f <b>cont</b>rol, <b>ta</b>sk <b>tok</b>en) |
| <b>Invoc</b>ation <b>lo</b>gs <b>cont</b>ain PII | **Encrypt (KMS) + mask + restrict + retention** |

### Testing and deployment

| <b>I</b>f <b>t</b>he <b>scen</b>ario <b>sa</b>ys… | <b>D</b>o… |
|---|---|
| <b>Sha</b>re <b>inte</b>rnal <b>pack</b>ages, <b>cac</b>he <b>pub</b>lic <b>on</b>es | **CodeArtifact** |
| <b>Prev</b>ent <b>overwr</b>iting <b>ima</b>ge <b>ta</b>gs | **Elastic Container Registry (ECR) tag immutability** |
| <b>O</b>ld <b>ima</b>ges <b>pi</b>le <b>u</b>p | **ECR lifecycle policy** |
| <b>Doc</b>ker <b>bui</b>ld <b>fai</b>ls <b>i</b>n <b>CodeB</b>uild | **Privileged mode** |
| <b>Feat</b>ure <b>fla</b>gs <b>wi</b>th <b>grad</b>ual <b>roll</b>out + <b>roll</b>back | **AppConfig** |
| <b>Diffe</b>rent <b>Lam</b>bda <b>ali</b>as <b>p</b>er API <b>sta</b>ge | **Stage variables** (+ <b>Lam</b>bda <b>permi</b>ssion <b>p</b>er <b>ali</b>as) |
| API <b>chan</b>ges <b>n</b>ot <b>visi</b>ble | **Deploy to stage** |
| <b>Te</b>st <b>n</b>ew API <b>vers</b>ion <b>o</b>n % <b>o</b>f <b>traf</b>fic | **Stage [canary](00-developer-basics.md#deployment-strategies)** |
| <b>Priv</b>ate API | **Private endpoint + resource policy** |
| <b>R</b>un <b>Lam</b>bda <b>loca</b>lly <b>wi</b>th <b>sam</b>ple <b>eve</b>nt | **`sam local generate-event` + `sam local invoke`** |
| <b>Te</b>st <b>Clo</b>ud <b>Develo</b>pment <b>K</b>it (CDK) <b>out</b>put | **CDK assertions / snapshot tests** |
| API <b>bef</b>ore <b>back</b>end <b>exi</b>sts | **API Gateway [mock](00-developer-basics.md#unit-integration-and-end-to-end-tests) integration** |
| <b>Bra</b>nch <b>prev</b>iew [<b>enviro</b>nments](00-developer-basics.md#environments-and-stages) | **Amplify [branches](00-developer-basics.md#source-control)/Pull Request (PR) previews** |
| <b>Sim</b>ple <b>mul</b>ti-<b>enviro</b>nment ECS <b>depl</b>oys | **AWS Copilot** |
| AI-<b>gener</b>ated <b>tes</b>ts <b>i</b>n <b>pipe</b>line | **Kiro (hooks/specs/headless) + CodeBuild reports** |
| <b>Pack</b>age > 250 MB | **Container image** (10 GB) |
| ECS <b>ze</b>ro-<b>down</b>time <b>upd</b>ate | <b>N</b>ew **task definition revision** + <b>roll</b>ing (<b>heal</b>thy %) |
| ECS <b>au</b>to <b>roll</b>back | **Deployment [circuit breaker](00-developer-basics.md#timeouts-and-circuit-breakers)** |
| <b>Grad</b>ual <b>Lam</b>bda <b>dep</b>loy <b>i</b>n <b>Serve</b>rless <b>Applic</b>ation <b>Mod</b>el (SAM) | **AutoPublishAlias + DeploymentPreference** |
| EC2 <b>dep</b>loy <b>o</b>ne <b>inst</b>ance <b>a</b>t <b>a</b> <b>ti</b>me | <b>CodeD</b>eploy **OneAtATime** |
| <b>Valid</b>ation <b>aft</b>er <b>inst</b>all | **ValidateService** <b>ho</b>ok |
| <b>Lam</b>bda <b>can</b>ary <b>wi</b>th <b>p</b>re-<b>traf</b>fic <b>tes</b>ts | **BeforeAllowTraffic** <b>ho</b>ok |
| <b>Mana</b>ger <b>appr</b>oval <b>bef</b>ore <b>pr</b>od | **CodePipeline manual approval** |
| <b>Secr</b>ets <b>i</b>n <b>bui</b>ld | <b>build</b>spec **`env.secrets-manager`** |
| <b>Bui</b>ld <b>reac</b>hes <b>priv</b>ate DB | **CodeBuild VPC config** |
| <b>Beans</b>talk <b>saf</b>est <b>dep</b>loy | **Immutable** (<b>o</b>r <b>bl</b>ue/<b>gre</b>en <b>Canon</b>ical <b>Na</b>me [CNAME] <b>sw</b>ap) |
| <b>Beans</b>talk <b>fu</b>ll <b>capa</b>city, <b>che</b>ap | **Rolling with additional batch** |
| <b>Ro</b>ll <b>ba</b>ck <b>o</b>n <b>ala</b>rm | **CodeDeploy alarms / CloudFormation rollback triggers** |
| <b>Inst</b>ant <b>Lam</b>bda <b>roll</b>back | **Move alias** |
| <b>Dep</b>loy <b>o</b>n <b>rele</b>ase <b>t</b>ag | **CodePipeline V2 tag trigger** |
| AI <b>rev</b>iew <b>bef</b>ore <b>appr</b>oval | **Bedrock/Kiro step + manual approval** |

### Troubleshooting and optimization

| <b>I</b>f <b>t</b>he <b>scen</b>ario <b>sa</b>ys… | <b>D</b>o… |
|---|---|
| API <b>Gate</b>way 502 | <b>F</b>ix **Lambda proxy response format** |
| API <b>Gate</b>way 504 | > **29 s** → <b>asy</b>nc/<b>str</b>eam/<b>opti</b>mize |
| <b>Co</b>ld <b>sta</b>rt <b>del</b>ays | `Init Duration` → **provisioned concurrency/SnapStart** |
| <b>O</b>ut <b>o</b>f <b>mem</b>ory / <b>sl</b>ow <b>Cent</b>ral <b>Proce</b>ssing <b>Un</b>it (CPU) | **Raise memory** |
| <b>Fi</b>nd <b>sl</b>ow <b>downs</b>tream <b>ca</b>ll | **X-Ray** <b>subseg</b>ments |
| <b>Fil</b>ter [<b>tra</b>ces](00-developer-basics.md#logs-metrics-and-traces) <b>b</b>y <b>cust</b>omer | **Annotations** |
| <b>Fi</b>nd <b>requ</b>ests <b>acr</b>oss <b>serv</b>ices | **Logs Insights + [correlation ID](00-developer-basics.md#correlation-id)** |
| [p99](00-developer-basics.md#percentiles) <b>late</b>ncy <b>fr</b>om <b>lo</b>gs | `stats pct(@duration, 99)` |
| <b>Sta</b>ck <b>fai</b>led | **Stack events** (<b>fir</b>st <b>fail</b>ure) |
| ECS `CannotPullContainerError` | **Execution role / ECR endpoints / NAT** |
| <b>Busi</b>ness <b>metr</b>ics <b>fr</b>om <b>Lam</b>bda <b>chea</b>ply | **Embedded Metric Format (EMF)** |
| <b>Ale</b>rt <b>ne</b>ar <b>Lam</b>bda <b>concur</b>rency <b>lim</b>it | **Service Quotas usage alarm** |
| <b>Not</b>ify <b>o</b>n <b>pipe</b>line <b>fail</b>ure | **Notification rule / EventBridge → SNS** |
| <b>Lam</b>bda <b>tra</b>ces <b>miss</b>ing | **Active tracing + X-Ray permissions** |
| <b>Po</b>ds <b>g</b>et <b>traf</b>fic <b>t</b>oo <b>ear</b>ly | **Readiness probe** |
| <b>Concur</b>rency <b>nee</b>ded | **Requests per second (rps) × duration** |
| DB <b>connec</b>tions <b>exhau</b>sted | **RDS Proxy** |
| <b>Co</b>st-<b>opti</b>mal <b>mem</b>ory | **Lambda Power Tuning / Compute Optimizer** |
| <b>H</b>ot <b>Dyna</b>moDB <b>ke</b>ys | **Contributor Insights** |
| <b>P</b>er-<b>func</b>tion CPU/<b>mem</b>ory | **Lambda Insights** |
| AI <b>ro</b>ot-<b>cau</b>se <b>anal</b>ysis | **Kiro / DevOps Agent / CloudWatch investigations** |

## Top traps

1. <b>Pro</b>xy <b>resp</b>onse <b>nee</b>ds `statusCode` + <b>str</b>ing `body`.
2. API <b>Gate</b>way **29 s** / **10 MB**; <b>Lam</b>bda <b>sy</b>nc **6 MB**, <b>asy</b>nc **1 MB**.
3. **Destinations** <b>car</b>ry <b>cont</b>ext; <b>De</b>ad-<b>Let</b>ter <b>Que</b>ue (DLQ) <b>does</b>n't.
4. <b>Stre</b>ams <b>blo</b>ck <b>o</b>n <b>err</b>ors → <b>bis</b>ect/<b>m</b>ax <b>a</b>ge/<b>destin</b>ation.
5. VPC <b>Lam</b>bda <b>h</b>as <b>n</b>o <b>inte</b>rnet <b>with</b>out **NAT**.
6. **API keys** <b>are</b>n't <b>au</b>th.
7. **Execution role** <b>v</b>s **resource policy**; **task role** <b>v</b>s **task execution role**.
8. <b>Presi</b>gned URL <b>di</b>es <b>wh</b>en <b>sign</b>er's **[temporary credentials](00-developer-basics.md#temporary-credentials)** <b>exp</b>ire.
9. KMS **Encrypt ≤ 4 KB**.
10. **Scan** <b>rea</b>ds <b>every</b>thing; **FilterExpression** <b>does</b>n't <b>c</b>ut <b>Re</b>ad <b>Capa</b>city <b>Un</b>it (RCU).
11. **GSI** <b>even</b>tual <b>on</b>ly; **LSI** <b>a</b>t <b>crea</b>tion <b>on</b>ly.
12. RCU 4 KB / <b>Wri</b>te <b>Capa</b>city <b>Un</b>it (WCU) 1 KB; <b>transa</b>ctions ×2; <b>even</b>tual ×0.5.
13. [<b>Represen</b>tational <b>Sta</b>te <b>Tran</b>sfer [REST]](00-developer-basics.md#rest-and-http-methods) API <b>chan</b>ges <b>ne</b>ed **deployment**.
14. **appspec** (<b>CodeD</b>eploy) ≠ **buildspec** (<b>CodeB</b>uild).
15. <b>Nat</b>ive <b>de</b>ps <b>mu</b>st <b>mat</b>ch **Lambda arch/Operating System (OS)**.
16. <b>Annota</b>tions <b>inde</b>xed; <b>meta</b>data <b>n</b>ot.
17. <b>La</b>zy <b>load</b>ing → <b>sta</b>le; <b>wri</b>te-<b>thro</b>ugh → <b>fre</b>sh.
18. <b>Bedr</b>ock <b>does</b>n't <b>tra</b>in <b>o</b>n <b>yo</b>ur <b>da</b>ta; <b>u</b>se **PrivateLink** <b>f</b>or <b>priv</b>ate <b>acc</b>ess.

## Numbers

| <b>It</b>em | <b>Val</b>ue |
|---|---|
| <b>Lam</b>bda <b>mem</b>ory / <b>time</b>out / `/tmp` | 128 MB–10 GB / 15 <b>m</b>in / 512 MB–10 GB |
| <b>Lam</b>bda <b>e</b>nv <b>va</b>rs | 4 KB <b>tot</b>al |
| <b>Lam</b>bda <b>pack</b>age | 50 MB <b>zip</b>ped <b>dir</b>ect, 250 MB <b>unzi</b>pped, <b>ima</b>ge 10 GB, 5 <b>lay</b>ers |
| <b>Lam</b>bda <b>payl</b>oad | <b>sy</b>nc 6 MB, <b>asy</b>nc 1 MB (<b>form</b>erly 256 KB) |
| <b>Lam</b>bda <b>concur</b>rency | 1,000 <b>defa</b>ult <b>p</b>er <b>Reg</b>ion; +1,000 <b>p</b>er 10 <b>s</b> <b>p</b>er <b>func</b>tion |
| <b>Lam</b>bda <b>asy</b>nc <b>retr</b>ies / <b>eve</b>nt <b>a</b>ge | 2 / <b>u</b>p <b>t</b>o 6 <b>h</b> |
| API <b>Gate</b>way | 29 <b>s</b>, 10 MB, 10,000 <b>r</b>ps (5,000 <b>bur</b>st), <b>cac</b>he TTL <b>defa</b>ult 300 <b>s</b> <b>m</b>ax 3,600 |
| SQS | 1 <b>M</b>iB (<b>form</b>erly 256 KB), <b>visib</b>ility 30 <b>s</b> <b>defa</b>ult / 12 <b>h</b> <b>m</b>ax, <b>reten</b>tion 4 <b>da</b>ys / 14 <b>m</b>ax, <b>lo</b>ng <b>po</b>ll 20 <b>s</b>, <b>del</b>ay 15 <b>m</b>in, <b>bat</b>ch 10 |
| SQS FIFO | 300 <b>Transa</b>ctions <b>P</b>er <b>Sec</b>ond [TPS] (3,000 <b>batc</b>hed); <b>ded</b>up <b>win</b>dow 5 <b>m</b>in |
| SNS | <b>mess</b>age 256 KB; <b>Publis</b>hBatch 10 |
| <b>EventB</b>ridge <b>PutEv</b>ents | 10 <b>entr</b>ies <b>p</b>er <b>ca</b>ll |
| <b>Kine</b>sis <b>sha</b>rd | 1 MB/<b>s</b> <b>o</b>r 1,000 <b>r</b>ec/<b>s</b> <b>i</b>n; 2 MB/<b>s</b> <b>o</b>ut; 5 <b>rea</b>ds/<b>s</b>; <b>reten</b>tion 24 <b>h</b>–365 <b>da</b>ys; <b>PutRe</b>cords 500 |
| <b>Dyna</b>moDB | <b>it</b>em 400 KB; <b>Que</b>ry/<b>Sc</b>an 1 MB <b>pag</b>es; <b>Batc</b>hGet 100/16 MB; <b>Batch</b>Write 25; <b>transa</b>ctions 100 <b>ite</b>ms; 20 <b>Glo</b>bal <b>Secon</b>dary <b>Inde</b>xes (GSIs), 5 <b>Loc</b>al <b>Secon</b>dary <b>Inde</b>xes (LSIs); <b>parti</b>tion 3,000 RCU/1,000 WCU |
| <b>St</b>ep <b>Funct</b>ions | <b>Stan</b>dard 1 <b>ye</b>ar; <b>Expr</b>ess 5 <b>m</b>in; <b>payl</b>oad 256 KB |
| <b>Cogn</b>ito <b>tok</b>ens | ID/<b>acc</b>ess 1 <b>h</b> <b>defa</b>ult; <b>refr</b>esh 30 <b>da</b>ys <b>defa</b>ult |
| <b>Secu</b>rity <b>Tok</b>en <b>Serv</b>ice (STS) | 15 <b>m</b>in–12 <b>h</b>; <b>ro</b>le <b>chai</b>ning 1 <b>h</b> |
| S3 <b>presi</b>gned URL | <b>u</b>p <b>t</b>o 7 <b>da</b>ys (SigV4, <b>Iden</b>tity <b>a</b>nd <b>Acc</b>ess <b>Manag</b>ement [IAM] <b>us</b>er) |
| KMS | <b>Encr</b>ypt 4 KB |
| <b>Param</b>eter <b>Sto</b>re / <b>Secr</b>ets <b>Mana</b>ger | 4 KB (8 KB <b>a</b>dv) / 64 KB |
| <b>CodeB</b>uild <b>time</b>out | 60 <b>m</b>in <b>defa</b>ult, <b>u</b>p <b>t</b>o 36 <b>h</b> |
| <b>Lam</b>bda <b>co</b>de <b>stor</b>age | 75 GB <b>p</b>er <b>Reg</b>ion |
