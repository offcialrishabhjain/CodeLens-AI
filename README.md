#  CodeLens AI
### *Understand any codebase in 90 seconds.*

> Built for the **AWS AI for Bharat Hackathon 2026** by **Code & Co.**

**CodeLens AI** is an AI-powered code intelligence platform built for the next generation of developers. Upload any project — get a full architecture diagram, AI explanation, execution trace, health score, refactor suggestions, and a live Q&A interface. No setup. No environment. No barrier.

🌐 **Live Demo:** [main.doiu62ayzwp4v.amplifyapp.com](https://main.doiu62ayzwp4v.amplifyapp.com)  
📁 **GitHub:** [github.com/offcialrishabhjain/CodeLens-AI](https://github.com/offcialrishabhjain/CodeLens-AI)

---

## 🇮🇳 The Problem We're Solving

Millions of developers are learning to code — but reading and understanding real codebases is brutally hard without a mentor. Most tools assume you already know what you're looking at.

**CodeLens AI bridges that gap.** Drop in any project. Walk away understanding it.

---

## ✨ Features

| # | Feature | What it does |
|---|---------|-------------|
|  01 | **Architecture Visualizer** | Auto-generates a dependency tree and architecture map from your code using static analysis |
|  02 | **AI Explanation** | Explains the entire codebase in **ELI5**, **Technical**, or **Expert** mode — tailored to your level |
|  03 | **Execution Simulator** | Traces call graphs and variable flow statically — understand how code runs without executing it |
|  04 | **Code Optimizer** | Detects naming issues, duplicate logic, security vulnerabilities, and performance bottlenecks with specific fixes |
|  05 | **Health Score** | Scores the codebase across Readability, Modularity, Security, and Test Coverage — each out of 10 |
|  06 | **Doubts Chat** | Ask anything about your uploaded project — context-aware Q&A powered by Amazon Nova |

---

##  How It Works

```
Developer uploads ZIP or pastes GitHub URL
            ↓
    AWS Lambda validates & stores to S3
            ↓
    S3 event triggers codelens-upload-validator λ
            ↓
    Reads ZIP → filters binaries → extracts source code
            ↓
    Builds structured prompt (up to 80,000 chars of context)
            ↓
    Amazon Bedrock → Nova Micro (primary, AP-SOUTH-1)
                  → Nova Lite  (fallback, same region)
                  → Nova Lite  (last resort, US-EAST-1)
            ↓
    Parses structured 6-section AI response
            ↓
    Stores results in DynamoDB (CodeLensResults)
            ↓
    Frontend polls DynamoDB → renders all 6 tabs instantly
```

**GitHub URL flow** — a dedicated Lambda (`codelens-github-fetcher`) resolves the default branch via a lightweight HEAD request, downloads the ZIP in 64 KB chunks with live byte-count enforcement, validates the magic bytes (`PK\x03\x04`), and uploads directly to S3 to trigger the same analysis pipeline.

---

##  Tech Stack

### Frontend
| Technology | Role |
|-----------|------|
| HTML · CSS · JavaScript | Single-file SPA — zero frameworks, zero dependencies |
| Bebas Neue + IBM Plex Mono | Terminal-aesthetic UI designed for focus |
| AWS SDK for JavaScript v2 | Direct S3 upload + Lambda invocation from the browser |
| AWS Amplify | Hosting, CI/CD pipeline, global CDN |

### Backend — Lambda Functions
| Function | Runtime | Role |
|---------|---------|------|
| `codelens-upload-validator` | Python 3.11 | Core analysis engine + Doubts chat handler |
| `codelens-github-fetcher` | Python 3.11 | GitHub URL → chunked ZIP download → S3 upload |

### AWS Services
| Service | Role |
|---------|------|
| Amazon S3 (`codelens-uploads-rishabh-2026`) | Upload storage, S3 event trigger for analysis |
| Amazon DynamoDB (`CodeLensResults`) | Stores analysis output, polled by frontend |
| Amazon Cognito Identity Pool | Federated browser auth with scoped IAM access |
| AWS Amplify | Frontend hosting + GitHub Actions CI/CD |

### AI Layer
| Model | Role |
|-------|------|
| `apac.amazon.nova-micro-v1:0` | **Primary** — fast, cost-efficient, AP-SOUTH-1 |
| `amazon.nova-lite-v1:0` (AP-SOUTH-1) | **Fallback** — higher quality, same region |
| `amazon.nova-lite-v1:0` (US-EAST-1) | **Last resort** — cross-region escape hatch for throttling |

---

##  Resilience Architecture

The system is built to never silently fail:

- **Multi-model fallback** — 3 models × 3 retries = up to 9 inference attempts before a hard error
- **Exponential backoff** — 1s → 2s → 4s (capped at 8s) per model on `ThrottlingException`
- **Immediate fallback** on `ModelErrorException` and `ServiceUnavailableException` — retries won't fix these, so we move on instantly
- **Chunked download** — 64 KB chunks with live byte-count enforcement, not just `Content-Length` header trust
- **ZIP magic byte validation** — rejects non-ZIP files even if `.zip` extension is present
- **DynamoDB polling** — exponential backoff starting at 3s, 1.5× multiplier, 15s cap, 30 max attempts
- **Context truncation** — graceful handling at 80,000 chars with a visible warning banner for large repos

---

##  Project Structure

```
CodeLens-AI/
├── index.html              # Complete frontend SPA (single file, fully self-contained)
├── design.md               # Full technical design document
├── requirements.md         # Project requirements specification
└── .github/
    └── workflows/          # GitHub Actions → AWS Amplify auto-deploy
```

**Lambda functions** (deployed on AWS, source in submission PDFs):
```
codelens-upload-validator   # Main engine: ZIP processing, AI invocation, chat handler
codelens-github-fetcher     # GitHub URL resolver and ZIP downloader
```

---

##  Try It Yourself

1. **Visit** [main.doiu62ayzwp4v.amplifyapp.com](https://main.doiu62ayzwp4v.amplifyapp.com)
2. **Set your level** — Beginner, Intermediate, or Advanced
3. **Upload your project** — two ways:
   - Drop a ZIP (any GitHub repo → `Code → Download ZIP`, max 50 MB)
   - Paste a public GitHub URL directly
4. **Hit Analyse** — wait ~60–90 seconds for full processing
5. **Explore** all 6 tabs — architecture, explanation, simulator, optimizer, health score, and ask doubts

---

##  Why Amazon Bedrock + Nova?

- **Zero infrastructure** — fully managed, no GPU provisioning, scales to zero between requests
- **Cost-smart** — Nova Micro handles the majority of requests at minimal cost; Nova Lite only kicks in on failure
- **Grounded chat** — each Doubts chat turn sends the prior analysis summary + code context, keeping answers relevant to the actual uploaded project
- **Structured output** — a strict 6-section numbered prompt format ensures every AI response is consistently parseable by the frontend
- **Regional resilience** — cross-region fallback means users in AP-SOUTH-1 are never blocked by a regional outage


---

## 👥 Team

**Code & Co.** — AWS AI for Bharat Hackathon 2026  
Region: `AP-SOUTH-1` · Build: `2026.03`

---
