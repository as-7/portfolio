---
author: Aakash
pubDatetime: 2026-07-25T17:58:11Z
title: "I Ship LLM Systems for a Living. I Can't Derive Backprop."
slug: i-ship-llm-systems-cant-derive-backprop
featured: true
draft: false
tags:
  - ai-engineering
  - learning-in-public
  - llm
  - genai
description: "Four+ years of engineering, two of them building production AI — an honest inventory of what I still don't know, and the 22-month plan I'm running in public to fix it."
---

I've shipped retrieval systems that answer questions over nearly 300,000 documents.

I also couldn't compute a gradient if you asked me to. Not "I'd need an hour to refresh" — I mean I don't know how backpropagation works.

Both of those things are true, and the second one has started to bother me.

**So why does it bother me?**

Because I'm proud of what I've built, not embarrassed by it. Those systems run reliably in production, used daily by thousands of people at a large enterprise client — real work, with real consequences when it breaks.

But building them is exactly what made me want to understand them even more. Two years of shipping AI has left me more curious about this field than when I started, not less. And curiosity that never converts into understanding just becomes a ceiling.

## The gap

There are two kinds of people building with AI right now.

The first kind composes. We pick an embedding model because a benchmark said so, wire up a vector store, add a reranker when recall is bad, swap in a bigger model when quality is bad, and ship. It works. I've been doing it for two years and the systems I've built are used every day.

The second kind understands what they're composing. When retrieval plateaus, they reason about _why_ this embedding model collapses distinctions that matter in this corpus. When someone asks "should we fine-tune?", they don't have an opinion — they have an experiment. When inference costs too much, they know which knob moves what, because they've measured it.

Here's the contrast, in code. This is roughly what I write on a normal Tuesday:

```python
# Four lines. Every one is a decision I made by reading someone else's benchmark.
retriever = vectorstore.as_retriever(search_kwargs={"k": 20})
docs = retriever.invoke(query)
reranked = reranker.rerank(query, docs, top_n=10)
answer = llm.invoke(prompt.format(context=reranked, question=query))
```

And this is what I want to be able to write from memory, and defend line by line:

```python
# Also four lines. I didn't write this one — I copied it in to make the point.
def backward(self, dout):
    dx = dout @ self.W.T
    self.dW = self.x.T @ dout
    self.db = dout.sum(axis=0)
    return dx
```

I want to be the second kind of engineer — not for a title, but because the first kind has a ceiling and I've hit it. Concretely, hitting it looks like this:

- I know prompt caching cuts input costs substantially. I have never measured the break-even against a self-hosted model myself.
- I've deployed with vLLM. I couldn't explain PagedAttention's memory layout at a whiteboard without hand-waving.
- I can build a RAG pipeline in an afternoon. I can't tell you, from my own data, whether contextual retrieval earns its indexing cost on a given corpus.
- I read papers. I understand them at "I could explain the abstract," not "I could reimplement Section 3."

Every one of those is the gap between using a thing and knowing it. Under production pressure, that gap is where you get found out.

## Why do this publicly

Three reasons, in order of how much I believe them.

**1. A private plan costs nothing to abandon.** I've written versions of this plan before. They lived in a notes app and died there, and nobody noticed — including me. This one has witnesses.

**2. Writing is the compression test.** You cannot write 2,000 clear words about something you only half-understand: the holes surface as vagueness, and you feel them while typing. Publishing closes the loop, because someone who knows more than me gets to say I'm wrong. That correction is cheap here. It's expensive in production.

**3. Invisible work doesn't compound.** Four years of shipping, two of them on AI systems, and almost none of it is public. That's a mistake I'd like to stop making.

There's a fourth, less noble reason: I want to be findable by people doing this work. Writing is the cheapest way I know to do that.

## Where I actually am

The honest inventory. What I can do today, with production systems behind it:

- **Retrieval at real scale.** RAG over nearly 300,000 documents — hybrid retrieval, reranking, and chunking strategies that survive messy enterprise data rather than clean PDFs.
- **Agentic systems.** Agent orchestration with Google's ADK, tool use, MCP servers, and state and memory that persist across turns.
- **LLMOps.** Tracing and cost attribution with Datadog, plus eval scaffolding on RAGAS-style metrics and custom ones where those didn't fit.
- **Text-to-SQL** against real schemas, with the guardrails that requires.
- **The unglamorous half.** FastAPI, Redis, Celery, Postgres, Docker, Kubernetes, three clouds. I've been on call for things I built. This matters more than it sounds — most AI systems fail as distributed systems, not as models.

What I can't do, stated plainly:

- Derive backprop, or compute a gradient by hand (see above).
- Implement a transformer from scratch and defend every line of it.
- Justify a fine-tuning decision with lift I measured myself.
- Design a clean object model under time pressure, with the vocabulary to explain why it's clean.
- Solve hard algorithmic problems reliably at speed.
- Explain distributed training beyond the conceptual level.
- Point to a single public artifact that demonstrates any of the above.

That last one is the reason for this post.

## The plan

Twenty-two months. Fifteen to twenty-five hours a week, alongside a full-time job — which is the real constraint, and the reason the timeline is long rather than heroic. Three phases:

```
Phase 1 — FOUNDATIONS          Phase 2 — BUILD & DEPTH        Phase 3 — SPECIALIZE
months 1-5                     months 6-12                    months 13-22

math -> classical ML           3 flagship projects            preference tuning
DL internals                   fine-tuning + evals            inference economics
transformer from scratch       MCP + multimodal               OSS + paper reproduction

gate: can I defend             gate: is any of this           gate: am I known for
every line of it?              actually good?                 one specific thing?
```

**Phase 1 — Foundations (months 1–5).** Linear algebra, calculus, probability — enough to read papers and reason about training, not a math degree. Classical ML implemented from scratch in NumPy. Then Karpathy's sequence: micrograd, makemore, nanoGPT, and if I get there, nanochat. The bar isn't "I watched the videos." The bar is a transformer I wrote whose every line I can defend.

**Phase 2 — Build and depth (months 6–12).** Three flagship projects, each public, deployed, and measured: a RAG system at a scale that breaks the naive design, an agent platform with trajectory evals and real users, and one differentiator I haven't chosen yet. Alongside them: a real fine-tune with measured lift, MCP servers, and an eval harness running in CI.

**Phase 3 — Specialize (months 13–22).** Preference tuning hands-on, inference economics I've benchmarked myself, open-source contributions, and a paper reproduction where I document honestly what didn't reproduce.

Running underneath all of it are six disciplines I think a senior engineer owes their team, whether or not anyone ever interviews them on it: algorithms, low-level design, system design (including ML and LLM system design, where my day job and my study meet), CS fundamentals, AI depth, and the ability to explain any of it to someone who doesn't share my context. Not all six at once — one or two in active focus at a time, the rest ticking over. 

## What I'll ship

Roughly monthly, for twenty-two months:

- **10+ posts in the first year** — technical, with numbers and code. Not summaries of things you could read at the source.
- **A transformer from scratch**, with a writeup of what surprised me.
- **Three deployed projects** with public repos and honest results tables, including the experiments that didn't work.
- **A fine-tuned model** with base-versus-tuned numbers and a clear verdict on when the tuning was worth it.
- **MCP servers** for tools I actually use.
- **An eval harness** good enough that I'd trust it to gate a release.

The rule I'm holding myself to: every month ships something public. Courses are inputs; artifacts are output. A month that ends with no artifact failed, regardless of how much I felt I learned.

## Honest outcome distribution

I've read enough posts like this to know how they usually end: a confident timeline, then silence around month four. So here's my real estimate.

**Most likely (~50%): I finish the substance, late.** Phases slip, the flagship projects take longer than planned, and a month or two goes to life instead of this. I end up with the depth and most of the artifacts, on a longer clock. This is the median outcome, and it's a good one.

**Likely (~30%): it runs roughly as designed.** Not through discipline, but because the plan is built to survive bad months — one active focus at a time, extend rather than abandon, and a hard rule that sleep and training are not raidable for study hours.

**Possible (~20%): it partly stalls.** If it does, I know exactly how: the tutorial trap. Eight finished courses, one shipped project. I know because that's how I've approached learning for most of my career — plenty consumed, almost nothing built from it.

What I won't do is quietly stop and let the archive imply I finished.

## Monthly check-ins

At the end of each month: a short post covering what shipped, what I got wrong, and what's next — including the months where the answer is "less than I planned." Those are more useful to read than the wins, and considerably more useful to write.

If you're doing this work — especially if you've gone deep on evals, retrieval at scale, or inference economics — I'd like to hear where I'm wrong. That's the whole reason to do this here rather than in a private notebook.

Month one starts now. The first post on the math is next.