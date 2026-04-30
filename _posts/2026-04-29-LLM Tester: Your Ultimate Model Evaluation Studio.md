---
layout: post
title: LLM Tester - Your Ultimate Model Evaluation Studio
date: 2026-04-29
description: A unified model evaluation studio for testing, comparing, and batching LLM queries from Claude and GPT with vector retrieval and prompt templates. 
tags: [genai,llm]
categories: GenAI
giscus_comments: false
related_posts: false
featured: true
toc:
  sidebar: left
pretty_table: true
---

**Author:** Kunal  
**Date:** April 2026  
**Read Time:** 8 minutes

---

## Introduction

Evaluating and testing Large Language Models can be a complex, tedious task. Whether you're comparing Claude and GPT side-by-side, running batch experiments on hundreds of prompts, or searching through vector databases for relevant context—existing tools often force you to juggle multiple tabs, scripts, and APIs.

**LLM Tester** is a unified, production-grade evaluation platform that brings all these capabilities under one intuitive interface. Built with [Reflex](https://reflex.dev), it's fast, secure, and designed for both quick experiments and serious model evaluation workflows.

---

## What is LLM Tester?

LLM Tester is a web-based **model evaluation studio** that lets you:
- Test single prompts against Claude and GPT models simultaneously
- Compare model responses side-by-side
- Run batch tests on hundreds of inputs in parallel
- Search vector databases (Pinecone) for semantic retrieval
- Organize and reuse prompt templates
- Track query history with cost analysis and ratings

Every feature is user-isolated, secure, and configurable. You control the temperature, max tokens, concurrency—everything.

---

## Key Features Explained

### 1. **🔐 User Authentication & Data Isolation**

Security comes first. LLM Tester uses SQLite authentication to keep your data private.

**What this means:**
- Each user has a separate login
- Your API keys never leave your session
- Your query history, templates, and settings are completely isolated
- No cross-user data leakage

**Perfect for:** Teams, research labs, or solo developers who need to keep experiments confidential.

#### Login Screen

![LLM Tester Login Interface](/assets/img/llm_tester_images/image_6.png)

---

### 2. **⚡ Single Prompt Testing**

Test any prompt instantly against Claude or GPT.

**Features:**
- **Multi-vendor support**: Run the same prompt on Anthropic (Claude) or OpenAI (GPT) with one click
- **Model selector**: Dynamic dropdown that updates based on your chosen vendor
- **Real-time responses**: Markdown-formatted output with syntax highlighting
- **Performance stats**: Automatic tracking of word count, latency (ms), tokens used, and cost

**Use case:** Quick brainstorming, prompt refinement, or verifying model behavior before batch testing.

#### Prompt Testing Interface

![LLM Tester Prompt Tab](/assets/img/llm_tester_images/image_7.png)

---

### 3. **🔀 Compare Tab**

See how different models stack up side-by-side.

**What you can do:**
- Enter a single prompt
- Run it on multiple models simultaneously
- Compare responses in a clean side-by-side layout
- Instantly see latency, token counts, and cost differences
- Rate responses and add notes for future reference

**Why it matters:** Model selection is critical. This tab eliminates the guesswork—you see exactly which model performs best for *your* specific use case, not some benchmark average.

#### Side-by-Side Model Comparison

![LLM Tester Compare Tab](/assets/img/llm_tester_images/image_8.png)

---

### 4. **🔍 Retrieval Tab (Vector Search)**

Connect to Pinecone and search semantic embeddings.

**Capabilities:**
- **Semantic search**: Find relevant documents using OpenAI's `text-embedding-3-small` model
- **MMR (Max Marginal Relevance)**: Toggle MMR to get diverse, non-redundant results
- **Metadata filtering**: Add filters with operators like `$eq`, `$ne`, `$gt`, `$in` (useful for date ranges, categories, etc.)
- **Multiple indices**: Support for different index types (articles, videos, etc.)
- **Structured results**: View matches in a clean table with metadata
- **Export**: Download results as JSON or CSV

#### Index Selection

![Retrieval Index Selection](/assets/img/llm_tester_images/image_4.png)

#### Retrieval Results Display

![Retrieval Results Interface](/assets/img/llm_tester_images/image_1.png)

**When to use:** 
- RAG (Retrieval-Augmented Generation) pipeline testing
- Finding training data for fine-tuning
- Gathering context for prompt engineering

---

### 5. **📁 Templates Tab**

Build a library of reusable prompts.

**Features:**
- Save any prompt as a named template
- Organize by category (e.g., "Customer Support", "Code Review", "Content Gen")
- Reuse templates across all tabs (Single, Compare, Batch)
- Full attribution—templates show who created them

**Why templates matter:** Good prompts are valuable. Don't rewrite "Explain this code" every time. Save it once, reuse it forever.

#### Template Management

![Templates Library Interface](/assets/img/llm_tester_images/image_5.png)

#### Saving a Template

![Save Template Dialog](/assets/img/llm_tester_images/image_2.png)

---

### 6. **📜 History Tab**

Never lose a query again.

**Track:**
- Every prompt you've ever submitted
- Full responses (cached for instant review)
- Timestamp, model, vendor, latency
- User ratings and custom notes
- Cost breakdown (calculated per request)

**Benefits:**
- Find past experiments without digging through notebooks
- Identify patterns in model behavior
- Audit trail for research reproducibility
- Cost tracking for budget management

#### Complete Query History

![History Tab with Query Records](/assets/img/llm_tester_images/image_0.png)

---

## Benefits: Why LLM Tester Stands Out

### **1. Speed**
- No context switching between tabs and CLIs
- Instant responses, live progress tracking
- Batch processing parallelization cuts evaluation time by 3–10x

### **2. Accuracy**
- Consistent prompting across models
- Side-by-side comparison prevents human bias
- Detailed logging for reproducibility

### **3. Security**
- User-isolated data (no data leakage)
- API keys stored in session only (never on disk)
- SQLite authentication with role-based access

### **4. Flexibility**
- Supports Anthropic, OpenAI, and Pinecone
- Configurable temperature, max tokens, concurrency
- Export in multiple formats (Excel, JSON, CSV)

### **5. Developer-Friendly**
- Dark UI with carefully chosen fonts (`Syne` + `JetBrains Mono`)
- Responsive design (desktop-first, mobile-compatible)
- Docker support for easy deployment

---

## How to Get Started

### **Installation (5 minutes)**

1. **Clone and install:**
   ```bash
   pip install -r requirements.txt
   ```

2. **Set up your API keys** in `.env`:
   ```
   OPENAI_API_KEY=sk-...
   ANTHROPIC_API_KEY=sk-ant-...
   PINECONE_API_KEY=...
   PINECONE_REGION=us-east-1-aws
   ```

3. **Create a user** via SQLite:
   ```bash
   sqlite3 reflex.db
   INSERT INTO usermodel (username, password, name, role) VALUES ('yourname@1234','1234','Your Name','user');
   ```

4. **Run the app:**
   ```bash
   cd llm_tester
   reflex run
   ```

5. **Open** http://localhost:3000 and log in.

---

### **Docker Deployment (3 minutes)**

```bash
# Build
sudo docker build --no-cache -t llm_tester .

# Run
sudo docker run -p 3000:3000 -p 8000:8000 llm_tester

# Create user (inside container)
sudo docker exec -ti <container-id> /bin/bash
sqlite3 reflex.db
INSERT INTO usermodel (username, password, name, role) VALUES ('yourname@1234','1234','Your Name','user');
```

---

## Usage Walkthrough

### **Scenario 1: Evaluating a New Prompt**

1. Go to the **Prompt tab**
2. Enter your prompt (e.g., *"Summarize this article in 3 bullet points"*)
3. Select vendor (Anthropic / OpenAI) and model
4. Set temperature (0.0 = deterministic, 2.0 = creative)
5. Click **Submit**
6. Review response, stats, and cost
7. Rate the response and add notes

---

### **Scenario 2: Comparing Models**

1. Go to the **Compare tab**
2. Enter the same prompt
3. Select multiple models (Claude 3.5, GPT-4o, etc.)
4. View side-by-side responses
5. Check latency and cost differences
6. Save your comparison in **History** for later reference

---


### **Scenario 3: Finding Relevant Training Data**

1. Go to the **Retrieval tab**
2. Load your Pinecone index (e.g., "customer-docs")
3. Enter a search query (e.g., *"How to reset my password?"*)
4. Toggle MMR for diversity
5. View results in the structured table
6. Export as CSV for preprocessing

---

### **Scenario 4: Building a Prompt Library**

1. Test and refine a prompt in the **Prompt tab**
2. When satisfied, click **Save as Template**
3. Name it (e.g., *"Customer Support - Issue Summary"*)
4. Tag it with a category
5. Reuse it anytime by selecting from the **Templates** dropdown

---

## Configuration & Settings

All major parameters are adjustable:

| Setting | Range | Default | Notes |
|---------|-------|---------|-------|
| **Temperature** | 0.0 – 2.0 | 0.0001 | 0 = deterministic, 2 = very creative |
| **Max Tokens** | 1 – 4096 | 1024 | Hard cap on response length |
| **Concurrency** | 1 – 10 | 3 | Parallel requests for batch processing |
| **MMR** | On/Off | Off | For retrieval diversity |

---

## API Key Management

Your API keys are **never stored on disk**. They're held in your browser session only.

| Provider | Where to Get | Cost |
|----------|-------------|------|
| **Anthropic** | https://console.anthropic.com/settings/keys | Pay per token |
| **OpenAI** | https://platform.openai.com/api-keys | Pay per token |
| **Pinecone** | https://www.pinecone.io | As per pricing documentation |

---

## Real-World Use Cases

### **Research Labs**
Test hypotheses on LLMs, log all experiments, compare across models, publish findings with full reproducibility.

### **Content Teams**
Batch-generate product descriptions, compare tone/style across models, save best-performing templates.

### **AI/ML Engineers**
Fine-tune retrieval systems, benchmark model performance, track improvements over time.

### **Customer Support**
Evaluate response templates before deployment, compare model accuracy on real support tickets, audit model behavior.

### **Product Managers**
A/B test AI features, understand model trade-offs (speed vs. cost vs. quality), demo to stakeholders.

---

## Advanced Tips

### **Tip 1: Use Templates for Consistency**
Save your best-performing prompts as templates. This ensures every experiment uses the same baseline, reducing noise in comparisons.

### **Tip 2: Export History for Analysis**
Export your full query history (available in the History tab) to CSV. Run statistical analysis on latency, cost, or model accuracy.

### **Tip 3: Filter Retrieval with Metadata**
If your Pinecone vectors have metadata (e.g., `category`, `date`), use metadata filters to narrow results before MMR ranking. This speeds up retrieval and improves relevance.

### **Tip 4: Monitor Cost in Real-Time**
LLM Tester auto-calculates cost per request. Use the History tab to identify expensive models or prompts, then optimize accordingly.

---

## What's Coming Next?

While **batch processing** is currently in use but not featured in this blog, future roadmap items may include:
- Fine-tuning pipeline integration
- Custom model support
- Collaboration & team features
- Advanced analytics dashboard
- Scheduled batch jobs

---

## Conclusion

LLM Tester brings structure, speed, and transparency to model evaluation. Whether you're running a one-off comparison or orchestrating hundreds of experiments, it's built to scale with your needs—without sacrificing ease of use.

**Ready to get started?** Install it in 5 minutes, log in, and run your first prompt. You'll never go back to scattered CLI calls and spreadsheets.

---

## Resources

- **Docs**: Run locally at http://localhost:3000
- **Issues**: Report bugs or feature requests
- **Support**: Contact kunal.singh@bloomai.co

---

**Happy testing! 🚀**