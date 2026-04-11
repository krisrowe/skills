---
name: for-later
description: Use when the user mentions a concern, idea, proposal, problem, or other topic of exploration, consideration, inquiry, or action that they explicitly want to defer to later rather than discuss or act on in the moment. Triggered by language like "note this for later", "don't forget", "action item", "come back to this", "for later", "remember this", "capture this", "side note", "btw", "by the way", "as an aside", "just noting", "we'll revisit". Activating this skill is itself the decision to punt — the agent will NOT analyze, evaluate, solve, respond to, or otherwise engage with the topic at the moment of invocation. It will only record the topic, continue whatever work was in flight, and surface the topic later for disposition. Do NOT invoke if the user seems to want the topic discussed or acted on now — use only when deferral is the clear intent.
---

# For Later — Capture and Continue

## Purpose

Some concerns arrive in the middle of active work. The user flags
something that needs to be addressed later without breaking the
current task flow. This skill captures the concern completely,
keeps the agent focused on what it was already doing, and ensures
the concern resurfaces at session end so nothing is lost.

## Core behavioral rules

These rules are absolute. Violating them defeats the skill's
purpose.

### Rule 1 — Do not treat the capture as an interruption

The user flagging an action item is **not** asking the agent to
stop what it was doing. It is not a pause, a redirect, or a
priority change. Continue whatever work was in progress. Finish
the current reasoning, the current tool call sequence, the current
response, and any logical follow-ups the original task required.

### Rule 2 — Do not treat the capture as correction or feedback

The action item is **not** a criticism of the current work. It is
**not** an implicit instruction to undo or revise what the agent
just did. It is **not** a hint that the current tool call was
wrong. Do not reinterpret current work in light of the new concern.
Do not second-guess the in-flight approach. Treat the capture as
fully orthogonal to the active task unless the user explicitly
says otherwise.

The user may phrase the capture using words that sound like
feedback ("you know, this makes me realize we have a problem with
X"). Resist the reflex to read this as "stop and fix X." The
user's framing is *"here is something we need to come back to,
but keep doing what you were doing right now."*

### Rule 3 — Capture in full, no summarization

Record the concern in the user's own words and preserve the
rationale, context, examples, and any connected issues they
mention. Do not compress to a bullet point. The capture must
contain enough detail that a future session (or the same session
after context rot) can act on it without needing to reconstruct
reasoning from a terse label.

Specifically, capture:
- What the concern is, in the user's framing
- Why it matters (the rationale they gave, or implied)
- What part of the project / system / conversation it touches
- Any specific places they pointed at (files, docs, issues, code
  symbols, test names, skill names, etc.)
- Any implied or explicit next steps — "we should file this",
  "we should rewrite this doc", "this needs a skill update"
- The time and loose context under which it was raised (what was
  the agent doing when the concern arrived?), so the concern can
  be connected back to its origin during later review

### Rule 4 — Do not spend thinking time on the captured concern at invocation

Capturing is a **lightweight, cheap** operation. At the moment of
capture, the agent records the concern and moves on. It does NOT:

- Analyze the concern's merits, risks, or implications.
- Evaluate whether it's a real problem, a nothingburger, or
  something in between.
- Compose a tailored response, proposed solution, or reasoned
  opinion about it.
- Break chain of thought on the active task to reason about the
  concern.
- Spend tokens on any kind of "thoughtful engagement" with the
  captured content at invocation.

The whole point of capturing is to **defer all of that thinking**
until the revisit-and-discuss step later in the session. Any
analytical effort spent at capture time is effort stolen from the
active task the user is already expecting the agent to finish.

This is the critical distinction from answer-and-forget aside
patterns elsewhere in the agent ecosystem (see the README for
this skill for specifics). Those patterns non-destructively
acknowledge the aside but still burn real thinking time
processing and responding to it in parallel with the main turn.
This skill does not do that. Capture is essentially a file-and-
forget at the moment of invocation, deliberately cheap, so the
agent's chain of thought on the in-flight task is not slowed
down or polluted.

Engaging analytically with the concern is what the later revisit
is for — and at that point, engagement with the user is
explicitly in scope.

### Rule 5 — Confirm the capture visibly, then continue

The user must be able to tell that the capture happened. A
captured action item with zero visible confirmation is
indistinguishable to the user from the agent ignoring the
request. Never silently continue.

**If the capture goes to the session's task-tracking tool and
that tool already surfaces new items in the user's UX**
(e.g., a task list that renders in the status area), that
visible surfacing IS the confirmation. The agent does not
need to add a separate acknowledgement sentence. Trust the
platform's UX to communicate the capture.

**If the capture goes anywhere else** — in-context mental
list, TODOs file, personal notes, memory — the agent MUST
emit a brief acknowledgement at the tail of its response.
Keep it short, unspecific, and fast to produce. Examples:

- "Noted — will revisit later in the session."
- "Noted [topic] to revisit later in the session."
- "Noted for revisit after other work."

Upper bound: roughly one sentence naming the topic and when
(later in the session, after other work). Don't restate the
user's concern in full. Don't propose anything. Don't
compose a thoughtful summary. Lean toward speed — the goal
is a micro-latency confirmation so the user knows the
request registered, not a considered response. Everything
analytical is the revisit's job.

After the acknowledgement (or after silent capture when the
UX already surfaces it), return to what was in flight and
finish it. Do not summarize the captured concern back to the
user as the next action. Do not ask whether to address the
captured concern now unless the user's language explicitly
left it ambiguous.

The default **immediate** response is: record + bare
acknowledgement + keep going. The default **eventual**
response — before session end — is the revisit-and-discuss
that reaches a real disposition (see "Final disposition"
below). Activation of this skill sets up the second step
without blocking on it.

## Where to capture — the storage spectrum

Storage options for captured action items span a spectrum from
"the lightest possible in-session trace" to "fully durable
cross-session artifact." The right choice depends on the
mechanisms already in use this session, the scope of the
concern, and how much work the user is authorizing now versus
later.

**Activation of this skill does not by itself establish a
durable record.** It establishes a cue to keep the topic
visible until disposition. How durable the cue becomes is a
separate decision made case by case.

Prefer the first option that fits without introducing new
tooling the session isn't already using. Escalate to a more
durable option only if the concern's scope or the user's
language justifies it.

### Option A — Session-scoped task tracking tool (if already in use)

If the current agent platform provides a task-tracking tool
(Claude Code's task list, Gemini CLI's equivalent, a user-
maintained working agenda, or any similar mechanism) **and the
session is already using it**, add the captured action item to
that list as a deferred / not-started entry. Mark it clearly so
it is visible as "noted, not being worked on now."

This is the preferred option when available because:

- The tool surfaces the item naturally throughout the session
- End-of-session wrap-up mechanisms already read from it
- The agent doesn't have to invent a new capture location
- The user can see the full pending set alongside active work

Important nuances:

- Noting such an action item, at the time of invoking this
  skill for it, **is** a commitment — not to make changes
  within the project or do other tangible work in this
  session, but to revisit the item with the user, discuss it
  together, and reach a disposition before the session ends. The disposition may be: execute on a
  plan, design, or solution that comes out of that discussion;
  rule out the need for any further discussion or any work;
  or promote the idea to a durable artifact (issue,
  persistent TODO, notes repo) for a future session. The
  revisit happens after currently in-flight work and after
  any action items captured earlier in the session.
- If the tool has a priority or status field, mark the item
  in a way that signals "noted, not currently being worked
  on" without implying it will never be looked at again. It
  is on the list specifically so the end-of-session sweep
  can pick it up.
- The task tool's memory is session-scoped, not durable
  across sessions. The session-end disposition step is the
  only thing that decides whether the concern should be
  promoted to a durable artifact or dropped.

If the session is **not** already using a task-tracking tool,
do not start one just to capture a single action item. That's
ceremony, not value. Fall back to Option B.

### Option B — In-context capture (the "mental list")

The lightest option: record the captured concern as a short,
clearly labeled note at the tail of the current response
("captured as action item: ...") and rely on the fact that the
prompt where the skill was invoked will remain in session
context. This is sometimes all that's needed for a drive-by
concern raised mid-task.

Caveats:

- Context compaction or session truncation may eventually drop
  the original prompt. If the concern is important enough to
  matter after compaction, escalate to Option A or beyond.
- The agent must still remember to surface the captured item
  at end-of-session review. "It's in the context" is only
  durable until the context goes away.

This option is best when the concern is small, the current
task is the dominant focus, and the user is likely to wrap up
the session within the same context window.

### Option C — A session-scoped TODOs file

If the session has already created or been directed to a
`TODOS.md` / `temp/TODOS.md` / personal-notes file for this
project's work, append the captured action item there with
enough context that it remains actionable without the
session's conversation history. Prefer gitignored or private
locations for raw capture — polish can happen later when the
item is promoted to a durable artifact.

Use this option when multiple captured items are accumulating
and Option A isn't available, or when the concerns are likely
to survive past the current session.

### Option D — Direct promotion to a GitHub issue

Sometimes the user's capture request is effectively "file an
issue for this." If the concern is well-formed, has a clear
home repo, and the user's language invites immediate issue
creation ("we need a ticket for this"), invoke the
`author-github-issue` skill (or equivalent issue-authoring
process) **after finishing the current active task**. Do NOT
file the issue before completing the in-flight work unless
the user explicitly asks.

### Option E — A permanent personal notes location

If the user has an established personal notes repo, knowledge
base, or similar durable location for ongoing thoughts, capture
there. This is best for concerns that cross sessions and
projects — architectural reflections, meta-observations, things
that aren't yet scoped well enough for a code change or issue.

### Option F — Agent memory

If the agent has a persistent memory system (file-based or
otherwise) for things that should survive across sessions, and
the captured concern is long-lived (meta-concerns, recurring
reminders, reference material), write it to memory. This is the
most durable option but also the hardest to rediscover — use it
for things that need to be pulled in by future sessions, not
for one-off follow-ups.

## Phrasing captured items written to structured storage

If the capture stays purely in-context — just leaving the
original user prompt in the session transcript as a "mental
list" entry (Option B) — phrasing doesn't matter. The user's
own words are the entry.

But when the capture is written to any **structured** location
(the session's task-tracking tool, a TODOs file, personal
notes, agent memory — Options A, C, D, E, F), the phrasing
of the stored entry matters. The default phrasing must frame
the item as a **revisit** of the topic, not a prescribed
action on it.

Examples:

- Good: "Revisit: the naming question the user raised"
- Good: "Revisit: the concern about the current approach"
- Good: "Revisit: the idea of splitting this into two"
- Bad: "Rename the thing"
- Bad: "Replace the current approach with approach B"
- Bad: "Split this into two"

The "bad" phrasings silently pre-commit the user (and any
future session that reads the entry) to a specific action.
But the whole reason the skill was invoked is that **no
action has been decided yet**. The revisit-and-discuss step
is where action gets decided. Writing an action at capture
time defeats the deferral and puts words in the user's mouth.

**Exception:** if the user's own language explicitly stated
a specific action in the same breath as the capture — e.g.,
"for later: archive the old items", "note for later: we need
to rename the thing to the other name" — then the action is
already decided and the entry CAN be phrased as that action
("Archive the old items", "Rename the thing"). The user has
done the analysis and committed; the skill is just deferring
execution, not deferring the decision. Phrase the entry as
an action **only** when the user's own language was an
action, not a concern, observation, idea, or problem.

When in doubt, use "revisit X" framing. Rewriting an entry
at revisit time is cheap; taking back a commitment the user
never actually made is not.

## When to surface captured items

A captured action item must become visible again before the
session ends, so the user can decide its disposition. The
agent must resurface all pending captured items in any of the
following moments:

- **Any point at which the agent is tempted to conclude that
  the session's intended scope is complete.** Before signaling
  "we're done" or "nothing left to do" or handing back to the
  user with an empty plate, the agent must check whether any
  captured items remain outstanding. Captured items belong in
  that check before any "done" verdict. An agent that says
  "we're done" while captured items are still pending has
  silently discarded them.

- **Any time a list of open items, remaining scope, pending
  work, or session inventory is rendered** — whether on the
  user's prompt ("what's left," "what's pending," "what have
  we done and what's outstanding") or on the agent's own
  initiative. Captured items must appear in that list.

- **Any "what have we done / what's outstanding / what's not
  captured / what's pending / what's next" discussion at all**
  — the agent should proactively raise captured items as part
  of that discussion, not wait to be asked for them by name.

- **When a session-wrap-up skill is explicitly invoked** —
  e.g., `capture-context`, `sessions`, or any equivalent
  end-of-session review process. Captured items must appear in
  the audit. Never let a captured action item disappear into
  the conversation's past without a disposition.

- **Before a risky or destructive action** — if the agent is
  about to commit, push, close an issue, or do anything that
  marks a chunk of work as done, check whether any captured
  action item should be handled first or bundled into that
  action.

### Ordering when resurfacing

When surfacing captured items alongside other open items or
session work:

1. **Concrete, deliberate, in-flight work items come first.**
   Things the agent and user were explicitly working on,
   blocked tasks, half-done edits, real TODOs from the
   session's active work — these go at the top of the list.
   They represent scope the user has already committed to and
   the agent has already started.

2. **Captured action items come after.** These are
   deferrals the user explicitly punted on. By construction
   they have lower priority than in-flight committed work.
   They are not ignored, but they are not elevated above real
   active work either.

3. **Within the captured items list itself, preserve the
   original capture order.** List captured items in the
   sequence they were initially noted during the session.
   Do not reorder by perceived importance, size, topic, or
   estimated effort. The user's own order of noting is the
   authoritative order for the captured list — the user gets
   to re-order later if they choose.

## Final disposition

Every captured action item must be revisited with the user and
resolved into one of these outcomes before the session is
considered complete. The revisit is a discussion, not a
unilateral agent decision — the user is part of the disposition
step.

- **Executed on a plan, design, or solution coming out of the
  revisit discussion** — after discussing the item, the user and
  agent agreed on concrete work that should happen, and the work
  was carried out this session. The captured record is
  superseded by the completed work.
- **Ruled out after discussion** — the user and agent discussed
  the item and concluded no further discussion or any work is
  needed. The concern is dropped with the user's explicit
  acknowledgement, not silently.
- **Promoted to a durable artifact for a future session** —
  filed as a GitHub issue (via the `author-github-issue` skill if
  available), added to a persistent TODO list, or written to
  permanent memory. Use this when the concern deserves attention
  but doesn't fit into the current session's scope or timeline.

Do NOT leave captured items in an ambiguous state at session
end. "We'll figure it out next time" without a revisit and a
chosen disposition is indistinguishable from lost.

## What this skill is NOT for

- **Immediate execution.** If the user is asking the agent to do
  something right now and not later, it's a regular task, not a
  deferred capture. Use whatever mechanism the session normally
  uses for in-flight work.
- **Clarifying questions or corrections about current work.** If
  the user is correcting or redirecting the current approach,
  treat their input as feedback on the active task, not as a
  deferred capture. Action-item capture is for concerns that are
  explicitly *separate* from current work.
- **End-of-session summaries.** That's the job of wrap-up skills
  like `capture-context`. This skill feeds that process by
  ensuring the captured items are available at wrap-up; it does
  not replace it.
- **Reminders that are trivially addressed in-place.** If a
  concern takes 30 seconds to handle and is strictly in-scope
  for the current work, just handle it. Don't ceremoniously
  capture it.

## Example invocations

These are all valid triggers for this skill:

- "Note for later: we should revisit how X works."
- "While you're doing that, remember that Y is broken — don't
  fix it now, just don't forget."
- "Capture this: we've never defined what Z means in the README.
  Come back to it."
- "Drive-by thought — we should file an issue for this at some
  point, but keep going with what you were doing."
- "Pending concern: the docs don't cover A. Don't stop working
  on B, just track it."
- "Side note: our skill for P should probably be updated to
  handle Q. Not now."
- "I'm just going to leave this here: we need to think about R
  eventually."

In each case, the agent's correct response is:

1. Capture the concern in full detail per the rules above
2. Briefly acknowledge the capture (one short sentence, tail of
   response)
3. Return to and complete the work that was in progress
4. Ensure the capture survives to end-of-session review
