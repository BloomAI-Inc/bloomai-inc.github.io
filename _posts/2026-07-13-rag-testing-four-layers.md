---
layout: post
title: "A RAG System Fails in Four Places, Not One"
date: 2026-07-13
description: A model that passes every jailbreak test can still hand back a confidently wrong answer - because in a RAG system, the model is just one of four places things break. The model, the retrieval path, the generated answer, and behavior under load each fail in ways the others can't see, and each needs its own test.
author: Devansh Dutta
tags: [ai, rag, testing, llm, retrieval, trust, engineering]
category: AI
categories: AI
giscus_comments: false
related_posts: false
featured: false
toc:
  sidebar: left
pretty_table: true
---


*By Devansh · July 2026 · ~8 min read*

Every retrieval-augmented generation (RAG) demo follows the same script: ask a financial question, watch a well-sourced answer appear a few seconds later, feel reassured. The part that never makes it into the demo is what happens when the load test hits forty concurrent users, or when the question is phrased slightly differently, or when the retriever quietly hands the model the wrong paragraph and the model — fluent as ever — builds a confident answer out of it anyway.

That's not a model problem. A red-team pass against the language model will pass cleanly and tell you nothing about any of it. If you're shipping a RAG system rather than a chatbot demo, testing the model is necessary but nowhere near sufficient — and treating it as the whole job is how "confidently wrong" gets into production.

## The failure mode that matters

A model that's been jailbroken is an obvious, visible failure. A model that leaks a system prompt is loud and easy to catch. What's much harder to catch is a RAG system that retrieves the wrong context, or ignores the right context, and still produces a clean, well-formatted, entirely wrong answer. Nothing about the output signals that anything went wrong. A financial analyst asking about a client's portfolio has no way to tell, from the answer alone, whether it came from the correct source document or from the model quietly filling a gap with something plausible.

This is the reason a security audit that stops at the model is answering the wrong question. It confirms the model won't misbehave when attacked directly. It says nothing about whether the retrieval pipeline hands over the right paragraph, whether the generated answer is faithful to that paragraph, or whether any of this holds up once real traffic hits it. Those are three additional, independent ways a production RAG system can go wrong — and in practice, they're where most of the actual damage happens, not in the model's willingness to be jailbroken.

![Testing only the model versus testing the whole system](/assets/img/rag-testing-four-layers/rag-compare.png)
<br/>
<em>A clean model-only test tells you the model resisted attack. It tells you nothing about what it retrieved or what it answered.</em>

## Four surfaces, tested independently

Once you accept that the model is only one of the places this can break, the shape of the testing problem changes. Each surface gets its own pass, with its own tooling, because each one fails in a genuinely different way:

|               | What it's really asking                             | How it fails                                                                  |
| ------------- | --------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Model**     | Can it be manipulated, jailbroken, or made to leak? | Prompt injection, credential leakage, hallucination under pressure            |
| **Retrieval** | Is it handing the model the right context?          | Wrong chunks, missed authorization boundaries, poisoned or duplicated content |
| **Answer**    | Is what it generated actually correct?              | Faithful to the wrong thing, incomplete, contradicts the source               |
| **Load**      | Does it hold up under real traffic?                 | Latency creep, silent failures, no ceiling on abuse                           |

None of these four questions can be answered by testing any of the others. A retrieval pipeline can be flawless and the model can still ignore what it was given. A model can be perfectly well-behaved under adversarial attack and still summarize a poisoned document as fact if nobody checked the document made it in cleanly. Treating this as one test — "is the AI safe?" — collapses four distinct engineering questions into one, and the ones that get silently dropped are usually retrieval and answer quality, because they're less dramatic to demo than a jailbreak.

## What each pass is actually checking

**The model, in isolation.** Before retrieval enters the picture at all, the model gets probed directly — hand-written attacks mapped to specific attack categories (prompt injection, credential extraction, system-prompt leakage, forced hallucination, denial-of-service via flooding) alongside a much larger set of automated adversarial runs that combine several attack vulnerability classes with several delivery mechanisms — roleplay framing, encoded payloads, multi-turn escalation. The result that matters here isn't a list of transcripts; it's a single number, an attack success rate, with a target of zero.

**The retrieval path.** This pass never looks at the final sentence — it checks the raw ingredients the model was handed before it started writing, by querying the vector store directly rather than going through the model at all. Does a known, unambiguous question actually retrieve the right document? Does an out-of-scope question come back empty instead of triggering a confident guess? Can the retrieval ranking be manipulated by wording a query a certain way? Is the same chunk coming back more than once? One of these checks runs first, as a positive control: a known-answerable question has to succeed before any other result in the pass can be trusted, because a refusal on a hard question looks identical to a broken endpoint from the outside.

**The generated answer.** Correct retrieval doesn't guarantee a correct answer — the model can still ignore the context it was given. This is graded with an LLM-as-judge, but not by comparing prose to a reference answer, which quietly punishes a correct-but-longer response against a short one. Instead, each ground-truth answer is broken into individual factual claims, and the judge labels every claim independently as contradicted, covered, or simply missing. Length stops being a factor. Alongside that, standard groundedness metrics check whether the answer stays inside what was actually retrieved, and whether the system declines gracefully on questions it has no business answering, rather than filling the gap.

**Behavior under load.** All three of the passes above assume one request at a time. The last pass simulates the traffic pattern that actually matters — dozens of people asking things concurrently — and checks round-trip latency, failure rate, throughput, and what happens when the traffic spikes all at once, the way it does around market open.

![Four testing layers as standing infrastructure](/assets/img/rag-testing-four-layers/rag-layers.png)
<br/>
<em>None of the four is a stage you complete once. Each keeps running as the system evolves — retrieval logic changes, the model gets swapped, load patterns shift.</em>


## The sanity check that has to come first

One detail is easy to skip and expensive to skip: before trusting any "the system correctly refused this" result, there has to be proof the system can answer at all. A refusal and a broken endpoint look identical from the outside. So the very first check in the retrieval pass is deliberately boring — a plainly answerable, known-domain question — and every other result in that pass is only meaningful once that one has passed. Skipping this step is how a team ends up celebrating a security win that's actually just an outage.

## What the numbers are actually for

None of this is abstract. Run against a live financial-research RAG endpoint, this approach turned up a genuinely useful, specific finding that a model-only audit would never have surfaced: the model itself held up cleanly against dozens of adversarial probes with a zero percent attack success rate, and retrieval was accurate and unpoisoned across every check — but answer quality on broad, multi-part questions came in measurably below target, not because the model was hallucinating (it wasn't, across the board), but because retrieval was pulling in some noise on those particular questions and the resulting answers stayed a little too generic to fully cover what was asked.

That is precisely the kind of finding a jailbreak test cannot produce. It's not a security hole. It's a quality gap, located specifically in the seam between retrieval and generation, and it only becomes visible once retrieval and answer quality are tested as their own surfaces instead of being assumed to be fine because the model behaved.

A similar pattern showed up in performance testing: latency stayed comfortably within target under simulated load, but a burst of concurrent requests revealed there was no rate limiting in place at all. Nothing was on fire. But "no rate limiting" is not a finding you get from red-teaming the model, either — it only shows up once you've built a pass whose entire job is asking what happens under real traffic.

## The takeaway

If you're shipping a RAG system, the language model is genuinely the easy part to get right — modern models are already good at generating fluent, well-sourced-sounding text once they have context, and adversarial testing against the model in isolation is mature, well-tooled, and fast to run. The retrieval pipeline, the faithfulness of the generated answer, and the system's behavior under real load are what actually determine whether the thing is trustworthy in production, and none of the three get checked by testing the model harder. They need their own pass, their own tooling, and their own pass/fail bar — because each one fails in a way the others can't see.
