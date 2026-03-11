# Agentic-System-Design-with-memory-confidence-scores
More reliable agentic system with memory using LangChain memory modules, confidence scores to recommendations, synthetic cases where the system should fail, refined prompts to reduce false conclusions.



Extend the same agentic system.
Add memory using LangChain memory modules.
Add confidence scores to recommendations.
Create synthetic cases where the system should fail.
Refine prompts to reduce false conclusions.
Document when humans should override the system.
Example tools:
LangChain Memory
OpenAI API
Python
Example output:
A safer, more reliable agentic growth system.



Memory is added to the agentic system


Recommendations include confidence scores


At least 5 bad or misleading scenarios are tested


Prompts or logic are refined to reduce wrong advice


README clearly states when humans should override the system


If this is done, she understands safety and trust in AI systems.



The README for this project should say exactly this — that you started with explicit failure cases, recognised the limitation, and the next evolution is scope-based guardrails. That shows product thinking, not just coding.
The three layers real products use:
Layer 1 — Scope guardrails in system prompt
Defines what the agent CAN and CANNOT reason about. Anything outside = LOW confidence automatically. This handles the infinite unknown unknowns.
Layer 2 — Tool boundary as hard guardrail
The agent literally cannot answer what it has no tool for. If there's no geo breakdown tool, it cannot return geo data no matter what it's asked. The tool library is a hard constraint, not just a soft instruction.
Layer 3 — A small set of curated failure cases for testing
Not to handle every case — but to test that the guardrails actually work. Like unit tests. You pick 5-10 representative cases that stress-test the boundary and run them before every deployment.
Stage 2 — Learning Log
1. Memory Architecture
What we built: Custom summary memory using a list + GPT compression call — no LangChain needed.
How it works:

investigation_memory[] stores every past run as {goal, conclusion}
get_memory_context() sends all past runs to GPT and asks it to compress into 3-4 sentences
save_to_memory() appends to the list after every run
Memory is injected into the system prompt at the start of every new run

Key insight: Memory has two operations — READ at the start, WRITE at the end. Together they create continuity across sessions.

2. Memory Wasn't Being Used — First Attempt Failed
What happened: Memory was saved correctly but the agent still ran all 4 tools on the follow-up question. It ignored what it already knew.
Why: The investigation protocol in the system prompt said "always call funnel summary first" — this rigid instruction overrode the memory context.
Fix: Added an explicit rule at the top of the protocol:

"First check memory — if root cause is already confirmed, skip to the specific tool needed"

Result: Tool calls dropped from 4 to 1 on follow-up questions.
Key insight: Instructions in the system prompt compete with each other. A rigid protocol will override soft context like memory unless you explicitly tell the agent to prioritise memory first.

3. Confidence Scoring
What we built: Agent self-assigns HIGH / MEDIUM / LOW at the end of every answer.
How it works:

System prompt instructs the agent to always end with CONFIDENCE: HIGH/MEDIUM/LOW + REASON
parse_confidence() does simple string matching on the output — no intelligence, pure text scan
The score reflects the agent's own reasoning about data quality and tool coverage

Key insight: Confidence is not mathematically calculated — it's the LLM self-assessing in natural language. This means it can be wrong. In Stage 3 a separate agent will review the primary agent's conclusion instead of relying on self-scoring.

4. Human Override Gate
What we built: input() pause that fires only on LOW confidence.
How it works:

HIGH and MEDIUM pass through automatically
LOW prints a warning and waits for human to type yes/no
Investigation is saved to memory regardless of human decision

Key insight: The gate fires based on confidence level, not on what the agent found. A correct finding with LOW confidence still gets a human review. Safety is about uncertainty, not just correctness.

5. Guardrails vs Failure Cases
What we started with: 3 explicit failure cases (F1, F2, F3)
What we learned: This doesn't scale. Real systems can have thousands of out-of-scope questions.
Better approach: Define the agent's SCOPE explicitly in the system prompt. Everything outside the scope automatically gets LOW confidence. The failure cases become a test suite to verify guardrails work — not the guardrails themselves.
Key insight: Guardrails define the fence. Failure cases test the fence. You need both, but they serve different purposes.

6. Tool Boundary as Hard Guardrail
Key insight: The agent literally cannot answer what it has no tool for. No geo breakdown tool = no geo data, no matter how the question is phrased. The tool library is a hard constraint on top of the soft constraint of the system prompt guardrails.

Coming in Stage 3

Multi-agent system: orchestrator + diagnosis agent + action agent
Scope-based guardrails replace explicit failure cases
Automated test suite runs against guardrails before every deployment
Separate agent reviews primary agent's conclusion instead of self-scoring confidence

7. Failure Cases — Results

Guardrails worked perfectly for out-of-scope questions (F1, F2) — agent correctly refused to answer without calling any tools
F3 revealed a gap: agent scores confidence based on data clarity, not question clarity. A vague question with clear data gets HIGH instead of MEDIUM
Fix for Stage 3: separate reviewer agent that scores confidence based on both data quality AND question specificity

This is the honest limitation we discussed — the agent self-scores confidence based on data clarity, not question clarity. The data was clear so it said HIGH, even though the question was ambiguous.


WHEN HUMANS SHOULD OVERRIDE THE AGENTIC SYSTEM
================================================

ALWAYS OVERRIDE WHEN:
1. Confidence is LOW — agent does not have enough data or tool
   coverage to conclude
2. Anomaly detection finds NO clear date — could be gradual
   degradation or a data pipeline issue, not a real product bug
3. Recommended action would affect >10% of users or >$100K revenue
4. Diagnosis contradicts what engineering already knows
   (e.g. no deploys happened on the anomaly date)
5. Two back-to-back investigations give different root causes
   for the same symptom

CONSIDER OVERRIDING WHEN (MEDIUM confidence):
1. Only one segment breakdown supports the conclusion —
   the other shows mixed signals
2. Anomaly period is very short (1-2 days) — could be noise,
   not a real issue
3. Past memory shows a different root cause for similar symptoms

DO NOT OVERRIDE WHEN (HIGH confidence):
1. Anomaly confirmed, clear start date, one segment clearly
   broken, other segment healthy
2. Consistent findings across BOTH device AND traffic breakdowns
3. Past memory aligns with current findings

WHAT TO DO AFTER OVERRIDING:
1. Note the reason for override in your incident log
2. Run the specific tool manually to sanity-check the data
3. Check with engineering about recent deploys on the anomaly date
4. If the agent was wrong, debug in this order:
   - Read reasoning trace — did agent call right tools?
   - Run tools manually — did they return correct data?
   - Check data freshness — is the dataset stale?
   - Check tool descriptions — did agent skip a needed tool?
   - Check system prompt — did guardrails fire incorrectly?

Stage 2 — Complete Learning Log
1. Memory Architecture
What we built: Custom summary memory using a list + GPT compression call — no LangChain needed.
How it works:

investigation_memory[] stores every past run as {goal, conclusion}
get_memory_context() sends all past runs to GPT and asks it to compress into 3-4 sentences
save_to_memory() appends to the list after every run
Memory is injected into the system prompt at the start of every new run

Key insight: Memory has two operations — READ at the start, WRITE at the end. Together they create continuity across sessions.

2. Memory Wasn't Being Used — First Attempt Failed
What happened: Memory was saved correctly but agent still ran all 4 tools on the follow-up question.
Why: The investigation protocol said "always call funnel summary first" — this rigid instruction overrode the memory context.
Fix: Added one rule at the top of the protocol:

"First check memory — if root cause already confirmed, skip to the specific tool needed"

Result: Tool calls dropped from 4 to 1 on follow-up questions.
Key insight: Instructions in the system prompt compete with each other. A rigid protocol overrides soft context like memory unless you explicitly tell the agent to prioritise memory first.

3. Confidence Scoring
What we built: Agent self-assigns HIGH / MEDIUM / LOW at the end of every answer.
How it works:

System prompt instructs agent to always end with CONFIDENCE: HIGH/MEDIUM/LOW + REASON
parse_confidence() does simple string matching — no intelligence, pure text scan
Score reflects agent's own reasoning about data quality and tool coverage

Key insight: Confidence is not mathematically calculated — it's the LLM self-assessing in natural language. It can be wrong. In Stage 3 a separate reviewer agent will replace self-scoring.

4. Human Override Gate
What we built: input() pause that fires only on LOW confidence.
How it works:

HIGH and MEDIUM pass through automatically
LOW prints a warning and waits for human to type yes/no
Investigation saved to memory regardless of human decision

Key insight: The gate fires based on uncertainty, not correctness. A correct finding with LOW confidence still gets human review. Safety is about what the agent doesn't know, not just what it found.

5. Guardrails vs Failure Cases
What we started with: 3 explicit failure cases (F1, F2, F3)
What we learned: Doesn't scale. Real systems have thousands of possible out-of-scope questions.
Better approach: Define SCOPE in the system prompt. Everything outside = LOW confidence automatically. Failure cases become a test suite to verify guardrails work.
Key insight: Guardrails define the fence. Failure cases test the fence. You need both but they serve different purposes.

6. Tool Boundary as Hard Guardrail
Key insight: The agent literally cannot answer what it has no tool for. No geo tool = no geo data, no matter how the question is phrased. Tool library is a hard constraint on top of the soft constraint of the system prompt.

7. Failure Cases — What Actually Happened
CaseExpectedGotPass?F1 — Pricing/competitor questionLOW confidence, no tools calledLOW confidence, 0 tool calls, gate fired✅ PerfectF2 — Geographic questionLOW confidence, no geo hallucinationLOW confidence, 0 tool calls, gate fired✅ PerfectF3 — Vague "something seems off"MEDIUM confidenceHIGH confidence⚠️ Partial
F3 gap explained: Agent scores confidence based on data clarity, not question clarity. Data was unambiguous so it said HIGH — even though the question was vague. A separate reviewer agent in Stage 3 will catch this.

8. What This System Cannot Do Yet

Confidence self-scoring can be overconfident (F3 showed this)
Memory is session-only — restarting Colab wipes it
Single agent doing everything — no separation of diagnosis vs action
No automated test suite — failure cases run manually
No persistent logging — reasoning traces lost after session


Coming in Stage 3

Multi-agent: orchestrator + diagnosis agent + action agent
Separate reviewer agent replaces self-scored confidence
Scope-based guardrails with automated test suite
Persistent memory across sessions
Separation of diagnosis and action with independent approval

next stage:
Multi-agent system (orchestrator + specialist agents)
Replace failure cases with scope-based guardrails + automated test suite against guardrails
