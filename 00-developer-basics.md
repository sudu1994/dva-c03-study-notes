# Developer Basics (the 101 layer)

> <b>Pla</b>in-<b>Engl</b>ish <b>explan</b>ations <b>o</b>f <b>t</b>he <b>soft</b>ware, <b>Applic</b>ation <b>Progra</b>mming <b>Inter</b>face (API), <b>da</b>ta, <b>secu</b>rity <b>a</b>nd <b>Artif</b>icial <b>Intell</b>igence (AI) <b>ide</b>as <b>t</b>he DVA-C03 <b>not</b>es <b>re</b>ly <b>o</b>n. <b>Clo</b>ud <b>Practi</b>tioner <b>bas</b>ics <b>a</b>re <b>le</b>ft <b>o</b>ut. <b>T</b>he <b>fir</b>st <b>ment</b>ion <b>o</b>f <b>ea</b>ch <b>conc</b>ept <b>i</b>n <b>t</b>he <b>dom</b>ain <b>fil</b>es <b>lin</b>ks <b>ba</b>ck <b>he</b>re. <b>Abbrevi</b>ations <b>a</b>re <b>i</b>n [00-abbreviations-glossary.md](00-abbreviations-glossary.md).

## How DVA questions think

DVA <b>as</b>ks *<b>wh</b>at <b>co</b>de, <b>configu</b>ration <b>o</b>r <b>serv</b>ice <b>feat</b>ure <b>a</b> <b>devel</b>oper <b>us</b>es* <b>t</b>o <b>ma</b>ke <b>a</b>n <b>a</b>pp <b>wo</b>rk, <b>sec</b>ure, <b>deplo</b>yable <b>a</b>nd <b>fa</b>st: <b>whi</b>ch <b>Soft</b>ware <b>Develo</b>pment <b>K</b>it (SDK) <b>ca</b>ll, <b>whi</b>ch <b>Lam</b>bda <b>sett</b>ing, <b>whi</b>ch <b>Dyna</b>moDB <b>k</b>ey <b>des</b>ign, <b>whi</b>ch <b>hea</b>der, <b>whi</b>ch <b>deplo</b>yment <b>configu</b>ration, <b>whi</b>ch <b>l</b>og <b>tel</b>ls <b>y</b>ou <b>w</b>hy <b>i</b>t <b>bro</b>ke.

| <b>Const</b>raint <b>wo</b>rd | <b>Usua</b>lly <b>mea</b>ns |
|---|---|
| "<b>lea</b>st <b>operat</b>ional <b>over</b>head" / "<b>mini</b>mal <b>co</b>de <b>chan</b>ges" | <b>Mana</b>ged <b>feat</b>ure/<b>configu</b>ration (<b>Lam</b>bda <b>destin</b>ations, <b>Sim</b>ple <b>Que</b>ue <b>Serv</b>ice [SQS] <b>De</b>ad-<b>Let</b>ter <b>Que</b>ue [DLQ], API <b>Gate</b>way <b>valid</b>ation) <b>ov</b>er <b>cus</b>tom <b>co</b>de |
| "<b>mo</b>st <b>sec</b>ure" | <b>Tempo</b>rary <b>creden</b>tials (<b>rol</b>es), <b>lea</b>st <b>privi</b>lege, <b>n</b>o <b>secr</b>ets <b>i</b>n <b>co</b>de, <b>encry</b>ption <b>wi</b>th <b>K</b>ey <b>Manag</b>ement <b>Serv</b>ice (KMS) |
| "<b>deco</b>uple" / "<b>asynch</b>ronous" | SQS, <b>Sim</b>ple <b>Notifi</b>cation <b>Serv</b>ice (SNS), <b>EventB</b>ridge, <b>St</b>ep <b>Funct</b>ions |
| "<b>re</b>al <b>ti</b>me" | <b>Kine</b>sis/<b>stre</b>ams, <b>WebSo</b>ckets, <b>Dyna</b>moDB <b>Stre</b>ams |
| "<b>with</b>out <b>impac</b>ting <b>produ</b>ction" | <b>Alia</b>ses, <b>sta</b>ges, <b>can</b>ary, <b>sepa</b>rate <b>enviro</b>nments |

---

## Software and API concepts

### API
<b>A</b>n **Application Programming Interface**: <b>a</b> <b>defi</b>ned <b>w</b>ay <b>f</b>or <b>prog</b>rams <b>t</b>o <b>ca</b>ll <b>ea</b>ch <b>oth</b>er. <b>O</b>n <b>Ama</b>zon <b>W</b>eb <b>Serv</b>ices (AWS), <b>eve</b>ry <b>act</b>ion (<b>cre</b>ate <b>a</b> <b>buc</b>ket, <b>p</b>ut <b>a</b>n <b>it</b>em) <b>i</b>s <b>a</b>n API <b>ca</b>ll; <b>t</b>he <b>cons</b>ole, <b>Comm</b>and <b>Li</b>ne <b>Inter</b>face (CLI) <b>a</b>nd <b>Soft</b>ware <b>Develo</b>pment <b>Ki</b>ts (SDKs) <b>a</b>ll <b>ca</b>ll <b>t</b>he <b>sa</b>me <b>Applic</b>ation <b>Progra</b>mming <b>Inter</b>faces (APIs).

### REST and HTTP methods
**Representational State Transfer (REST)** APIs <b>u</b>se <b>Hyper</b>text <b>Tran</b>sfer <b>Prot</b>ocol (HTTP): `GET` (<b>re</b>ad), `POST` (<b>cre</b>ate), `PUT` (<b>repl</b>ace), `PATCH` (<b>part</b>ial <b>upd</b>ate), `DELETE`. <b>Resou</b>rces <b>a</b>re <b>addre</b>ssed <b>b</b>y URLs (`/orders/123`). <b>Respo</b>nses <b>car</b>ry **status codes** (200 OK, 201 <b>Crea</b>ted, 400 <b>b</b>ad <b>requ</b>est, 401 <b>unauthen</b>ticated, 403 <b>forbi</b>dden, 404 <b>n</b>ot <b>fou</b>nd, 409 <b>conf</b>lict, 429 <b>throt</b>tled, 500 <b>ser</b>ver <b>err</b>or, 502 <b>b</b>ad <b>gate</b>way, 503 <b>unavai</b>lable, 504 <b>time</b>out).

### GraphQL and WebSocket
**Graph Query Language (GraphQL)**: <b>clie</b>nts <b>a</b>sk <b>f</b>or <b>exac</b>tly <b>t</b>he <b>fie</b>lds <b>th</b>ey <b>ne</b>ed <b>i</b>n <b>o</b>ne <b>que</b>ry (AWS <b>AppS</b>ync). **WebSocket**: <b>a</b> <b>persi</b>stent <b>t</b>wo-<b>w</b>ay <b>conne</b>ction <b>s</b>o <b>t</b>he <b>ser</b>ver <b>c</b>an <b>pu</b>sh <b>mess</b>ages (<b>ch</b>at, <b>li</b>ve <b>upda</b>tes).

### SDK and CLI
<b>A</b>n **SDK** (<b>soft</b>ware <b>develo</b>pment <b>k</b>it) <b>i</b>s <b>a</b> <b>libr</b>ary <b>f</b>or <b>a</b> <b>progra</b>mming <b>lang</b>uage (boto3 <b>f</b>or <b>Pyt</b>hon, AWS SDK <b>f</b>or <b>JavaS</b>cript v3, <b>Ja</b>va v2) <b>th</b>at <b>sig</b>ns <b>requ</b>ests, <b>retr</b>ies, <b>pagin</b>ates <b>a</b>nd <b>hand</b>les <b>err</b>ors. <b>T</b>he **AWS CLI** <b>i</b>s <b>a</b> <b>comm</b>and-<b>li</b>ne <b>to</b>ol <b>ov</b>er <b>t</b>he <b>sa</b>me APIs.

### Request signing (SigV4)
<b>Eve</b>ry AWS API <b>requ</b>est <b>i</b>s <b>sig</b>ned <b>wi</b>th **Signature Version 4** <b>usi</b>ng <b>t</b>he <b>call</b>er's <b>sec</b>ret <b>k</b>ey, <b>prov</b>ing <b>iden</b>tity <b>a</b>nd <b>integ</b>rity. SDKs/CLI <b>d</b>o <b>i</b>t <b>automat</b>ically; <b>y</b>ou <b>on</b>ly <b>d</b>o <b>i</b>t <b>b</b>y <b>ha</b>nd <b>f</b>or <b>r</b>aw HTTP <b>cal</b>ls (<b>e</b>.g., <b>t</b>o <b>a</b>n IAM-<b>autho</b>rized API <b>Gate</b>way).

### Pagination
APIs <b>ret</b>urn <b>lar</b>ge <b>res</b>ult <b>se</b>ts <b>i</b>n **pages** <b>wi</b>th <b>a</b> `NextToken`/`LastEvaluatedKey`; <b>ke</b>ep <b>call</b>ing <b>unt</b>il <b>n</b>o <b>tok</b>en <b>i</b>s <b>retu</b>rned (SDK **paginators** <b>d</b>o <b>th</b>is).

### Synchronous vs asynchronous
**Synchronous**: <b>t</b>he <b>cal</b>ler <b>wai</b>ts <b>f</b>or <b>t</b>he <b>res</b>ult (API <b>Gate</b>way → <b>Lam</b>bda). **Asynchronous**: <b>t</b>he <b>cal</b>ler <b>han</b>ds <b>wo</b>rk <b>t</b>o <b>a</b> <b>que</b>ue/<b>eve</b>nt <b>sys</b>tem <b>a</b>nd <b>conti</b>nues; <b>resu</b>lts <b>co</b>me <b>lat</b>er (<b>Sim</b>ple <b>Stor</b>age <b>Serv</b>ice [S3] <b>eve</b>nt → <b>Lam</b>bda, SQS).

### Tightly vs loosely coupled
**Tightly coupled** <b>compo</b>nents <b>ca</b>ll <b>ea</b>ch <b>oth</b>er <b>dire</b>ctly <b>a</b>nd <b>fa</b>il <b>toge</b>ther. **Loosely coupled** <b>compo</b>nents <b>ta</b>lk <b>thro</b>ugh <b>que</b>ues/<b>top</b>ics/<b>eve</b>nts, <b>s</b>o <b>o</b>ne <b>c</b>an <b>b</b>e <b>sl</b>ow <b>o</b>r <b>do</b>wn <b>with</b>out <b>brea</b>king <b>t</b>he <b>oth</b>er.

### Event-driven architecture
<b>Compo</b>nents <b>rea</b>ct <b>t</b>o **events** ("<b>ord</b>er <b>pla</b>ced") <b>publi</b>shed <b>b</b>y <b>oth</b>ers <b>inst</b>ead <b>o</b>f <b>bei</b>ng <b>cal</b>led <b>dire</b>ctly.

### Idempotency
<b>Doi</b>ng <b>t</b>he <b>sa</b>me <b>requ</b>est <b>twi</b>ce <b>h</b>as <b>t</b>he <b>sa</b>me <b>eff</b>ect <b>a</b>s <b>on</b>ce. <b>Nee</b>ded <b>beca</b>use <b>retr</b>ies <b>a</b>nd <b>a</b>t-<b>lea</b>st-<b>on</b>ce <b>deli</b>very <b>cau</b>se <b>dupli</b>cates. <b>Imple</b>ment <b>wi</b>th <b>a</b>n **idempotency key** <b>sto</b>red <b>i</b>n <b>Dyna</b>moDB (<b>e</b>.g., <b>Power</b>tools <b>f</b>or AWS <b>Lam</b>bda <b>idempo</b>tency <b>util</b>ity) <b>o</b>r <b>condit</b>ional <b>wri</b>tes.

### Retries, exponential backoff and jitter
<b>O</b>n <b>throt</b>tling (429/`ThrottlingException`) <b>o</b>r <b>trans</b>ient 5xx <b>err</b>ors, <b>ret</b>ry <b>aft</b>er <b>wait</b>ing 1, 2, 4, 8 <b>s</b>…(**exponential backoff**) <b>pl</b>us <b>rando</b>mness (**jitter**). SDKs <b>d</b>o <b>th</b>is <b>automat</b>ically (<b>config</b>urable <b>m</b>ax <b>atte</b>mpts, <b>ret</b>ry <b>mo</b>de `standard`/`adaptive`). <b>Don</b>'t <b>ret</b>ry 4xx <b>valid</b>ation <b>err</b>ors.

### Timeouts and circuit breakers
<b>Alw</b>ays <b>s</b>et <b>time</b>outs <b>o</b>n <b>outb</b>ound <b>cal</b>ls. <b>A</b> **circuit breaker** <b>sto</b>ps <b>call</b>ing <b>a</b> <b>fail</b>ing <b>depen</b>dency <b>f</b>or <b>a</b> <b>whi</b>le <b>t</b>o <b>l</b>et <b>i</b>t <b>reco</b>ver.

### Graceful degradation
<b>Wh</b>en <b>a</b> <b>depen</b>dency <b>fai</b>ls, <b>ke</b>ep <b>serv</b>ing <b>a</b> <b>redu</b>ced <b>exper</b>ience (<b>cac</b>hed <b>da</b>ta, <b>defa</b>ult <b>resp</b>onse) <b>inst</b>ead <b>o</b>f <b>fail</b>ing <b>compl</b>etely.

### Serialization
<b>Conve</b>rting <b>obje</b>cts <b>t</b>o <b>a</b> <b>transmi</b>ttable <b>for</b>mat (**serialize**: <b>obj</b>ect → <b>JavaS</b>cript <b>Obj</b>ect <b>Nota</b>tion [JSON] <b>str</b>ing/<b>byt</b>es) <b>a</b>nd <b>ba</b>ck (**deserialize**). AWS APIs <b>u</b>se JSON; <b>Lam</b>bda <b>eve</b>nts <b>arr</b>ive <b>a</b>s JSON; <b>Kine</b>sis/SQS <b>payl</b>oads <b>a</b>re <b>stri</b>ngs/<b>byt</b>es (<b>oft</b>en JSON <b>o</b>r **base64**-<b>enco</b>ded <b>bin</b>ary); <b>Dyna</b>moDB <b>h</b>as <b>i</b>ts <b>o</b>wn <b>typ</b>ed JSON (`{"S": "text"}`, `{"N": "5"}`) <b>th</b>at SDK <b>docu</b>ment <b>clie</b>nts <b>conv</b>ert <b>f</b>or <b>y</b>ou. **Avro/Protobuf** <b>a</b>re <b>comp</b>act <b>bin</b>ary <b>form</b>ats <b>wi</b>th <b>sche</b>mas.

### Base64
<b>Enco</b>ding <b>bin</b>ary <b>da</b>ta <b>a</b>s <b>te</b>xt. <b>Kine</b>sis <b>reco</b>rds <b>deliv</b>ered <b>t</b>o <b>Lam</b>bda <b>a</b>nd API <b>Gate</b>way <b>bin</b>ary <b>payl</b>oads <b>a</b>re base64-<b>enco</b>ded; <b>dec</b>ode <b>bef</b>ore <b>u</b>se.

### Environment variables and configuration
<b>Val</b>ues <b>giv</b>en <b>t</b>o <b>t</b>he <b>a</b>pp <b>a</b>t <b>runt</b>ime (<b>tab</b>le <b>na</b>me, <b>sta</b>ge, <b>feat</b>ure <b>fla</b>gs) <b>inst</b>ead <b>o</b>f <b>ha</b>rd-<b>cod</b>ing. <b>Secr</b>ets <b>g</b>o <b>i</b>n <b>Secr</b>ets <b>Mana</b>ger/<b>Param</b>eter <b>Sto</b>re, <b>n</b>ot <b>pla</b>in <b>e</b>nv <b>va</b>rs.

### Dependency management and packaging
<b>Yo</b>ur <b>co</b>de <b>pl</b>us <b>i</b>ts <b>libra</b>ries (`requirements.txt`, `package.json`, `pom.xml`) <b>bund</b>led <b>in</b>to <b>a</b> **deployment artifact** (<b>z</b>ip, JAR, <b>conta</b>iner <b>ima</b>ge). <b>Lo</b>ck <b>vers</b>ions <b>f</b>or <b>repea</b>table <b>bui</b>lds.

### Containers and images
<b>A</b>n **image** <b>pack</b>ages <b>a</b>pp + <b>runt</b>ime + <b>libra</b>ries (<b>bui</b>lt <b>fr</b>om <b>a</b> **Dockerfile**); <b>a</b> **container** <b>i</b>s <b>a</b> <b>runn</b>ing <b>ima</b>ge. <b>Ima</b>ges <b>li</b>ve <b>i</b>n <b>regis</b>tries (ECR) <b>a</b>nd <b>a</b>re <b>ident</b>ified <b>b</b>y **tags** (`v1.4.2`, `latest`) <b>a</b>nd <b>immut</b>able **digests** (`sha256:…`).

---

## Data concepts

### Relational vs NoSQL
**Relational** (<b>Relat</b>ional <b>Data</b>base <b>Serv</b>ice [RDS], <b>Aur</b>ora): <b>tab</b>les, <b>Struc</b>tured <b>Que</b>ry <b>Lang</b>uage (SQL), <b>joi</b>ns, <b>transa</b>ctions. **Not only SQL (NoSQL) key-value/document** (<b>Dyna</b>moDB): <b>acc</b>ess <b>b</b>y <b>k</b>ey <b>a</b>t <b>a</b>ny <b>sca</b>le, <b>flex</b>ible <b>attri</b>butes, <b>des</b>ign <b>tab</b>les <b>aro</b>und <b>quer</b>ies.

### Partition key and cardinality
<b>Dyna</b>moDB <b>spre</b>ads <b>da</b>ta <b>acr</b>oss <b>parti</b>tions <b>b</b>y <b>hash</b>ing <b>t</b>he **partition key**. **High cardinality** (<b>ma</b>ny <b>dist</b>inct <b>val</b>ues <b>li</b>ke `userId`) <b>spre</b>ads <b>lo</b>ad; **low cardinality** (<b>li</b>ke `status`) <b>crea</b>tes **hot partitions** <b>a</b>nd <b>throt</b>tling.

### Read consistency
**Eventually consistent** <b>rea</b>ds <b>m</b>ay <b>ret</b>urn <b>slig</b>htly <b>sta</b>le <b>da</b>ta (<b>chea</b>per, <b>defa</b>ult). **Strongly consistent** <b>rea</b>ds <b>ret</b>urn <b>t</b>he <b>lat</b>est <b>commi</b>tted <b>wri</b>te (<b>co</b>st <b>dou</b>ble, <b>n</b>ot <b>avail</b>able <b>o</b>n <b>Glo</b>bal <b>Secon</b>dary <b>Inde</b>xes [GSIs]).

### Caching
<b>Stor</b>ing <b>frequ</b>ently <b>re</b>ad <b>da</b>ta <b>i</b>n <b>fa</b>st <b>mem</b>ory <b>o</b>r <b>a</b>t <b>t</b>he <b>ed</b>ge. <b>Strat</b>egies: **lazy loading / cache-aside** (<b>fi</b>ll <b>o</b>n <b>mi</b>ss), **write-through** (<b>upd</b>ate <b>cac</b>he <b>o</b>n <b>eve</b>ry <b>wri</b>te), **Time To Live (TTL)** <b>exp</b>iry. <b>Tra</b>de-<b>o</b>ff: <b>spe</b>ed <b>v</b>s <b>stale</b>ness.

### TTL and data lifecycle
**Time to live**: <b>autom</b>atic <b>exp</b>iry <b>o</b>f <b>da</b>ta (<b>Dyna</b>moDB TTL, <b>cac</b>he TTL). **Lifecycle policies** <b>mo</b>ve <b>o</b>r <b>del</b>ete <b>da</b>ta <b>a</b>s <b>i</b>t <b>ag</b>es (S3 <b>Lifec</b>ycle, <b>Cloud</b>Watch <b>Lo</b>gs <b>reten</b>tion).

### Streaming data
<b>Conti</b>nuous <b>flo</b>ws <b>o</b>f <b>reco</b>rds (<b>cli</b>cks, <b>I</b>oT) <b>proce</b>ssed <b>i</b>n <b>ord</b>er <b>p</b>er <b>k</b>ey, <b>wi</b>th <b>reten</b>tion <b>s</b>o <b>consu</b>mers <b>c</b>an <b>rep</b>lay (<b>Kine</b>sis <b>Da</b>ta <b>Stre</b>ams, <b>Dyna</b>moDB <b>Stre</b>ams, <b>Kaf</b>ka).

---

## Security concepts

### Authentication vs authorization
**Authentication**: <b>w</b>ho <b>y</b>ou <b>a</b>re (<b>pass</b>word, <b>tok</b>en, <b>ke</b>ys). **Authorization**: <b>wh</b>at <b>y</b>ou <b>c</b>an <b>d</b>o (<b>Iden</b>tity <b>a</b>nd <b>Acc</b>ess <b>Manag</b>ement [IAM] <b>poli</b>cies, <b>sco</b>pes, <b>gro</b>ups).

### Principal, policy and least privilege
<b>A</b> **principal** <b>i</b>s <b>w</b>ho <b>cal</b>ls (<b>us</b>er, <b>ro</b>le, <b>serv</b>ice). <b>A</b> **policy** (JSON) <b>all</b>ows/<b>den</b>ies <b>acti</b>ons <b>o</b>n <b>resou</b>rces. **Least privilege** = <b>on</b>ly <b>wha</b>t's <b>nee</b>ded.

### Temporary credentials
<b>Sho</b>rt-<b>liv</b>ed <b>acc</b>ess <b>k</b>ey + <b>sec</b>ret + **session token** <b>fr</b>om <b>Secu</b>rity <b>Tok</b>en <b>Serv</b>ice [STS] (<b>rol</b>es). <b>Saf</b>er <b>th</b>an <b>lo</b>ng-<b>te</b>rm <b>acc</b>ess <b>ke</b>ys; SDKs <b>lo</b>ad <b>th</b>em <b>automat</b>ically <b>fr</b>om <b>t</b>he <b>enviro</b>nment, <b>inst</b>ance <b>meta</b>data, <b>conta</b>iner <b>creden</b>tials <b>o</b>r <b>Sin</b>gle <b>Si</b>gn-<b>O</b>n (SSO).

### Federation, OAuth 2.0, OIDC, SAML and JWT
**Federation**: <b>u</b>se <b>a</b>n <b>exis</b>ting <b>iden</b>tity <b>prov</b>ider (<b>Goo</b>gle, <b>corpo</b>rate <b>direc</b>tory). **Open Authorization (OAuth) 2.0**: <b>authori</b>zation <b>frame</b>work <b>issu</b>ing **access tokens** <b>wi</b>th **scopes**. **OpenID Connect (OIDC)**: <b>iden</b>tity <b>lay</b>er <b>o</b>n <b>OAu</b>th 2.0 <b>issu</b>ing **ID tokens**. **Security Assertion Markup Language (SAML) 2.0**: XML-<b>bas</b>ed <b>enter</b>prise <b>feder</b>ation. **JWT (JSON Web Token)**: <b>sig</b>ned <b>tok</b>en `header.payload.signature` <b>wi</b>th <b>cla</b>ims (`sub`, `iss`, `aud`, `exp`); <b>ver</b>ify <b>signa</b>ture <b>wi</b>th <b>t</b>he <b>issu</b>er's <b>pub</b>lic <b>ke</b>ys (JWKS) <b>a</b>nd <b>che</b>ck <b>exp</b>iry/<b>audi</b>ence.

### MFA
<b>Mul</b>ti-<b>fac</b>tor <b>authent</b>ication: <b>somet</b>hing <b>y</b>ou <b>kn</b>ow + <b>somet</b>hing <b>y</b>ou <b>ha</b>ve (<b>authent</b>icator <b>co</b>de, <b>pass</b>key).

### Encryption at rest and in transit, symmetric and asymmetric
**At rest**: <b>sto</b>red <b>da</b>ta <b>encry</b>pted (KMS <b>ke</b>ys). **In transit**: TLS/<b>Hyper</b>text <b>Tran</b>sfer <b>Prot</b>ocol <b>Sec</b>ure (HTTPS). **Symmetric**: <b>sa</b>me <b>k</b>ey <b>encr</b>ypts/<b>decr</b>ypts (AES). **Asymmetric**: <b>pub</b>lic/<b>priv</b>ate <b>k</b>ey <b>pa</b>ir (<b>Riv</b>est–<b>Sha</b>mir–<b>Adle</b>man [RSA]/<b>Elli</b>ptic <b>Cur</b>ve <b>Crypto</b>graphy [ECC]) <b>f</b>or <b>encry</b>ption <b>o</b>r <b>sign</b>ing.

### Envelope encryption
<b>Encr</b>ypt <b>da</b>ta <b>wi</b>th <b>a</b> **data key**; <b>encr</b>ypt <b>th</b>at <b>da</b>ta <b>k</b>ey <b>wi</b>th <b>a</b> **KMS key**; <b>sto</b>re <b>t</b>he <b>encry</b>pted <b>da</b>ta <b>k</b>ey <b>wi</b>th <b>t</b>he <b>da</b>ta. KMS <b>on</b>ly <b>hand</b>les <b>sma</b>ll <b>ke</b>ys, <b>s</b>o <b>y</b>ou <b>c</b>an <b>encr</b>ypt <b>a</b>ny <b>si</b>ze <b>loca</b>lly.

### Client-side vs server-side encryption
**Server-side**: <b>t</b>he <b>serv</b>ice <b>encr</b>ypts <b>aft</b>er <b>recei</b>ving <b>t</b>he <b>da</b>ta (S3 SSE). **Client-side**: <b>y</b>ou <b>encr</b>ypt <b>bef</b>ore <b>send</b>ing, <b>s</b>o <b>t</b>he <b>serv</b>ice <b>on</b>ly <b>se</b>es <b>ciphe</b>rtext (AWS <b>Encry</b>ption SDK, S3 <b>Encry</b>ption <b>Cli</b>ent).

### Certificates and SSH keys
<b>A</b> **Transport Layer Security (TLS) certificate** <b>bin</b>ds <b>a</b> <b>dom</b>ain <b>t</b>o <b>a</b> <b>pub</b>lic <b>k</b>ey, <b>sig</b>ned <b>b</b>y <b>a</b> <b>certif</b>icate <b>autho</b>rity (<b>pub</b>lic <b>Certif</b>icate <b>Autho</b>rity [CA] <b>o</b>r <b>yo</b>ur <b>priv</b>ate CA). <b>A</b> **Secure Shell (SSH) key pair** (<b>priv</b>ate <b>k</b>ey <b>ke</b>pt <b>b</b>y <b>y</b>ou, <b>pub</b>lic <b>k</b>ey <b>o</b>n <b>t</b>he <b>ser</b>ver) <b>authent</b>icates SSH <b>log</b>ins; <b>gene</b>rate <b>wi</b>th `ssh-keygen` <b>o</b>r <b>Elas</b>tic <b>Comp</b>ute <b>Clo</b>ud (EC2) <b>k</b>ey <b>pai</b>rs.

### Hashing and HMAC
**Hashing** (SHA-256) = <b>o</b>ne-<b>w</b>ay <b>finger</b>print <b>f</b>or <b>integ</b>rity. **Hash-based Message Authentication Code (HMAC)** = <b>ha</b>sh <b>wi</b>th <b>a</b> <b>sec</b>ret <b>k</b>ey, <b>prov</b>ing <b>a</b> <b>mess</b>age <b>ca</b>me <b>fr</b>om <b>some</b>one <b>wi</b>th <b>t</b>he <b>k</b>ey (<b>us</b>ed <b>i</b>n <b>Signa</b>ture <b>Vers</b>ion 4 [SigV4] <b>a</b>nd <b>webh</b>ooks).

### PII and PHI, masking and redaction
**Personally Identifiable Information (PII)**: <b>da</b>ta <b>identi</b>fying <b>a</b> <b>per</b>son; **Protected Health Information (PHI)**: <b>hea</b>lth <b>da</b>ta. **Masking** <b>hid</b>es <b>pa</b>rt (`***-**-1234`); **redaction** <b>remo</b>ves <b>i</b>t; **tokenization** <b>repl</b>aces <b>i</b>t <b>wi</b>th <b>a</b> <b>refer</b>ence.

---

## Delivery concepts

### CI/CD
**Continuous integration**: <b>eve</b>ry <b>cha</b>nge <b>i</b>s <b>bui</b>lt <b>a</b>nd <b>tes</b>ted <b>automat</b>ically. **Continuous delivery/deployment**: <b>eve</b>ry <b>pass</b>ing <b>cha</b>nge <b>i</b>s <b>rele</b>ased <b>t</b>o <b>enviro</b>nments <b>automat</b>ically (<b>wi</b>th <b>o</b>r <b>with</b>out <b>a</b> <b>man</b>ual <b>appr</b>oval <b>t</b>o <b>produ</b>ction).

### Environments and stages
<b>Sepa</b>rate <b>cop</b>ies <b>o</b>f <b>t</b>he <b>a</b>pp (<b>d</b>ev, <b>te</b>st, <b>stag</b>ing, <b>pr</b>od) <b>wi</b>th <b>the</b>ir <b>o</b>wn <b>configu</b>ration <b>a</b>nd <b>da</b>ta <b>s</b>o <b>chan</b>ges <b>a</b>re <b>veri</b>fied <b>bef</b>ore <b>reac</b>hing <b>use</b>rs.

### Unit, integration and end-to-end tests
**Unit**: <b>o</b>ne <b>func</b>tion <b>i</b>n <b>isola</b>tion (<b>depend</b>encies <b>moc</b>ked). **Integration**: <b>compo</b>nents <b>toge</b>ther <b>wi</b>th <b>re</b>al <b>serv</b>ices. **End-to-end**: <b>t</b>he <b>who</b>le <b>us</b>er <b>fl</b>ow. **Regression**: <b>r</b>e-<b>runn</b>ing <b>tes</b>ts <b>t</b>o <b>cat</b>ch <b>o</b>ld <b>bu</b>gs <b>retur</b>ning. **Mock**: <b>a</b> <b>fa</b>ke <b>depen</b>dency <b>retur</b>ning <b>can</b>ned <b>respo</b>nses.

### Deployment strategies
**All-at-once**, **rolling** (<b>batc</b>hes), **immutable** (<b>n</b>ew <b>insta</b>nces), **blue/green** (<b>swi</b>tch <b>traf</b>fic <b>betw</b>een <b>t</b>wo <b>fu</b>ll <b>enviro</b>nments), **canary** (<b>sma</b>ll % <b>fir</b>st), **linear** (<b>equ</b>al <b>ste</b>ps). <b>Roll</b>back = <b>ret</b>urn <b>traf</b>fic/<b>vers</b>ion <b>t</b>o <b>t</b>he <b>prev</b>ious <b>go</b>od <b>sta</b>te.

### Infrastructure as Code (IaC)
<b>Defi</b>ning <b>infrast</b>ructure <b>i</b>n <b>templ</b>ates/<b>co</b>de (<b>CloudFo</b>rmation, <b>Serve</b>rless <b>Applic</b>ation <b>Mod</b>el [SAM], <b>Clo</b>ud <b>Develo</b>pment <b>K</b>it [CDK]) <b>s</b>o <b>enviro</b>nments <b>a</b>re <b>repea</b>table <b>a</b>nd <b>versi</b>oned.

### Source control
**Git**: <b>comm</b>its, **branches** (<b>feat</b>ure <b>wo</b>rk), **tags** (<b>rele</b>ase <b>lab</b>els <b>li</b>ke `v2.0.1`), **pull requests** (<b>rev</b>iew <b>bef</b>ore <b>mer</b>ge), <b>mer</b>ge <b>strat</b>egies.

### Semantic versioning
`MAJOR.MINOR.PATCH`: <b>maj</b>or = <b>brea</b>king <b>cha</b>nge, <b>min</b>or = <b>n</b>ew <b>feat</b>ure, <b>pat</b>ch = <b>b</b>ug <b>f</b>ix.

---

## Observability concepts

### Logs, metrics and traces
**Logs**: <b>eve</b>nt <b>reco</b>rds (<b>pre</b>fer **structured** JSON <b>wi</b>th <b>requ</b>est IDs). **Metrics**: <b>numb</b>ers <b>ov</b>er <b>ti</b>me (<b>late</b>ncy, <b>err</b>ors). **Traces**: <b>a</b> <b>reque</b>st's <b>pa</b>th <b>acr</b>oss <b>serv</b>ices <b>wi</b>th <b>tim</b>ing (<b>X</b>-<b>R</b>ay, <b>OpenTel</b>emetry). <b>Toge</b>ther <b>th</b>ey <b>ans</b>wer "<b>wh</b>at <b>bro</b>ke, <b>whe</b>re <b>a</b>nd <b>w</b>hy."

### Correlation ID
<b>A</b> <b>uni</b>que ID <b>pas</b>sed <b>thro</b>ugh <b>a</b>ll <b>serv</b>ices <b>f</b>or <b>o</b>ne <b>requ</b>est <b>s</b>o <b>i</b>ts <b>lo</b>gs <b>a</b>nd <b>tra</b>ces <b>c</b>an <b>b</b>e <b>joi</b>ned.

### Health checks, liveness and readiness
**Liveness**: <b>i</b>s <b>t</b>he <b>proc</b>ess <b>ali</b>ve (<b>rest</b>art <b>i</b>f <b>n</b>ot)? **Readiness**: <b>i</b>s <b>i</b>t <b>rea</b>dy <b>t</b>o <b>rece</b>ive <b>traf</b>fic (<b>e</b>.g., <b>fini</b>shed <b>load</b>ing, DB <b>reach</b>able)? <b>Lo</b>ad <b>balan</b>cers <b>a</b>nd <b>Kuber</b>netes <b>u</b>se <b>the</b>se <b>pro</b>bes.

### Percentiles
p50/p90/p99 <b>late</b>ncy <b>sh</b>ow <b>typi</b>cal <b>a</b>nd <b>wor</b>st-<b>ca</b>se <b>us</b>er <b>exper</b>ience <b>bet</b>ter <b>th</b>an <b>aver</b>ages.

### Concurrency, thread pools and connection pools
**Concurrency** = <b>h</b>ow <b>ma</b>ny <b>requ</b>ests <b>a</b>re <b>hand</b>led <b>a</b>t <b>on</b>ce. <b>A</b> **thread pool** <b>lim</b>its <b>a</b>nd <b>reu</b>ses <b>wor</b>ker <b>thre</b>ads; <b>a</b> **connection pool** <b>reu</b>ses <b>data</b>base <b>connec</b>tions <b>inst</b>ead <b>o</b>f <b>open</b>ing <b>n</b>ew <b>on</b>es <b>p</b>er <b>requ</b>est.

---

## AI concepts for developers

### Foundation model, LLM and tokens
<b>A</b> **foundation model** <b>i</b>s <b>a</b> <b>lar</b>ge <b>p</b>re-<b>trai</b>ned <b>mod</b>el; <b>a</b> **Large Language Model (LLM)** <b>i</b>s <b>a</b> <b>found</b>ation <b>mod</b>el <b>f</b>or <b>te</b>xt. <b>Inp</b>ut <b>a</b>nd <b>out</b>put <b>a</b>re <b>meas</b>ured <b>i</b>n **tokens** (<b>wo</b>rd <b>pie</b>ces); <b>pric</b>ing, <b>lim</b>its <b>a</b>nd <b>late</b>ncy <b>dep</b>end <b>o</b>n <b>tok</b>en <b>cou</b>nts.

### Prompt, system prompt and inference parameters
<b>T</b>he **prompt** <b>i</b>s <b>t</b>he <b>inp</b>ut; <b>a</b> **system prompt** <b>se</b>ts <b>beha</b>vior <b>rul</b>es. <b>Param</b>eters: **temperature** (<b>rando</b>mness), **top-p**, **max tokens**, **stop sequences**.

### Streaming responses
<b>Retur</b>ning <b>mod</b>el <b>out</b>put <b>tok</b>en <b>b</b>y <b>tok</b>en <b>a</b>s <b>it</b>'s <b>gener</b>ated (<b>Bedr</b>ock `ConverseStream`/`InvokeModelWithResponseStream`) <b>f</b>or <b>respo</b>nsive UIs.

### RAG
**Retrieval-Augmented Generation**: <b>retr</b>ieve <b>rele</b>vant <b>docum</b>ents (<b>v</b>ia <b>embed</b>dings/<b>vec</b>tor <b>sea</b>rch) <b>a</b>nd <b>a</b>dd <b>th</b>em <b>t</b>o <b>t</b>he <b>pro</b>mpt <b>s</b>o <b>t</b>he <b>mod</b>el <b>answ</b>ers <b>fr</b>om <b>yo</b>ur <b>da</b>ta.

### Embeddings
<b>Num</b>ber <b>vect</b>ors <b>repres</b>enting <b>mean</b>ing; <b>simi</b>lar <b>tex</b>ts <b>ha</b>ve <b>nea</b>rby <b>vect</b>ors. <b>Us</b>ed <b>f</b>or <b>sema</b>ntic <b>sea</b>rch <b>a</b>nd <b>Retri</b>eval-<b>Augme</b>nted <b>Gener</b>ation (RAG).

### AI agents, tools and human-in-the-loop
<b>A</b>n **agent** <b>us</b>es <b>a</b> <b>mod</b>el <b>t</b>o <b>pl</b>an <b>a</b>nd <b>ca</b>ll **tools** (APIs, <b>funct</b>ions) <b>ov</b>er <b>seve</b>ral <b>ste</b>ps. **Tool-use authorization** <b>lim</b>its <b>whi</b>ch <b>too</b>ls/<b>acti</b>ons <b>i</b>t <b>m</b>ay <b>ca</b>ll <b>wi</b>th <b>whi</b>ch <b>permis</b>sions; **session isolation** <b>kee</b>ps <b>o</b>ne <b>use</b>r's <b>cont</b>ext <b>a</b>nd <b>da</b>ta <b>sepa</b>rate <b>fr</b>om <b>anoth</b>er's; **human-in-the-loop** <b>requ</b>ires <b>a</b> <b>per</b>son <b>t</b>o <b>appr</b>ove <b>sensi</b>tive <b>acti</b>ons.

### Guardrails
<b>Poli</b>cies <b>th</b>at <b>fil</b>ter <b>mod</b>el <b>inp</b>uts/<b>outp</b>uts: <b>harm</b>ful <b>cont</b>ent <b>categ</b>ories, **denied topics**, **sensitive information (PII) detection/redaction**, <b>wo</b>rd <b>filt</b>ers, **prompt attack (injection/jailbreak) detection**, <b>groun</b>ding <b>che</b>cks.

### Prompt injection
<b>A</b>n <b>att</b>ack <b>whe</b>re <b>inp</b>ut <b>te</b>xt (<b>fr</b>om <b>a</b> <b>us</b>er <b>o</b>r <b>a</b> <b>retri</b>eved <b>docu</b>ment) <b>tri</b>es <b>t</b>o <b>over</b>ride <b>t</b>he <b>mode</b>l's <b>instru</b>ctions ("<b>ign</b>ore <b>prev</b>ious <b>instru</b>ctions <b>a</b>nd …"). <b>Miti</b>gate <b>wi</b>th <b>guardr</b>ails' <b>pro</b>mpt <b>att</b>ack <b>filt</b>ers, <b>separ</b>ating <b>instru</b>ctions <b>fr</b>om <b>da</b>ta, <b>lea</b>st-<b>privi</b>lege <b>too</b>ls <b>a</b>nd <b>hum</b>an <b>appr</b>oval.

### AI-assisted development
<b>Usi</b>ng AI <b>too</b>ls <b>t</b>o <b>gene</b>rate <b>co</b>de <b>fr</b>om <b>specifi</b>cations (**spec-driven development**), <b>comp</b>lete <b>co</b>de, <b>rev</b>iew <b>i</b>t, <b>sugg</b>est <b>refac</b>tors, <b>sc</b>an <b>f</b>or <b>secu</b>rity <b>iss</b>ues, <b>wri</b>te <b>tes</b>ts, <b>anal</b>yze <b>err</b>ors <b>a</b>nd <b>lo</b>gs. **Kiro** <b>i</b>s <b>AWS</b>'s <b>agen</b>tic <b>develo</b>pment <b>enviro</b>nment <b>f</b>or <b>th</b>is.

### MCP
**Model Context Protocol**: <b>op</b>en <b>stan</b>dard <b>f</b>or <b>conne</b>cting AI <b>age</b>nts/<b>assis</b>tants <b>t</b>o <b>too</b>ls <b>a</b>nd <b>da</b>ta <b>sour</b>ces.
