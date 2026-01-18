---
layout: post
title: "The Protocol Wars: MCP, A2A, and Why Humans Are Still the Missing Piece"
date: 2026-01-16
author: Srirajasekhar "Bobby" Koritala
categories: [ai, protocols, mcp]
excerpt: "As AI protocols battle for dominance, we're missing the most important integration point of all—humans."
---

Something interesting is happening in AI right now. The biggest players are racing to define how AI agents talk to each other—and to us.

Anthropic has MCP. Google has A2A. OpenAI has their Agents SDK. Everyone's building protocols.

But after nine years of building AI systems—including several patented ones—I keep noticing what's missing from these conversations: humans.

## The Protocol Landscape

Let me break down what's actually happening.

<div class="mermaid">
graph TB
    subgraph "Current Protocol Focus"
        MCP[MCP<br/>Context Sharing]
        A2A[A2A<br/>Agent Coordination]
        SDK[OpenAI SDK<br/>Agent Framework]
    end

    MCP --> AI1[AI Agent]
    A2A --> AI2[AI Agent]
    SDK --> AI3[AI Agent]

    AI1 <--> AI2
    AI2 <--> AI3
    AI1 <--> AI3

    Human[👤 Human]
    Human -.->|"?"| AI1
    Human -.->|"?"| AI2
    Human -.->|"?"| AI3

    style Human fill:#ffcccc,stroke:#cc0000
    style MCP fill:#e6f3ff,stroke:#0066cc
    style A2A fill:#e6ffe6,stroke:#00cc00
    style SDK fill:#fff0e6,stroke:#cc6600
</div>

### MCP: Anthropic's Model Context Protocol

MCP (Model Context Protocol) is Anthropic's attempt to standardize how AI models share context. Announced in November 2024, it's designed as an open protocol that creates a universal way for AI assistants to connect with data sources and tools[^1].

The problem MCP solves is real: when you chain AI calls together, context gets lost. The second model doesn't know what the first model was thinking. MCP creates a structured way to pass that context along.

**What MCP does well:**
- Standardized context format across models
- Clean handoffs between AI components
- Works across different AI providers (not just Claude)
- Open specification that anyone can implement

**What MCP doesn't address:**
- What happens when a human needs to intervene?
- How does human context get preserved and passed?
- Who decides when AI should stop and ask for help?

### A2A: Google's Agent-to-Agent Protocol

Google's A2A, announced in April 2025, takes a different angle. Instead of focusing on context, it focuses on how autonomous agents communicate and coordinate with each other[^2].

Built on existing standards like HTTP and JSON-RPC, A2A defines how agents discover each other's capabilities, negotiate tasks, and collaborate on complex workflows. It's designed for a world where multiple AI agents from different vendors need to work together.

**What A2A does well:**
- Multi-agent coordination across vendors
- Capability discovery and negotiation
- Task delegation between specialized agents
- Built on proven web standards

**What A2A doesn't address:**
- Same gap: where do humans fit?
- When Agent A hands off to Agent B, what if a human should have been Agent B?
- How do you audit decisions that were never meant to be audited?

### OpenAI's Agents SDK

OpenAI released their Agents SDK in March 2025, providing a production-ready framework for building multi-agent systems[^3]. It replaced the experimental Swarm framework with something more robust.

**What it does well:**
- Clean developer experience
- Good defaults for common patterns
- Tight integration with OpenAI's models
- Production-ready tooling

**What it doesn't address:**
- Vendor lock-in (it's OpenAI-first)
- The human question, again

## The Pattern I Keep Seeing

Every major protocol focuses on AI-to-AI communication. That makes sense—it's a hard technical problem, and the companies building these protocols are AI companies.

But here's what I've learned from building AI systems in healthcare, finance, and logistics: **the hardest part isn't AI talking to AI. It's AI talking to humans, and knowing when it should.**

Research from Stanford's Human-Centered AI Institute consistently shows that human-AI collaboration outperforms either alone. Their 2024 study on AI-assisted decision making found that humans with AI support made 23% better decisions than AI alone—but only when the handoff between human and AI was well-designed[^4].

### The Handoff Problem

Consider a common workflow: an AI agent analyzes customer data, generates a recommendation, and takes action.

<div class="mermaid">
graph LR
    subgraph "Current: AI-Only Pipeline"
        D1[📊 Data Agent] --> A1[🔍 Analysis Agent]
        A1 --> R1[💡 Recommendation Agent]
        R1 --> E1[⚡ Action Agent]
    end

    style D1 fill:#e6f3ff
    style A1 fill:#e6f3ff
    style R1 fill:#e6f3ff
    style E1 fill:#e6f3ff
</div>

With current protocols, the AI-to-AI parts work great:
1. Data agent extracts information ✓
2. Analysis agent processes it ✓
3. Recommendation agent generates options ✓
4. Action agent executes ✓

But what if step 3 should have been "Human reviews options before action"?

<div class="mermaid">
graph LR
    subgraph "Needed: Human-Aware Pipeline"
        D2[📊 Data Agent] --> A2[🔍 Analysis Agent]
        A2 --> R2[💡 Recommendation Agent]
        R2 --> H2[👤 Human Review]
        H2 --> E2[⚡ Action Agent]
    end

    style D2 fill:#e6f3ff
    style A2 fill:#e6f3ff
    style R2 fill:#e6f3ff
    style H2 fill:#ffe6e6,stroke:#cc0000,stroke-width:2px
    style E2 fill:#e6f3ff
</div>

Current protocols don't have a clean answer for this. You're left bolting on custom solutions:
- A Slack notification that someone might miss
- An email that sits in an inbox
- A dashboard nobody checks

The context that made the AI's recommendation make sense? Often lost by the time a human sees it.

A 2024 McKinsey study on AI in enterprise workflows found that 67% of failed AI implementations cited "poor human-AI handoff design" as a primary factor[^5].

### The Confidence Problem

Here's another gap: AI doesn't know what it doesn't know.

When an AI agent is uncertain, it should probably ask a human. But current protocols don't standardize:
- How to express uncertainty
- When uncertainty should trigger human involvement
- How to preserve context for the human handoff

Research from MIT's Computer Science and Artificial Intelligence Laboratory (CSAIL) found that LLMs are often confidently wrong—expressing high certainty on incorrect answers 31% of the time[^6]. Without confidence-based routing to humans, these errors propagate through automated workflows.

### The Audit Problem

Regulations are catching up to AI. GDPR, HIPAA, SOC2, the EU AI Act—all require some form of explainability and audit trail.

The EU AI Act, which took full effect in 2025, specifically requires "meaningful human oversight" for high-risk AI systems[^7]. Article 14 mandates that humans must be able to understand AI outputs and intervene when necessary.

Current protocols focus on what happened between agents. But auditors ask different questions:
- Who made this decision?
- Was a human involved?
- Could a human have intervened?
- Why wasn't a human involved?

If your protocol doesn't have humans as first-class citizens, you're going to struggle with these questions.

## What Would Human-Aware Protocols Look Like?

I've been thinking about this for years. Here's what I believe is needed:

### 1. Humans as First-Class Participants

A human shouldn't be a "fallback" or an "escalation path." They should be a valid node type, just like an AI agent.

```
Workflow:
  [AI: Analyze] → [Human: Validate] → [AI: Execute]
```

The protocol should handle routing to humans the same way it handles routing to AI—with preserved context, clear expectations, and tracked outcomes.

### 2. Context Preservation Across the Human Boundary

When AI hands off to a human, the human should understand:
- What the AI was trying to do
- Why it stopped
- What options it considered
- What it recommends

And when the human hands back to AI, the AI should understand:
- What the human decided
- Why they decided it
- Any additional context they provided

MCP is great for AI-to-AI context. We need the same rigor for AI-to-human and human-to-AI.

### 3. Confidence-Based Routing

Protocols should support routing decisions based on confidence:

<div class="mermaid">
graph TD
    AI[🤖 AI Decision] --> Conf{Confidence?}

    Conf -->|"> 90%"| Auto[✅ Auto-Execute]
    Conf -->|"70-90%"| Notify[📧 Notify Human<br/>Proceed if no objection]
    Conf -->|"< 70%"| Require[⏸️ Require Approval]

    Auto --> Log[📋 Log for Audit]
    Notify --> Log
    Require --> Human[👤 Human Reviews]
    Human --> Decide{Decision}
    Decide -->|Approve| Execute[⚡ Execute]
    Decide -->|Reject| Stop[🛑 Stop]
    Execute --> Log
    Stop --> Log

    style AI fill:#e6f3ff
    style Human fill:#ffe6e6
    style Auto fill:#e6ffe6
    style Notify fill:#fff0e6
    style Require fill:#ffe6e6
</div>

This isn't just a nice-to-have. For regulated industries, it's becoming mandatory.

### 4. Native Audit Trails

Every decision point should be auditable:
- What information was available
- What decision was made (by AI or human)
- Why that decision was made
- What happened next

This needs to be built into the protocol, not bolted on after.

## The Opportunity

The companies building MCP, A2A, and other protocols are solving real problems. I'm not criticizing their work—I'm building on it.

But there's a gap in the market for human-aware AI orchestration. Something that:
- Speaks MCP, A2A, and other protocols
- Treats humans as first-class workflow participants
- Preserves context across the human boundary
- Provides native governance and audit capabilities

The AI orchestration market is projected to reach $42.8 billion by 2032, growing at 23.4% CAGR[^8]. Most of that will go to enterprise use cases. And enterprises can't deploy AI workflows that don't have humans in the loop—their compliance teams won't allow it.

## What I'm Working On

I've spent nine years building AI systems that work with humans, not around them. Several of those systems are now patented.

The common thread across all of them: **the most powerful AI systems don't replace humans. They collaborate with us.**

I'm now working on bringing this approach to AI orchestration more broadly. If you're interested in human-aware AI workflows, stay tuned—I'll have more to share soon.

In the meantime, I'm curious: what are the biggest gaps you see in current AI protocols? Where do humans fit in your AI workflows? I'd love to hear your perspective.

---

*Srirajasekhar "Bobby" Koritala is the founder of Bodaty. He has been building production AI systems for nearly a decade and holds multiple patents in AI and human-AI collaboration systems.*

---

## References

[^1]: Anthropic. (2024). "Introducing the Model Context Protocol." [anthropic.com/news/model-context-protocol](https://www.anthropic.com/news/model-context-protocol)

[^2]: Google Cloud. (2025). "Agent2Agent: An Open Protocol for AI Agent Interoperability." [cloud.google.com/blog/products/ai-machine-learning/a2a-protocol](https://cloud.google.com/blog/products/ai-machine-learning/a2a-protocol)

[^3]: OpenAI. (2025). "Introducing the Agents SDK." [openai.com/blog/agents-sdk](https://openai.com/blog/agents-sdk)

[^4]: Stanford HAI. (2024). "Human-AI Collaboration in High-Stakes Decision Making." [hai.stanford.edu/research/human-ai-collaboration](https://hai.stanford.edu/research)

[^5]: McKinsey & Company. (2024). "The State of AI in 2024: Generative AI's Breakout Year." [mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)

[^6]: MIT CSAIL. (2024). "Calibrating Large Language Model Confidence." [csail.mit.edu/research/llm-calibration](https://www.csail.mit.edu/research)

[^7]: European Commission. (2024). "The EU Artificial Intelligence Act." [digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai)

[^8]: Grand View Research. (2024). "AI Orchestration Market Size Report, 2024-2032." [grandviewresearch.com/industry-analysis/ai-orchestration-market](https://www.grandviewresearch.com/industry-analysis/ai-orchestration-market)
