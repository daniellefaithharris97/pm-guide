# Understanding Evals: The Product Manager’s Guide

Based on the conversation with Hamill Hussein & Shreya Shankar (Lenny’s Podcast, “Building Great AI Products with Evals”)

---

## What Are Evals

Evals (evaluations) are a structured way to measure how well an AI product performs.  
They are essentially data analytics for AI behavior, providing a clear feedback loop to improve quality.

Think of evals as:
- Unit tests for your AI’s reasoning
- Product analytics for your LLM’s responses
- A foundation for building trust and reliability in AI applications

---

## Why Evals Matter

- Every major AI lab (OpenAI, Anthropic, etc.) uses evals to measure and iterate.
- Product builders must learn evals—it’s now a core skill for AI-era PMs.
- The goal isn’t perfection; it’s to actionably improve your product.

---

## The 7-Step Evals Process

### 1. Start with Error Analysis

Look at your application’s logs or “traces” (AI conversations, interactions, outputs).  
Identify where things go wrong and write notes, called open codes.

Examples:
- “AI said there’s a virtual tour—but none exists.”
- “Text message cut off mid-sentence.”
- “Didn’t hand off to a human when needed.”

This is the manual discovery phase—mapping failure modes by hand before automating anything.

---

### 2. Appoint a Benevolent Dictator

One person (usually the PM or domain expert) should own this phase.  
They act as the benevolent dictator, deciding what counts as good or bad behavior.

Why:
- Committees slow things down
- The dictator understands business context (e.g., real estate, healthcare)
- The goal is speed, clarity, and action—not consensus

---

### 3. Stop at “Theoretical Saturation”

Don’t analyze every log. Stop when you stop discovering new failure types—typically after 40–100 traces.  
This concept, borrowed from qualitative research, ensures efficiency and focus.

---

### 4. Group Notes into Categories

Once you have open codes (raw notes), group them into higher-level themes called axial codes.  
These represent recurring failure modes.

Examples of Axial Codes:
- Human handoff issues  
- Hallucinated or false information  
- Formatting errors  
- Broken conversational flow  
- Incorrect tool use  

You can use an LLM (like Claude or GPT) to help cluster your open notes into categories.

---

### 5. Quantify Your Findings

Turn categories into counts. A simple pivot table or spreadsheet can show what’s breaking most often.

| Failure Mode          | Count |
|------------------------|-------|
| Conversational flow    | 17    |
| Human handoff          | 9     |
| Hallucinations         | 7     |

This provides a data-driven roadmap for prioritizing fixes.

---

### 6. Automate Evals for Key Failures

Once you know your failure modes, build automated tests to prevent regressions.

#### a. Code-Based Evals
- Deterministic tests (like unit tests)
- Example: checking if output JSON is valid or properly formatted

#### b. LLM-as-a-Judge Evals
- Used for subjective or contextual checks
- Another AI model evaluates if the main AI’s output was correct
- The judge gives a binary answer (true/false, pass/fail)

Binary decisions are clearer and more actionable than 1–5 or 1–7 rating scales.

---

### 7. Validate and Align with Humans

Before trusting your AI judge, compare its decisions with human evaluations on sample data.  
If it agrees consistently, you can safely scale it up to monitor thousands of traces in production.

This keeps evals aligned with real-world quality and prevents false confidence.

---

## Core Principles of Great Evals

| Principle | Description |
|------------|-------------|
| Start with data, not tests | Observe real examples before automating anything. |
| Product people must be involved | PMs understand what “good” looks like. |
| Keep it simple | Avoid committees; use lightweight tools like Sheets or Notion. |
| Binary beats scoring | Yes/No results are more reliable than subjective ratings. |
| Validate your evals | Check that AI judges agree with humans. |
| Iterate, don’t overbuild | One solid eval setup can last months with small updates. |

---

## The Big Picture

Evals turn AI product quality into measurable data by combining:

1. Human intuition – spotting and labeling issues  
2. Structured synthesis – grouping patterns into categories  
3. Automation – using code or LLMs to test performance continuously

Done right, evals become the product’s feedback engine: an analytics loop that ensures constant improvement.

---

## Summary

- Evals are the backbone of AI product reliability.  
- Start by manually exploring what’s broken.  
- Categorize and count issues to find your biggest problems.  
- Automate tests with code or AI judges.  
- Align automation with human understanding.  
- Use evals to continuously learn and iterate.
