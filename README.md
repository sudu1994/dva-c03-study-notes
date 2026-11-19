# AWS Certified Developer – Associate (DVA-C03) Study Notes

Self-contained study notes for **DVA-C03**, the Developer Associate version in use from **December 1, 2026** (DVA-C02's last day is November 30, 2026). Built from the official DVA-C03 exam guide and its in-scope service list. Every term is defined where it's used.

## How to study with this repo

1. **Skim [00-developer-basics.md](00-developer-basics.md)**: APIs, SDKs, SigV4, idempotency, retries, serialization, DynamoDB keys, caching, OAuth/JWT, envelope encryption, CI/CD, observability, and AI concepts (tokens, RAG, agents, guardrails, prompt injection). Domain files link back to it.
2. **Work through the four domain files.** Each starts with the official tasks/skills, then covers every service and feature: what it is and key terms, how to use it in code/config, troubleshooting, and **exam triggers** (scenario words → answer). Each ends with gotchas and a quick-recall summary.
3. **Review [05-cheat-sheet.md](05-cheat-sheet.md)** before the exam.
4. **Look up abbreviations in [00-abbreviations-glossary.md](00-abbreviations-glossary.md).** Each is also spelled out on first use in every file; inside brackets the abbreviation uses square brackets.

## Files

| File | Domain | Weight |
|---|---|---|
| [00-developer-basics.md](00-developer-basics.md) | The 101 layer for developers | — |
| [01-development-with-aws-services.md](01-development-with-aws-services.md) | Domain 1: Development with AWS Services | 30% |
| [02-security.md](02-security.md) | Domain 2: Security (incl. new AI security task) | 26% |
| [03-testing-deployment.md](03-testing-deployment.md) | Domain 3: Testing and Deployment | 22% |
| [04-troubleshooting-optimization.md](04-troubleshooting-optimization.md) | Domain 4: Troubleshooting and Optimization | 22% |
| [05-cheat-sheet.md](05-cheat-sheet.md) | Decision tables, traps, numbers | — |
| [00-abbreviations-glossary.md](00-abbreviations-glossary.md) | A–Z abbreviations | — |

## Overlap tags

`[also: DOP]` (DevOps Engineer Professional), `[also: SAA]` (Solutions Architect Associate), `[also: DEA]` (Data Engineer Associate), `[also: AIP]` (Generative AI Developer Professional), `[also: AIF]` (AI Practitioner) mark shared material. Everything is still covered in full here.

## What's new in C03 vs C02

- New **Task 2.3: AI service security** (model access, data privacy, guardrails, agent security, protecting AI logs).
- AI-assisted development, testing, CI/CD, error analysis and optimization skills (Kiro, Bedrock, AgentCore, DevOps Agent).
- Presigned URLs, container image builds, ECS/EKS/Fargate deployments added; unit testing with SAM moved to Domain 3; Lambda tuning moved to Domain 4.
- Weights: Development 32 → 30%, Deployment 24 → 22% (now "Testing and Deployment"), Troubleshooting 18 → 22%.

## Exam at a glance

- 65 questions (50 scored, 15 unscored), 130 minutes, passing score 720 / 1000; multiple choice and multiple response.
- Official guide: https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03.html

> Numbers and quotas change over time. Where AWS raised a limit recently, both values are given. Newer AI services are described only as far as AWS's official pages describe them.
