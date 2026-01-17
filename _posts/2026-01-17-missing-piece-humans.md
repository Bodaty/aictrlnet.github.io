---
layout: post
title: "The Missing Piece in AI Orchestration: Humans"
date: 2026-01-17
author: Bobby Bodaty
categories: [ai, hitl, orchestration]
excerpt: "Every AI orchestration platform focuses on connecting agents. Almost none focus on connecting humans."
---

Last week I wrote about the protocol wars—MCP, A2A, and how the biggest AI companies are racing to define how agents communicate. The response was overwhelming, and one theme kept coming up:

*"Okay, I see the gap. But what does human-in-the-loop actually look like in practice?"*

Let me show you.

## The Demo That Always Fails

I've sat through dozens of AI demos over the years. They all follow the same script:

1. "Watch this AI agent analyze your data..."
2. "Now it's generating recommendations..."
3. "And here it executes the action automatically!"
4. *Applause*

Then someone asks: "What if the recommendation is wrong?"

The presenter pauses. "Well, you could... add an approval step?"

And that's where the demo ends, because what comes next isn't pretty. In real implementations, "add an approval step" means:

- Building a custom notification system
- Creating a UI for reviewing AI decisions
- Figuring out how to preserve context so the reviewer understands what they're approving
- Handling timeouts, escalations, and edge cases
- Maintaining audit trails for compliance

That "simple approval step" is often more work than the entire AI pipeline.

According to Gartner's 2024 AI adoption survey, organizations spend an average of 40% of their AI project budget on "integration and operationalization"—which includes human oversight mechanisms[^1].

## Why Human-in-the-Loop Is So Hard

After building AI systems for nine years, I've identified three reasons why HITL (human-in-the-loop) is consistently underestimated:

### 1. Context Collapse

When an AI hands off to a human, context collapses.

The AI "knows" why it made a decision—it has the full chain of reasoning, the data it analyzed, the alternatives it considered. But surfacing that to a human in a useful way? That's a different problem entirely.

Most systems show humans the output ("Recommend: Approve loan") without the reasoning ("Based on credit score 720, debt-to-income 0.3, similar approved applications, confidence 87%").

The human is asked to approve something they don't fully understand. So they either:
- Rubber-stamp everything (defeating the purpose)
- Reject everything out of caution (defeating the purpose)
- Waste time investigating each decision manually (defeating the efficiency gains)

A 2023 study from Carnegie Mellon's Human-Computer Interaction Institute found that users who received AI recommendations *without* explanations agreed with the AI 89% of the time—but only 61% of their agreements were actually correct[^2]. They were rubber-stamping bad decisions.

**The fix:** Context must be a first-class citizen in the handoff. Not an afterthought. Not a log file. Structured, relevant, actionable context.

### 2. Workflow Impedance Mismatch

AI operates in milliseconds. Humans operate in minutes, hours, or days.

When you insert a human into an AI workflow, you create an impedance mismatch. The workflow was designed for speed; now it has to wait. And waiting creates problems:

- What if the data changes while waiting for approval?
- What if the human never responds?
- What if the human is on vacation?
- What if the decision is time-sensitive?

Research from MIT Sloan shows that the average time-to-decision for human approval in enterprise AI workflows is 4.2 hours—but 23% of requests take longer than 24 hours[^3]. Most AI orchestration tools can't handle this gracefully.

**The fix:** Human steps need different primitives. Timeouts. Escalation paths. Delegation. Async execution with callbacks. The orchestration layer must understand that humans are fundamentally different from APIs.

### 3. The Audit Paradox

Here's a paradox I've encountered repeatedly:

The more you automate with AI, the more you need to prove humans were involved.

Regulators, compliance teams, and auditors don't trust "the AI decided." They want to know:
- Who was accountable for this decision?
- Could a human have intervened?
- Why didn't they?
- If they did intervene, what did they decide and why?

The Federal Reserve's 2023 guidance on AI in financial services explicitly states: "Financial institutions must be able to demonstrate appropriate human oversight of AI-driven decisions"[^4]. Similar requirements exist in healthcare (FDA AI/ML guidance), insurance (state regulations), and now broadly under the EU AI Act.

**The fix:** Audit trails must capture human involvement (or deliberate non-involvement) at every decision point. This needs to be built into the orchestration layer, not bolted on.

## What Real Human-in-the-Loop Looks Like

Let me describe what I think proper HITL should look like. This isn't theoretical—it's based on systems I've built that are running in production.

### Humans as Nodes, Not Exceptions

In a well-designed system, a human is just another node type:

```
[AI: Extract Data] → [AI: Analyze] → [Human: Validate] → [AI: Execute]
```

The human node has the same interface as an AI node:
- It receives structured input
- It produces structured output
- It can succeed, fail, or timeout
- Its decision is logged and auditable

The orchestration layer doesn't care whether a node is AI or human. It just routes work to the right place and handles the response.

### Context That Travels

When work arrives at a human, they should see:

```
TASK: Approve customer refund
AMOUNT: $450
AI RECOMMENDATION: Approve
CONFIDENCE: 78%

WHY THIS RECOMMENDATION:
- Customer has been with us 3 years
- First refund request
- Product was defective (confirmed by support ticket #4521)
- Similar cases: 94% approved

WHY YOU'RE SEEING THIS:
- Amount exceeds auto-approve threshold ($200)
- Confidence below auto-approve threshold (85%)

OPTIONS:
[Approve] [Reject] [Request More Info] [Escalate]
```

The human has everything they need to make a decision. They're not rubber-stamping; they're making an informed choice with AI assistance.

Research from Stanford HAI shows that presenting AI recommendations with structured explanations increases human decision accuracy by 31% compared to recommendations alone[^5].

### Confidence-Based Routing

Not every decision needs human review. The system should route intelligently:

| Confidence | Routing |
|------------|---------|
| 95%+ | Auto-execute, log for audit |
| 80-95% | Auto-execute, notify human, allow override window |
| 60-80% | Require human approval |
| <60% | Require senior human approval |

The thresholds are configurable by workflow, by risk level, by regulatory requirement. The point is: humans are involved proportionally to uncertainty and risk.

This approach is supported by research from the Harvard Business School, which found that "tiered autonomy" models—where AI handles routine decisions and humans handle edge cases—increased both throughput and accuracy compared to either full automation or full manual review[^6].

### Escalation That Works

When a human doesn't respond, the system shouldn't just... wait forever.

```
Initial assignment: Agent (SLA: 4 hours)
        ↓ timeout
Escalate to: Team Lead (SLA: 2 hours)
        ↓ timeout
Escalate to: Manager (SLA: 1 hour)
        ↓ timeout
Fallback: Auto-reject with notification to all parties
```

Every escalation is logged. At any point, auditors can see exactly what happened and why.

### Audit by Design

Every decision point captures:

```json
{
  "timestamp": "2026-01-17T10:23:45Z",
  "workflow_id": "refund-4521",
  "node": "human-validation",
  "assigned_to": "agent@company.com",
  "decision": "approved",
  "reasoning": "Customer's explanation matches support ticket",
  "time_to_decision": "3m 42s",
  "context_viewed": true,
  "ai_recommendation": "approve",
  "ai_confidence": 0.78,
  "override": false
}
```

Six months later, when someone asks "why did we approve this refund?", you have a complete answer.

## The Gap in Current Tools

I've evaluated most of the AI orchestration tools on the market. They fall into two categories:

**Code-first frameworks** (LangChain, CrewAI, AutoGen): Powerful, but humans are DIY. You can build HITL, but you're building it from scratch every time.

LangChain's 2024 survey of enterprise users found that 72% had built custom human-in-the-loop functionality, and 58% cited it as "the most time-consuming part of deployment"[^7].

**Visual automation tools** (n8n, Zapier, Make): Easy to use, but humans are an afterthought. You can add a "wait for webhook" step, but that's not the same as proper HITL.

Neither category treats humans as first-class workflow participants. Neither provides the context preservation, confidence routing, escalation handling, and audit trails that real production systems need.

## What I've Been Building

For the past several years, I've been working on this problem. Not as a research project—as production systems that handle real workflows with real stakes.

The patterns I described above? They're not hypothetical. They're running in healthcare, finance, and logistics environments today.

I've taken those patterns and built them into something more general. A framework where humans and AI are equal participants in workflows. Where context travels across the human boundary. Where governance and audit trails are built in, not bolted on.

I'm almost ready to share it publicly.

If you're building AI systems that need human involvement—and in my experience, that's most AI systems worth building—I think you'll find it useful.

More soon.

---

*Have you struggled with human-in-the-loop in your AI systems? What patterns have you found that work? I'd love to hear—reply or reach out directly.*

---

*Bobby Bodaty has been building production AI systems for nearly a decade. He holds multiple patents in AI and human-AI collaboration systems.*

---

## References

[^1]: Gartner. (2024). "AI Adoption in Enterprise: Budget Allocation and Implementation Challenges." [gartner.com/en/documents/ai-adoption-2024](https://www.gartner.com/en/information-technology/insights/artificial-intelligence)

[^2]: Carnegie Mellon HCII. (2023). "Understanding Human Reliance on AI Recommendations." [hcii.cmu.edu/research/ai-decision-support](https://www.hcii.cmu.edu/research)

[^3]: MIT Sloan Management Review. (2024). "The Hidden Costs of Human-AI Handoffs." [sloanreview.mit.edu/article/human-ai-handoffs](https://sloanreview.mit.edu/)

[^4]: Federal Reserve. (2023). "Supervisory Guidance on Artificial Intelligence." [federalreserve.gov/supervisionreg/ai-guidance](https://www.federalreserve.gov/supervisionreg.htm)

[^5]: Stanford HAI. (2024). "Explainable AI and Human Decision Quality." [hai.stanford.edu/research/explainable-ai](https://hai.stanford.edu/research)

[^6]: Harvard Business School. (2024). "Tiered Autonomy in AI-Assisted Decision Making." [hbs.edu/research/ai-tiered-autonomy](https://www.hbs.edu/research/)

[^7]: LangChain. (2024). "State of LLM Applications: Enterprise Survey Results." [langchain.com/state-of-llm-apps-2024](https://www.langchain.com/)
