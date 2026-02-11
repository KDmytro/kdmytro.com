---
title: "LLM-as-Judge is Evolving. Meet Agent-as-Judge."
date: 2026-02-10
draft: false
description: "Why evaluating agents requires agents — from passive scoring to active investigation"
tags: ["ai-agents", "evaluation", "llm", "agent-as-judge"]
---

LLM-as-Judge was a breakthrough - using LLMs to evaluate LLM outputs at scale. But as we move from simple chatbots to complex agents, the paradigm is breaking.

Agents do multi-step reasoning, execute tools, and make decisions over time. Evaluating just the final output isn't enough anymore. You cannot judge a journey by looking only at the destination. We need judges that can investigate the process, verify claims, and adapt their criteria.

Enter **Agent-as-Judge**.

---

## The Problem with LLM-as-Judge

LLM-as-Judge (using GPT-4 or Claude to score a response) has been instrumental in scaling evaluation, often achieving 80%+ correlation with human preferences. But it operates on a "Snapshot" basis: it looks at a static input and a static output.

As systems evolved from chatbots to agents, the cracks widened.

**Where it breaks down:**

1. **Surface-level scoring:** An LLM judge assesses based on linguistic patterns. It asks, "Does this *sound* correct?" rather than "Is this correct?" It cannot verify if a database was actually updated or if the generated code actually compiles.
2. **Process blindness:** An agent might hallucinate a tool call or reach a correct answer through flawed logic - a "lucky guess." A standard judge sees the correct final answer and assigns a high score, masking a critical reliability flaw that will surface in production.
3. **Silent side-effects:** Agents act on the world. If an agent is tasked with "Cancel my subscription," an LLM judge only reads the text "Subscription cancelled." It has no way of knowing if the API call actually failed silently.
4. **Cognitive overload:** LLM judges favor verbosity and succumb to position bias. When faced with complex agent traces (thought-action-observation loops) a single prompt context window is easily overwhelmed, leading to coarse-grained, unreliable scores.

The fundamental problem: **LLM-as-Judge is pattern matching on text without grounding in reality.**

---

## What Makes Agent-as-Judge Different

Agent-as-Judge isn't just "better prompting." It's a fundamental architectural shift from **passive observation** to **active investigation**.

| Feature | LLM-as-Judge | Agent-as-Judge |
| :--- | :--- | :--- |
| **Role** | Passive Observer | Active Investigator |
| **Scope** | Final Output Only | Full Trajectory (Thought + Action) |
| **Verification** | None (Text-based) | Tool-assisted (Code/API/Search) |
| **Criteria** | Static Rubric | Adaptive/Generated Rubric |
| **Perspective** | Single Point of View | Multi-Agent Debate |

### Tools Are the Differentiator

An Agent-as-Judge gets access to the same (or superior) tools as the agent it is evaluating. This is what turns it from a critic into an investigator.

**The Scenario: The SQL Trap**
Imagine an agent asked to "Count the active users in California."
- **The Agent** generates a SQL query that runs but uses the wrong table join. It outputs: "There are 4,200 active users."
- **The LLM-as-Judge** sees a confident, well-formatted number. It rates the response **5/5**.
- **The Agent-as-Judge** sees the SQL query in the trace. It runs the query against a sandbox database using a `verify_sql` tool. The join is incorrect - 0 rows returned. It rates the response **1/5** and flags the exact logic error.

But tools alone aren't enough. The architecture has matured through several key ideas:

- **Multi-agent debate.** Instead of one judge, a panel of LLM evaluators with diverse perspectives (security, correctness, helpfulness) debate before reaching a verdict. Different viewpoints catch different failure modes.
- **Auto-generated personas.** Rather than hand-crafting evaluator roles, the system reads domain documents - research papers, guidelines, case law - and synthesizes specialist personas automatically. Evaluation generalizes across domains without manual setup.
- **Meta-judges.** A higher-level judge watches the evaluators themselves. If a rubric doesn't fit the task, the meta-judge critiques the criteria. If an evaluator is consistently harsh or hallucinates rules, it intervenes. The system becomes self-correcting.

Research backs this up. In the DevAI benchmark for automated software development, Agent-as-Judge achieved **90% agreement** with human experts, compared to 70% for standard LLM-as-Judge while reducing evaluation costs by 97% compared to human labor.

---

## In Production: MLflow's Implementation

This is no longer just academic theory. **MLflow** has shipped Agent-as-Judge for production environments. Their judge agent navigates execution traces using a tool-based protocol—retrieving trace metadata, walking the span hierarchy (what called what, in what order), and searching for specific tool inputs and outputs via regex.

The judge can see exactly what tools the agent called, with what arguments, and trace *how* the answer was reached. In MLflow's benchmarks, their agent-based judges matched human agreement at significantly higher rates than single LLM judges (they report 99.7% vs 69%, though these numbers come from their own evaluation on their own product, so treat them as directional rather than universal).

The key insight from their implementation: giving the judge access to structured trace data, not just the raw text log, dramatically improves evaluation quality.

---

## The Trade-off: The Cost of Accuracy

We must address the elephant in the room: **Latency and Cost.**

Agent-as-Judge is computationally expensive. A single evaluation might require the judge to plan, execute multiple tool calls, and synthesize the results. This is 10x to 50x more expensive than a simple LLM-as-Judge pass.

**The mitigation strategy:** Don't run deep evaluation on everything.

1. **Online (100% of traffic):** Lightweight code-based assertions (e.g., "Is the JSON valid?", "Did the tool return a 200?") and cheap LLM judges for tone and safety.
2. **Offline (5-10% sampled):** Asynchronously route a sample of traces to the full Agent-as-Judge pipeline for deep verification.
3. **Golden datasets:** Use Agent-as-Judge to curate high-quality labeled data, then fine-tune smaller, faster models to approximate its judgments at scale.

This tiered approach gives you the depth of agent evaluation where it matters without blowing up your inference bill.

---

## Where This Is Heading

The survey literature points to two directions worth watching:

1. **Context-aware rubric generation.** Future judges won't need predefined criteria. They'll analyze the user's prompt, infer the intent, and synthesize evaluation dimensions on the fly, catching failure modes "not anticipated during design."
2. **Adaptive granularity.** Simple tasks get holistic pass/fail evaluation. Complex multi-step tasks get automatically decomposed into fine-grained sub-rubrics. The judge decides how deep to go based on the complexity of the trace.

The vision is an evaluation system that learns your standards, evolves as your product changes, and maintains alignment over time without constant manual tuning.

---

## Conclusion

LLM-as-Judge was a necessary first step - it scaled evaluation beyond human capacity. But it was always a vibe check, a text-level pattern match with no way to verify what actually happened.

Agent-as-Judge closes that gap. With **tools**, the judge becomes an investigator. With **planning**, complex evaluations decompose into verifiable steps. With **debate**, blind spots get surfaced. With **meta-judges**, the system stays calibrated.

The era of "black box" evaluation is ending. The era of trajectory analysis ("glass box", tool-assisted, self-correcting) is beginning.

If you're building agents, start with the tiered evaluation strategy: assertions and cheap judges online, Agent-as-Judge offline on sampled traces. You don't need the full architecture on day one. But you need to be moving in this direction, because your agents are already outrunning your ability to evaluate them.

---

*Further reading:*
- [A Survey on Agent-as-a-Judge](https://arxiv.org/abs/2601.05111) — Comprehensive survey of the paradigm
- [Agent-as-a-Judge: Evaluate Agents with Agents](https://arxiv.org/abs/2410.10934) — Foundational paper introducing the framework
- [MLflow Agent-as-Judge Docs](https://mlflow.org/docs/latest/genai/eval-monitor/scorers/llm-judge/agentic-overview/) — Practical implementation guide
