---
title: "How Do You Actually Evaluate an AI Research Agent?"
date: 2025-01-14
draft: false
description: "Beyond 'it runs' — figuring out how to know if your AI agent is actually good"
tags: ["ai-agents", "evaluation", "llm", "grep-ai"]
---

We're building expert AI research agents at [Grep.ai](https://grep.ai) — think due diligence reports, business research, compliance checks. The kind of work where you need depth, accuracy, and real sources.

Getting the agent to *run* was the easy part. Making sure it's actually *good*? That's where things get interesting.

## The problem with "it works"

Our initial eval setup was basic: does the agent complete the task? Does it return a report? Does it cite sources? Check, check, check.

But that doesn't tell you if the report is *correct*. Or if it's thorough. Or if it's better than what a human analyst would produce — or what OpenAI's Deep Research or Gemini would give you for the same query.

We're building domain-expert agents. "Expert" is a strong claim. How do you prove it?

## Single-answer benchmarks don't fit

The standard approach for evaluating AI is benchmarks like [GAIA](https://arxiv.org/abs/2311.12983) or [Humanity's Last Exam](https://arxiv.org/abs/2501.14249) — give the model a question, check if the answer is correct. Clean and simple.

But our agents don't produce answers. They produce 10-page research reports. There's no single "correct" response to compare against. The output is long-form, multi-faceted, and inherently subjective in some dimensions.

So I went looking for how others solve this.

## What I found: the decompose-then-verify approach

The most rigorous methodology for evaluating long-form content comes from [FActScore](https://arxiv.org/abs/2305.14251) and Google's [SAFE](https://arxiv.org/abs/2403.18802) framework:

1. **Break the report into atomic claims** — individual facts that can be verified independently
2. **Verify each claim** against authoritative sources (Google Search, domain databases, etc.)
3. **Calculate precision** — what percentage of claims are actually supported?

This is how Google found that ChatGPT only achieves ~58% factual accuracy on biography generation when you actually check the claims. Sobering.

For research reports, [VeriScore](https://arxiv.org/abs/2406.19276) adds a useful refinement: distinguishing *verifiable* claims from subjective analysis. You can't fact-check an opinion, but you can check whether the facts supporting that opinion are accurate.

## LLM-as-judge for the subjective stuff

Factuality is necessary but not sufficient. A report can be 100% factually accurate and still be shallow, poorly organized, or miss the point entirely.

For these "softer" dimensions, [G-Eval](https://arxiv.org/abs/2303.16634) provides a solid framework:

- Define explicit criteria and rubrics for each dimension (depth, coherence, instruction-following)
- Have the LLM reason through its evaluation before scoring
- Use a different model family than your agent to avoid self-preference bias

The key insight: evaluate one dimension per prompt. Asking "rate this report's quality" gives you noise. Asking "does this report consider multiple perspectives on the topic?" gives you signal.

## The benchmarks that actually exist

For comparing against commercial solutions, [DeepResearch Bench](https://deepresearch-bench.github.io/) is the closest to what we need — 100 PhD-level research tasks evaluated on completeness, depth, and citation accuracy. Current leaderboard: Gemini Deep Research at 48.88, OpenAI at 46.98.

For domain-specific evaluation:
- **Legal**: [LegalBench](https://github.com/HazyResearch/legalbench) — 162 tasks across 6 reasoning types
- **Medical**: [MedXpertQA](https://arxiv.org/abs/2501.18362) — 4,460 questions across 17 specialties  
- **Financial**: [PIXIU/FinBen](https://github.com/The-FinAI/PIXIU) — 24 tasks including stock trading

Notable gap: no standardized benchmarks for compliance/KYC/AML. That's our domain. We'll probably have to build our own.

## What we're actually going to do

Based on this research, here's the eval stack I'm planning:

**Tier 1 — Automated (every run)**
- Task completion and format compliance
- Claim extraction + verification via search (FActScore-style)
- Citation validity checks (do sources exist? do they support the claims?)

**Tier 2 — LLM-as-judge (sampled)**
- Depth of analysis
- Instruction following  
- Reasoning coherence
- Domain-appropriate caveats

**Tier 3 — Human expert (calibration + edge cases)**
- Build gold-standard dataset for calibrating automated evals
- Review cases where Tier 1 and Tier 2 disagree
- Periodic audits on production outputs

**Tier 4 — Competitive benchmarking**
- Same queries to our agents vs. OpenAI/Gemini/Perplexity
- Track accuracy, depth, cost, and latency
- Pareto frontier analysis (quality vs. cost)

## The efficiency question

One thing that surprised me in the research: [Anthropic found](https://www.anthropic.com/engineering/multi-agent-research-system) that token usage explains 80% of performance variance on browsing benchmarks. More tokens = better results, up to a point.

This means you can't just compare accuracy — you need quality-adjusted cost metrics. A system that's 10% more accurate but 5x more expensive might not be the right tradeoff.

We're tracking cost-per-query alongside accuracy. The goal is finding the Pareto frontier: maximum quality for a given budget.

## Still figuring it out

This is a work in progress. The eval framework will evolve as we learn what actually predicts real-world usefulness.

If you're working on similar problems — evaluating AI agents that produce long-form outputs, especially in regulated domains — I'd love to compare notes. Reach out on [LinkedIn](https://www.linkedin.com/in/kdmytro/).

---

*I'm building [Grep.ai](https://grep.ai), an AI-powered business due diligence platform. We're hiring. If turning AI demos into production systems sounds interesting, let's talk.*
