# Introducing lipas: Why Agent Runtimes Need a "Claim Algebra"
*April 2026 · Alpha Release Notes*

## The Bug I Couldn't Reproduce
It happened like this:

An agent failed at step three in production. I fed it the exact same input and prompt, and it finished step two perfectly. I couldn't reproduce the error.

After staring at the logs for twenty minutes, I realized something: this wasn't a problem with my testing infrastructure or a lack of detailed logging. **The LLM agent paradigm itself is non-reproducible**—unless you treat every event as a recordable, replayable "fact" from day one.

Mainstream frameworks don't do this. They treat LLM calls and tool executions as "processes." Processes are fluid and carry side effects; when something goes wrong, you can only guess what happened by tracing them. "Facts," however, are typed, collapsible, and replayable.

Lipas is a runtime built on this distinction.

## Three Problems Most Frameworks Ignore
When building lipas, I focused on three specific pain points that most frameworks handle poorly:

**1. Tool side effects are never taken seriously.**
`add(a, b)` and `send_email(to, subject)` are fundamentally different. You can retry the former a thousand times without issue, but retrying the latter once is a disaster. Yet, most frameworks treat them the same: just functions with signatures and returns. 

Lipas makes side-effect types a **mandatory declaration** at registration: `PURE`, `READ_ONLY`, `IDEMPOTENT_WRITE`, or `EXTERNAL_WRITE`. Because the framework knows the type, it knows whether it’s safe to retry or replay a tool without the user writing a single line of orchestration code.

**2. Budget checks are always "post-mortem."**
You set a token limit. One LLM call costs half your budget, the next costs the other half, and then you're over. The system sends an alert, but the money is already spent. This model is broken.

Budgeting should be a **pre-flight interceptor**. Lipas checks "current spend + estimated cost" against your limit *before* a call is made. If it’s going to exceed the limit, the call is blocked, and a typed "Rejection Claim" is stored instead. You’ll never be surprised by a bill again.

**3. Deterministic replay is usually just "best effort."**
Most frameworks claim to support replay, but because LLMs use sampling (temperature) and tools touch the outside world, it rarely works perfectly. 

Lipas doesn't just "try." LLM calls are short-circuited via a `ReplayCursor`: they don't hit the network or consume tokens; they return exactly what was recorded. This is guaranteed by the underlying data structure—the **Claim Store** is a join-semilattice. In this model, duplicate claims are no-ops and the order of delivery doesn't change the final state. 

*(Note: In Alpha, tool replay is only 100% safe for PURE/READ_ONLY. External writes will currently re-execute. This will be fixed in Phase 4.)*

## How It Works Locally
**Lipas centers on one core abstraction: _the Claim_**.

Everything that happens—an LLM call, a tool result, resource consumption, or a guard rejection—is a typed Claim. All claims are folded into an append-only ClaimStore. 

Because of the mathematical properties of these claims (idempotency and commutativity), features like "auditability" and "replayability" aren't extra modules you have to build—they are **direct consequences** of the data structure itself.

## Alpha Version: Status Report

**What it can do:**
* Single-agent ReAct loops with full effect log auditing.
* Side-effect-aware tool harnesses with budget pre-flight gates.
* Deterministic LLM replay via `ReplayCursor`.
* Custom policy guards with structured "allow/deny" reasoning.

**What it can't do (yet):**
* **Streaming:** It currently waits for the full response.
* **Multi-agent orchestration:** This is planned for Phase 4.
* **Persistence:** Currently in-memory only; data is lost on restart.
* **OpenAI/Anthropic adapters:** Currently Ollama only (Phase 5).

## Seeking Feedback
If you’ve built agents before and have been frustrated by "ghost bugs" you can't reproduce or budget alerts that arrive too late, I want to hear from you.

I might have solved the wrong half of the problem—perhaps the real pain is in prompt management or tool automation. If you think this direction is off-track, tell me.

**Repo:** [lipas-otp/lipas](https://github.com/lipas-otp/lipas)
**Discussion:** Open an issue or a PR on the main repo.
