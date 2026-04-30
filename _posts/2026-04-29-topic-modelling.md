---
layout: post
title: We did topic modeling on 5.2k+ articles — here's how it went
date: 2026-04-29
description:  
tags: [genai,llm,nlp]
categories: [nlp,Content Intelligence] 
giscus_comments: false
related_posts: false
featured: true
toc:
  sidebar: left
pretty_table: true
---

**Author:** Devansh Dutta  
**Category:** Content Intelligence  
**Read Time:** 8 min read

---

## What's this about?

We had a big pile of articles from Asset management firms — thousands of them. And we wanted to know: what topics are they actually writing about? Not just the obvious ones, but the small, niche ones that barely anyone covers.

Reading them manually was obviously not happening. So, we built a Python pipeline that reads all of them, groups similar ones together, and tells us which topics only have a tiny number of articles — those are the niche ones we actually care about.

![Topic modelling overview ](/assets/img/topic-modeling-images/topic_modelling_1.png)

---

## How the pipeline works

It's 4 scripts that run one after another. Here's what each one does:

### 1. Remove duplicates

Same article sometimes shows up more than once. We clean up the title, lowercase it, and drop any repeats. Simple but important — duplicates mess up the topic counts.

### 2. Clean the text

These articles come with a lot of junk — cookie notices, "read more" buttons, legal disclaimers, brand names. We strip all that out using regex patterns so we're left with the actual content.

### 3. Turn articles into numbers (embeddings)

We use a model called **all-MiniLM-L6-v2** to convert each article into a list of numbers that captures what it means. Articles about similar things end up with similar numbers. We cache these so we don't have to recompute every time.

### 4. Cluster them into topics

We use **UMAP** to squish the numbers down to a manageable size, then **HDBSCAN** to find natural groups. **BERTopic** wraps all of this and also picks keywords to describe each group. Any article that doesn't fit anywhere gets labeled as noise (topic -1).

---

## Things that didn't work great

### ⚠️ ~15–20% of articles ended up as noise

Mostly short articles or ones where cleaning removed too much content. The model couldn't figure out where to put them.

### ⚠️ Words like "investor", "market", "risk" kept showing up everywhere

These words are in literally every article in this space, so they made the topic labels useless. We had to build a custom stopword list and remove them before running the model.

### ⚠️ Too many near-identical topics

The model kept creating separate clusters for things that were basically the same topic. We used `reduce_topics(auto)` to merge them back together.

### ⚠️ Embedding cache kept breaking

When we added or removed articles mid-run, the cached file had the wrong number of rows. We added a check at the start — if the count doesn't match, delete and regenerate.

---

## What we got out of it

### Output Files

| File | What's in it |
|------|-------------|
| **topic mapping** | Every article with its topic ID and a confidence score |
| **topic summary** | All 66 topics with keyword lists and doc counts |
| **topic representation** | Just the niche topics (30 articles or fewer each) |
| **cleaned documents** | The cleaned text we actually fed into the model |

---

## What we learned

Most articles cluster into the obvious big topics — things that everyone writes about. The interesting part is the small clusters: topics with only 10–25 articles. Those are either something new that's starting to get attention, or something very niche that not many people are covering yet.

That's what we were trying to find. And it actually works — the niche topics we got out are genuinely interesting and not something you'd spot by just browsing.

---

## Built with

| Tool | Purpose |
|------|---------|
| **BERTopic** | Topic modeling framework |
| **Sentence Transformers** | Text embeddings |
| **UMAP** | Dimensionality reduction |
| **HDBSCAN** | Clustering algorithm |
| **KeyBERT** | Keyword extraction |
| **scikit-learn** | ML utilities |
| **pandas** | Data processing |
| **Python** | Language |

---

## What's next

- Use an LLM to write better topic names instead of the auto-generated ones
- Group the 66 topics into broader themes so it's easier to browse
- Build a simple dashboard to explore topics visually
- Track how topics change over time — what's growing, what's fading

