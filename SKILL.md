---
name: llm-council
description: Run any question, idea, or decision through a council of 5 AI advisors who independently analyze it, anonymously peer-review and rank each other, and synthesize a final verdict under a strict word cap. Based on Karpathy's LLM Council methodology, with peer-review scoring and brevity discipline from Stanford's decision-council prompt. MANDATORY TRIGGERS: 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. STRONG TRIGGERS (use when combined with a real decision or tradeoff): 'should I X or Y', 'which option', 'what would you do', 'is this the right move', 'validate this', 'get multiple perspectives', 'I can't decide', 'I'm torn between'. PROACTIVE TRIGGER: even with no phrase at all, if the user is visibly mid-decision with real stakes and multiple options (weighing tradeoffs out loud, listing pros/cons, going back and forth, asking "what would you do") and hasn't invoked the council yet, offer to run it — do not auto-run, ask first. Do NOT trigger on simple yes/no questions, factual lookups, or casual 'should I' without a meaningful tradeoff (e.g. 'should I use markdown' is not a council question). DO trigger when the user presents a genuine decision with stakes, multiple options, and context that suggests they want it pressure-tested from multiple angles.
---

# LLM Council

You ask one AI a question, you get one answer. That answer might be great. It might be mid. You have no way to tell because you only saw one perspective.

The council fixes this. It runs your question through 5 independent advisors, each thinking from a fundamentally different angle. Then they review and rank each other's work. Then a chairman synthesizes everything into a final recommendation that tells you where the advisors agree, where they clash, and what you should actually do.

This is adapted from Andrej Karpathy's LLM Council. He dispatches queries to multiple models, has them peer-review each other anonymously, then a chairman produces the final answer. We do the same thing inside Claude using sub-agents with different thinking lenses instead of different models. The anonymous peer-review scoring and the strict word cap on the final verdict are adapted from a Stanford decision-council prompt.

---

## quick start: name your stuck decision

The council works best when the question is specific. If the user hasn't given you enough to work with, ask them to fill this in (or fill it in yourself from the conversation before framing):

```
DECISIÓN EN LA QUE ESTOY ATRAPADO:
[La decisión o pregunta específica. Entre más específicos sean la situación,
las restricciones, y cómo se ve "bien", mejor será el consejo.]
```

This is not a separate step — it feeds directly into step 1 (frame the question) below.

---

## proactive detection (no trigger phrase needed)

Don't wait for "council this." Watch for these signs *anywhere* in the conversation — not just when context is weak, but any time the user is mid-decision and hasn't called the skill:

- They're weighing two or more real options out loud (pros/cons, "on one hand... on the other")
- They ask "what would you do?", "am I overthinking this?", "does this make sense?" about a choice with real stakes
- They describe a decision and then keep talking themselves in circles about it across multiple messages
- Real cost of being wrong: money, time, reputation, a relationship, a hire, a pivot — not a trivial or reversible choice

When you spot this, **do not auto-run the council.** Running it burns 15 sub-agent calls and takes real time — that's not something to spend on a guess. Instead, name what you noticed and offer it in one line, e.g.: "Esto suena a una decisión real con varias opciones en juego — ¿quieres que la pase por el consejo de 5 asesores?" Then wait for confirmation before starting step 1.

If they say yes but the context is still thin, use the quick-start template above to get what step 1 needs — this is the same weak-context handling as before, just reached through a different door.

Don't repeat the offer if they already declined it once for the same decision in this conversation.

---

## when to run the council

The council is for questions where being wrong is expensive.

Good council questions:

- "Should I launch a $97 workshop or a $497 course?"
- "Which of these 3 positioning angles is strongest?"
- "I'm thinking of pivoting from X to Y. Am I crazy?"
- "Here's my landing page copy. What's weak?"
- "Should I hire a VA or build an automation first?"

Bad council questions:

- "What's the capital of France?" (one right answer, no need for perspectives)
- "Write me a tweet" (creation task, not a decision)
- "Summarize this article" (processing task, not judgment)

The council shines when there's genuine uncertainty and the cost of a bad call is high. If you already know the answer and just want validation, the council will likely tell you things you don't want to hear. That's the point.

---

## the five advisors

Each advisor thinks from a different angle. They're not job titles or personas. They're thinking styles that naturally create tension with each other.

### 1. The Contrarian
Actively looks for what's wrong, what's missing, what will fail. Assumes the idea has a fatal flaw and tries to find it. If everything looks solid, digs deeper. Not a pessimist — the friend who saves you from a bad deal by asking the questions you're avoiding. Lists every reason the decision is wrong, what breaks first, and the worst plausible outcome.

### 2. The First Principles Thinker
Ignores the surface-level question and asks "what are we actually trying to solve here?" Strips away assumptions, destroys them if needed, and rebuilds the problem from the ground up as if no obvious framework were available. Sometimes the most valuable output is saying "you're asking the wrong question entirely."

### 3. The Expansionist
Looks for the upside everyone else is missing. What could be bigger? What adjacent opportunity is hiding? What's being undervalued? Doesn't care about risk (that's the Contrarian's job) — cares about the asymmetric outcome if this works even better than expected, and what the larger version of it opens up.

### 4. The Outsider
Has zero context about you, your field, your industry, or your history. Responds purely to what's in front of them and asks the "dumb" questions only an outsider asks. Catches the curse of knowledge: things people inside stopped questioning that are obvious to you but confusing to everyone else.

### 5. The Executor
Doesn't care about strategy. Cares about Monday morning. Only asks: can this actually be done, and what's the fastest path? Tells you exactly what to do this week — the email to send, the conversation to have, the file to create, or the decision to defer. If an idea sounds brilliant but has no clear first step, the Executor says so.

**Why these five:** They create three natural tensions. Contrarian vs Expansionist (downside vs upside). First Principles vs Executor (rethink everything vs just do it). The Outsider sits in the middle keeping everyone honest by seeing what fresh eyes see.

---

## how a council session works

### step 1: frame the question (with context enrichment)

When the user says "council this" (or any trigger phrase), do two things before framing:

**A. Scan the workspace for context.** The user's question is often just the tip of the iceberg. Before framing, quickly scan for and read any relevant context files:

- `CLAUDE.md` or `claude.md` in the project root or workspace (business context, preferences, constraints)
- Any `memory/` folder (audience profiles, voice docs, business details, past decisions)
- Any files the user explicitly referenced or attached
- Recent council transcripts in this folder (to avoid re-counciling the same ground)
- Any other context files relevant to the specific question (e.g. pricing questions → look for revenue data, past launch results, audience research)

Use `Glob` and quick `Read` calls to find these. Don't spend more than 30 seconds on this. You're looking for the 2-3 files that would give advisors what they need to give specific, grounded advice instead of generic takes.

**B. Frame the question.** Take the user's raw question (using the "quick start" template above if they haven't specified their constraints and what "good" looks like) AND the enriched context, and reframe it as a clear, neutral prompt all five advisors will receive. The framed question should include:

1. The core decision or question
2. Key context from the user's message (situation, constraints, what "good" looks like)
3. Key context from workspace files (business stage, audience, constraints, past results, relevant numbers)
4. What's at stake (why this decision matters)

Don't add your own opinion. Don't steer it. If the question is too vague ("council this: my business"), ask one clarifying question. Just one. Then proceed.

Save the framed question for the transcript.

### step 2: convene the council (5 sub-agents in parallel)

Spawn all 5 advisors simultaneously as sub-agents. Each gets:

1. Their advisor identity and thinking style (from the descriptions above)
2. The framed question
3. A clear instruction: respond independently. Do not hedge. Do not try to be balanced. Lean fully into your assigned perspective. If you see a fatal flaw, say it. If you see massive upside, say it. Stay in character — different vocabulary, different priorities, different blind spots per advisor.

Each advisor should produce a response of 150-300 words.

**Sub-agent prompt template:**

```
You are [Advisor Name] on an LLM Council.

Your thinking style: [advisor description from above]

A user has brought this question to the council:

---
[framed question]
---

Respond from your perspective. Be direct and specific. Don't hedge or try to be balanced.
Lean fully into your assigned angle. The other advisors will cover the angles you're not covering.

Keep your response between 150-300 words. No preamble. Go straight into your analysis.
```

### step 3: anonymous peer review + ranking (5 sub-agents in parallel)

This is the step that makes the council more than "ask 5 times."

Collect all 5 advisor responses. Anonymize them as Response A through E (randomize the mapping so there's no positional bias — never reveal to a reviewer which advisor is which).

Spawn 5 new sub-agents, one per advisor. Each reviewer sees all 5 anonymized responses and must produce both a ranking and an assessment:

1. **Rank the other four responses 1-4 on accuracy and insight** (the reviewer never ranks itself).
2. For each of those four, one paragraph explaining what it got right and what it missed.
3. Which response has the biggest blind spot, and what is it?
4. What did ALL five responses miss that the council should consider?

**Reviewer prompt template:**

```
You are reviewing the outputs of an LLM Council. Five advisors independently answered this question:

---
[framed question]
---

Here are their anonymized responses:

**Response A:** [response]
**Response B:** [response]
**Response C:** [response]
**Response D:** [response]
**Response E:** [response]

Rank the OTHER FOUR responses (not your own) 1-4 on accuracy and insight, and for each write
one paragraph on what it got right and what it missed. Then answer:

- Which response has the biggest blind spot? What is it missing?
- What did ALL five responses miss that the council should consider?

Be specific. Reference responses by letter. Keep the whole review under 250 words. Be direct.
```

### step 4: chairman synthesis

One agent gets everything: the original question, all 5 advisor responses (de-anonymized), and all 5 rankings + reviews.

The chairman's job is the final council output, following this structure:

**COUNCIL VERDICT**

1. **Where the council agrees** — points multiple advisors converged on independently. High-confidence signals.
2. **Where the council clashes** — genuine disagreements. Don't smooth these over. Present both sides and why reasonable advisors disagree.
3. **Blind spots the council caught** — things that only emerged through peer review and ranking.
4. **The recommendation** — a clear, actionable call. Not "it depends." The chairman can side with a minority advisor if the reasoning and rankings support it.
5. **The one thing you should do first** — a single concrete next step, not a list.

Sections 1-3 can run as long as they need to be genuinely useful. Section 4 (recommendation) and 5 (next step) together must stay **under 250 words** — state the correct call, the strongest reason for it, the biggest risk to watch, and the specific next step for the next 7 days. No hedging, no "both sides" in this part. Sharper is better.

**Chairman prompt template:**

```
You are the Chairman of an LLM Council. Synthesize the work of 5 advisors and their
anonymous peer rankings/reviews into a final verdict.

The question brought to the council:
---
[framed question]
---

ADVISOR RESPONSES:
**The Contrarian:** [response]
**The First Principles Thinker:** [response]
**The Expansionist:** [response]
**The Outsider:** [response]
**The Executor:** [response]

PEER RANKINGS AND REVIEWS:
[all 5 peer reviews, including their 1-4 rankings]

Produce the council verdict using this exact structure:

## Where the Council Agrees
[High-confidence convergent points.]

## Where the Council Clashes
[Genuine disagreements, both sides, why reasonable advisors disagree.]

## Blind Spots the Council Caught
[What peer review/ranking surfaced that individual advisors missed.]

## The Recommendation
[The real answer. No hedging.]

## The One Thing to Do First
[One concrete next step for the next 7 days.]

The Recommendation + The One Thing to Do First sections combined must be under 250 words.
State: the correct call, the strongest reason, the biggest risk, and the specific 7-day next step.
Be direct. Don't hedge.
```

### step 5: present the verdict in chat

After the chairman synthesis is complete, present the full verdict directly in chat using markdown. Do NOT generate an HTML report or any files unless the user asks for one.

```
## Council Verdict: {short topic}

### Where the Council Agrees
{content}

### Where the Council Clashes
{content}

### Blind Spots the Council Caught
{content}

### The Recommendation
{content}

### The One Thing to Do First
{content}
```

Keep it scannable. Use bullet points.

### step 6: save the transcript (optional)

Only save a transcript if the user asks for it or the question is significant enough to reference later. If saving, write to `council-transcript-[timestamp].md` in the project's `active/` directory.

---

## important notes

- **Always spawn all 5 advisors in parallel.** Sequential spawning wastes time and lets earlier responses bleed into later ones.
- **Always anonymize for peer review and ranking.** If reviewers know which advisor said what, they'll defer to certain thinking styles instead of evaluating on merit.
- **The chairman can disagree with the majority.** If 4 out of 5 advisors say "do it" but the 1 dissenter ranks highest on accuracy/insight, the chairman should side with the dissenter and explain why.
- **The Recommendation + Next Step is capped at 250 words.** This is deliberate — the analysis sections can be long, but the actual call has to be sharp enough to act on immediately.
- **Don't council trivial questions.** If the user asks something with one right answer, just answer it.
