# Chapter 9 — Security and guardrails

Grounding for the security part of **Area 5**. Source: *Building Generative AI Services with FastAPI*, Ch 9.

## Guardrails overview

Guardrails are detective controls that guide the application toward intended outcomes. I/O guardrails verify data entering the model and outputs sent to users or downstream systems. You need not build from scratch: open-source frameworks exist (NVIDIA NeMo Guardrails, LLM-Guard, Guardrails AI) and commercial ones (OpenAI Moderation API, Azure AI Content Safety, Google), each with trade-offs (framework-specific languages, added latency, dependency bloat; commercial ones closed or thin on quality metrics). Guardrails are an active research area; AI-backed attacks can still bypass them, so it is an ongoing arms race. Rate limiting on public and model-backed endpoints sits here too.

## Input guardrails (stop bad content reaching the model)

- **Topical:** steer away from off-topic or sensitive content (politics, explicit).
- **Direct prompt injection / jailbreaking:** stop users overriding or revealing system prompts and secrets. Longer input is more prone.
- **Indirect prompt injection:** reject malicious content from external sources (files, websites, images) that can confuse the model or cause remote code execution downstream. Payloads may be hidden or encoded (invisible characters, doc overrides, scripts in URLs or transcripts). Sanitise.
- **Moderation:** enforce brand, legal, and branding rules; flag profanity, competitor mentions, explicit content, PII, self-harm.
- **Attribute:** validate input properties (query length, file size, choices, range, format, structure).
- Combine with content sanitisers. DIY guardrails can start from advanced prompt engineering in the system prompt, or use auto-evaluators (AI models).

## Running guardrails async

AI guardrails make multiple model API calls per query, so run them **in parallel to inference** (Ch 5) to protect UX. Pattern: launch guardrail and generation as asyncio tasks with `asyncio.wait`, returning as soon as one completes; if the guardrail triggers, cancel generation and return a hard-coded response (log it, notify); otherwise inject the model response through dependency injection. Watch for provider rate-limiting/throttling from the extra calls; request higher limits or slow the call rate if needed.

## Limits of guardrails

They are probabilistic: still vulnerable to injection and jailbreak, and can over-refuse valid queries (false positives that hurt UX). Mitigate by combining with rules-based or traditional ML detection, and consider judging only the latest message to avoid long-conversation confusion. Design against the trade-off between accuracy, latency, and cost.

## Output guardrails (validate generated content before it leaves)

- **Hallucination / fact-checking:** block hallucinations, return canned responses ("I don't know"); in RAG, measure relevancy, coherence, consistency, fluency against ground truth.
- **Moderation:** apply brand/corporate guidelines, filter or rewrite breaches; check readability, toxicity, sentiment, competitor-mention counts.
- **Syntax checks:** verify structure and content; detect-and-retry or handle gracefully; validate JSON schemas and function parameters in function-calling, and tool/agent selection in agentic workflows.
- All rely on a **threshold** to flag invalid responses.

## Thresholds

Tune per use case. More false positives annoy users and reduce usability; more false negatives harm reputation and explode costs (abuse, injection). Assess the worst case of a false negative and decide the trade-off deliberately.

## Moderation via G-Eval

A moderation guardrail can use the G-Eval method with an LLM auto-evaluator. Components: a domain name (content type to moderate), criteria (valid vs invalid), an ordered list of grading instruction steps, and the content scored on a discrete 1 to 5 scale.

## Grading calibration (Area 5, security)

- Judge by exposure. A user-facing or externally exposed endpoint needs input guardrails against prompt injection plus moderation; an internal-only service may need far less. Do not fail the absence of a guardrail framework on a service that takes no untrusted input.
- Guardrails that add latency should run in parallel to inference, not block sequentially.
- Keys and secrets must not be read from string literals in code.
