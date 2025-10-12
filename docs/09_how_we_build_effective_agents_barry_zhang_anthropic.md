# How We Build Effective Agents — Barry Zhang (Anthropic)

*Transcript from the AI Engineer Summit, “How We Build Effective Agents” by Barry Zhang, Anthropic.*  
*Original video: [https://www.youtube.com/watch?v=D7_ipDqhtwk](https://www.youtube.com/watch?v=D7_ipDqhtwk)*

---

## Introduction

> “Wow, it's incredible to be on the same stage as so many people I’ve learned so much from.”

Barry Zhang opens the talk by introducing himself and outlining the three main ideas from the blog post *Building Effective Agents*:

1. **Don’t build agents for everything**  
2. **Keep it simple**  
3. **Think like your agents**

---

## How We Got Here

The evolution of agent systems:

- **Early Days:**  
  Developers started with simple tasks — summarization, classification, extraction — which once felt magical but are now table stakes.

- **Workflows:**  
  As sophistication grew, multiple model calls were orchestrated in predefined control flows, balancing cost and latency for performance.  
  → This marked the beginning of **agentic systems**.

- **Agents:**  
  Agents can now decide their own trajectory and operate based on environment feedback, increasing capability **but also** cost, latency, and error consequences.

---

## 1. Don’t Build Agents for Everything

> “Agents are a way to scale complex and valuable tasks — not a drop-in upgrade for every use case.”

### When Should You Build an Agent?

**Checklist:**

1. **Task Complexity**  
   - Agents thrive in *ambiguous* problem spaces.  
   - If the decision tree can be easily mapped, build a simple workflow instead — cheaper and more controllable.

2. **Task Value**  
   - Exploration costs tokens.  
   - Low-value, high-volume tasks (e.g. customer support) should use workflows for common scenarios.  
   - High-value tasks justify higher compute.

3. **Critical Capabilities**  
   - Identify potential bottlenecks in the agent’s trajectory (e.g., code generation, debugging, recovery).  
   - If bottlenecks exist, **reduce scope** and retry.

4. **Error Cost & Discovery**  
   - High-stakes, hard-to-detect errors reduce trust and autonomy.  
   - Mitigate via *read-only access* or *human-in-the-loop*, though this limits scalability.

### Example: Coding Agents

Coding agents are a great fit because:

- The task is **complex** and **ambiguous**.  
- **High value** (good code has measurable business impact).  
- **Strong existing performance** from LLMs.  
- **Verifiable output** through tests and CI.

---

## 2. Keep It Simple

> “We’ve learned the hard way to keep this simple — complexity kills iteration speed.”

### Core Agent Components

Agents = **Models using tools in a loop**, defined by three key parts:

1. **Environment** — the system the agent operates in.  
2. **Tools** — the interfaces for action and feedback.  
3. **System Prompt** — defines goals, constraints, and desired behavior.

The model runs repeatedly within this loop — that’s an **agent**.

### Design Principles

- **Start simple.**  
  Iterating on these three components gives the highest ROI.
- **Optimize later.**  
  Once behaviors stabilize, tune for cost, latency, and user trust.

### Examples

Three agent use cases (internal or customer-facing) look vastly different in scope and capability — but **share almost identical backbone code**.

> “The only two design decisions that matter are:  
> (1) What tools does the agent have?  
> (2) What prompt defines its goals?”

### Optimization Tips

- Cache trajectories to reduce cost.  
- Parallelize tool calls for latency gains.  
- Expose progress transparently to build user trust.

---

## 3. Think Like Your Agent

> “Put yourself in the agent’s context window.”

LLMs act on limited context (10–20k tokens). To debug or improve behavior, **see the world as your agent does**.

### Exercise: The “Computer Use Agent”

Imagine you’re an agent operating a computer:
- You see a single screenshot and a vague description.
- You execute a click *blindly* and wait several seconds before seeing the next screenshot.
- You might have succeeded — or shut down the computer.  
→ This illustrates how **context limits perception**.

### What Agents Need

- **Screen resolution awareness**  
- **Recommended actions and limits**  
- **Guardrails** to prevent unnecessary exploration  

Do this exercise for your own agent — identify what context or metadata it truly needs.

### Use the Model as a Mirror

Because our systems “speak our language,” you can:

- Ask the model if your system prompt is clear or ambiguous.  
- Check if it understands tool descriptions or parameter needs.  
- Feed the agent’s trajectory back into the model and ask *why* it made certain decisions.

> “This doesn’t replace human reasoning — but it helps bridge our understanding and the agent’s.”

---

## Open Questions and Future Directions

Barry’s personal reflections on where agents are heading:

1. **Budget Awareness**  
   - Agents currently lack strong cost and latency control.  
   - How can we define and enforce budgets (tokens, time, dollars)?

2. **Self-Evolving Tools**  
   - Agents could eventually **design and improve their own tools**, adapting ergonomics dynamically.

3. **Multi-Agent Collaboration**  
   - Anticipates more **multi-agent systems** by end of year:  
     - Parallelized, modular, with separation of concerns.  
   - Open question:  
     - *How will agents communicate asynchronously?*  
     - *How do they recognize and coordinate with each other?*

---

## Key Takeaways

1. **Don’t build agents for everything.**  
   Focus on complex, high-value tasks where ambiguity is high.

2. **Keep it simple.**  
   Start with environment, tools, and prompt. Optimize later.

3. **Think like your agent.**  
   Debug from its limited perspective and provide the context it needs.

---

## Closing Remarks

> “Back in 2023, I was building AI products at Meta. I changed my title to *AI Engineer* — I loved the focus on practicality and usefulness.”

Barry ends with encouragement to stay grounded in building real, useful systems:

> “Let’s keep building.”

---

**Speaker:** Barry Zhang, Anthropic  
**Event:** AI Engineer Summit  
**Topic:** Building Effective Agents  
**Date:** 2024  
**Source:** [YouTube Transcript](https://www.youtube.com/watch?v=D7_ipDqhtwk)
