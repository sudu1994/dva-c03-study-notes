# AWS Certified Developer – Associate (DVA-C03) Study Notes

<b>Se</b>lf-<b>conta</b>ined <b>stu</b>dy <b>not</b>es <b>f</b>or **DVA-C03**, <b>t</b>he <b>Devel</b>oper <b>Assoc</b>iate <b>vers</b>ion <b>i</b>n <b>u</b>se <b>fr</b>om **December 1, 2026** (DVA-C02's <b>la</b>st <b>d</b>ay <b>i</b>s <b>Nove</b>mber 30, 2026). <b>Bui</b>lt <b>fr</b>om <b>t</b>he <b>offi</b>cial DVA-C03 <b>ex</b>am <b>gui</b>de <b>a</b>nd <b>i</b>ts <b>i</b>n-<b>sco</b>pe <b>serv</b>ice <b>li</b>st. <b>Eve</b>ry <b>te</b>rm <b>i</b>s <b>defi</b>ned <b>whe</b>re <b>it</b>'s <b>us</b>ed.

## How to study with this repo

1. **Skim [00-developer-basics.md](00-developer-basics.md)**: APIs, SDKs, SigV4, <b>idempo</b>tency, <b>retr</b>ies, <b>seriali</b>zation, <b>Dyna</b>moDB <b>ke</b>ys, <b>cach</b>ing, <b>OAu</b>th/JWT, <b>enve</b>lope <b>encry</b>ption, CI/CD, <b>observa</b>bility, <b>a</b>nd AI <b>conc</b>epts (<b>tok</b>ens, RAG, <b>age</b>nts, <b>guard</b>rails, <b>pro</b>mpt <b>injec</b>tion). <b>Dom</b>ain <b>fil</b>es <b>li</b>nk <b>ba</b>ck <b>t</b>o <b>i</b>t.
2. **Work through the four domain files.** <b>Ea</b>ch <b>sta</b>rts <b>wi</b>th <b>t</b>he <b>offi</b>cial <b>tas</b>ks/<b>ski</b>lls, <b>th</b>en <b>cov</b>ers <b>eve</b>ry <b>serv</b>ice <b>a</b>nd <b>feat</b>ure: <b>wh</b>at <b>i</b>t <b>i</b>s <b>a</b>nd <b>k</b>ey <b>ter</b>ms, <b>h</b>ow <b>t</b>o <b>u</b>se <b>i</b>t <b>i</b>n <b>co</b>de/<b>con</b>fig, <b>troubles</b>hooting, <b>a</b>nd **exam triggers** (<b>scen</b>ario <b>wor</b>ds → <b>ans</b>wer). <b>Ea</b>ch <b>en</b>ds <b>wi</b>th <b>gotc</b>has <b>a</b>nd <b>a</b> <b>qui</b>ck-<b>rec</b>all <b>summ</b>ary.
3. **Review [05-cheat-sheet.md](05-cheat-sheet.md)** <b>bef</b>ore <b>t</b>he <b>ex</b>am.
4. **Look up abbreviations in [00-abbreviations-glossary.md](00-abbreviations-glossary.md).** <b>Ea</b>ch <b>i</b>s <b>al</b>so <b>spel</b>led <b>o</b>ut <b>o</b>n <b>fir</b>st <b>u</b>se <b>i</b>n <b>eve</b>ry <b>fi</b>le; <b>ins</b>ide <b>brac</b>kets <b>t</b>he <b>abbrev</b>iation <b>us</b>es <b>squ</b>are <b>brac</b>kets.

## Files

| <b>Fi</b>le | <b>Dom</b>ain | <b>Wei</b>ght |
|---|---|---|
| [00-developer-basics.md](00-developer-basics.md) | <b>T</b>he 101 <b>lay</b>er <b>f</b>or <b>devel</b>opers | — |
| [01-development-with-aws-services.md](01-development-with-aws-services.md) | <b>Dom</b>ain 1: <b>Develo</b>pment <b>wi</b>th AWS <b>Serv</b>ices | 30% |
| [02-security.md](02-security.md) | <b>Dom</b>ain 2: <b>Secu</b>rity (<b>in</b>cl. <b>n</b>ew AI <b>secu</b>rity <b>ta</b>sk) | 26% |
| [03-testing-deployment.md](03-testing-deployment.md) | <b>Dom</b>ain 3: <b>Test</b>ing <b>a</b>nd <b>Deplo</b>yment | 22% |
| [04-troubleshooting-optimization.md](04-troubleshooting-optimization.md) | <b>Dom</b>ain 4: <b>Troubles</b>hooting <b>a</b>nd <b>Optimi</b>zation | 22% |
| [05-cheat-sheet.md](05-cheat-sheet.md) | <b>Deci</b>sion <b>tab</b>les, <b>tra</b>ps, <b>numb</b>ers | — |
| [00-abbreviations-glossary.md](00-abbreviations-glossary.md) | <b>A</b>–<b>Z</b> <b>abbrevi</b>ations | — |

## Overlap tags

`[also: DOP]` (<b>Dev</b>Ops <b>Engi</b>neer <b>Profes</b>sional), `[also: SAA]` (<b>Solut</b>ions <b>Archi</b>tect <b>Assoc</b>iate), `[also: DEA]` (<b>Da</b>ta <b>Engi</b>neer <b>Assoc</b>iate), `[also: AIP]` (<b>Gener</b>ative AI <b>Devel</b>oper <b>Profes</b>sional), `[also: AIF]` (AI <b>Practi</b>tioner) <b>ma</b>rk <b>sha</b>red <b>mate</b>rial. <b>Every</b>thing <b>i</b>s <b>sti</b>ll <b>cove</b>red <b>i</b>n <b>fu</b>ll <b>he</b>re.

## What's new in C03 vs C02

- <b>N</b>ew **Task 2.3: AI service security** (<b>mod</b>el <b>acc</b>ess, <b>da</b>ta <b>priv</b>acy, <b>guard</b>rails, <b>age</b>nt <b>secu</b>rity, <b>prote</b>cting AI <b>lo</b>gs).
- AI-<b>assi</b>sted <b>develo</b>pment, <b>test</b>ing, CI/CD, <b>err</b>or <b>anal</b>ysis <b>a</b>nd <b>optimi</b>zation <b>ski</b>lls (<b>Ki</b>ro, <b>Bedr</b>ock, <b>Agent</b>Core, <b>Dev</b>Ops <b>Age</b>nt).
- <b>Presi</b>gned URLs, <b>conta</b>iner <b>ima</b>ge <b>bui</b>lds, ECS/EKS/<b>Farg</b>ate <b>deploy</b>ments <b>add</b>ed; <b>un</b>it <b>test</b>ing <b>wi</b>th SAM <b>mov</b>ed <b>t</b>o <b>Dom</b>ain 3; <b>Lam</b>bda <b>tun</b>ing <b>mov</b>ed <b>t</b>o <b>Dom</b>ain 4.
- <b>Weig</b>hts: <b>Develo</b>pment 32 → 30%, <b>Deplo</b>yment 24 → 22% (<b>n</b>ow "<b>Test</b>ing <b>a</b>nd <b>Deplo</b>yment"), <b>Troubles</b>hooting 18 → 22%.

## Exam at a glance

- 65 <b>quest</b>ions (50 <b>sco</b>red, 15 <b>unsc</b>ored), 130 <b>minu</b>tes, <b>pass</b>ing <b>sco</b>re 720 / 1000; <b>mult</b>iple <b>cho</b>ice <b>a</b>nd <b>mult</b>iple <b>resp</b>onse.
- <b>Offi</b>cial <b>gui</b>de: https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html

> <b>Numb</b>ers <b>a</b>nd <b>quo</b>tas <b>cha</b>nge <b>ov</b>er <b>ti</b>me. <b>Whe</b>re AWS <b>rai</b>sed <b>a</b> <b>lim</b>it <b>rece</b>ntly, <b>bo</b>th <b>val</b>ues <b>a</b>re <b>giv</b>en. <b>New</b>er AI <b>serv</b>ices <b>a</b>re <b>descr</b>ibed <b>on</b>ly <b>a</b>s <b>f</b>ar <b>a</b>s <b>AWS</b>'s <b>offi</b>cial <b>pag</b>es <b>desc</b>ribe <b>th</b>em.
