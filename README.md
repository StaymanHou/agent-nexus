# AgentNexus - A Claude Code Multi-Agent Product Workspace

A lightweight, document-only multi-agent system for product work. Agents coordinate via a shared folder, minimal context, and explicit handoffs. You invoke each agent by @handle; a **built-in executor agent** (Claude Code’s default) can then carry out the actual implementation tasks that your `@le` plan specifies.

Inspired by:
- https://github.com/buildermethods/agent-os
- https://github.com/ramakay/claude-self-reflect

---

## Quick Install

1. **Copy the agents**

   * You already have `agents/*.md` (e.g., `bookkeeper.md`, `product-analyst.md`, …).
   * Drop them into either:

     * **Project scope:** `./.claude/agents/`
     * **Personal scope:** `~/.claude/agents/`
   * They’re ready to use—no editing required.

2. **Create the workspace skeleton**

```
/product/
  index.md
  /context/        background.md, personas.md, glossary.md
  /analysis/       pid.md, decisions/
  /architecture/   overview.md, components.md, staging-plan.md
  /planning/       mvp.md, mvp-plan.md, roadmap.md, risks.md
  /ux/             user-stories.md, features/
  /eng/            impl-guides.md
  /memory/         recents.md, index.md, archive/
```

> Fixed root is `./product/`. Everything must be reachable from `/product/index.md`.

---

## Agents & Handles

* `@bk` Bookkeeper — memory & structure hygiene; session open/close; link integrity
* `@pa` Product Analyst — motives/needs → PID
* `@arch` Architect — tech stack options & system decomposition
* `@pm` Product Manager — **defines the MVP** + roadmap & ADRs
* `@pjm` Project Manager — orders MVP into an executable plan
* `@ux` UX Designer — user stories & acceptance criteria
* `@le` Lead Engineer — per-feature implementation plan & TODOs

> All agents act **only** when explicitly @mentioned and write **markdown only**.

**Built-in executor (default)**
Claude Code’s default/built-in agent (no custom file) can perform the **actual implementation work** once `@le` produces the plan (e.g., coding, tests, edits). Treat it as the “doer” that follows the plan; your subagents are planners, not executors.

---

## Design Principles

* **One folder, one brain** — `./product/` is the shared source of truth.
* **Strict context discipline** — each agent reads `index.md` → its contract files → only linked docs (no repo-wide scans).
* **Hierarchy + hygiene**

  * Split any file **> ~300 lines or ~10KB** into `overview.md` + children
  * Consolidate folders **> 20 files** or with “tiny-file sprawl”
  * Memory rotates **by size**, archives named `RECENT-YYYY-MM-DD[-a].md`
* **Traceability** — every write appends a **Change Receipt** (date, agent, summary, links).
* **Numeric feature IDs** — `F-001`, `F-002`, …; ADRs `D-YYYY-MM-DD-a.md`.

---

## Typical End-to-End Workflow

1. **Kickoff / context surfacing**

```
@bk open for "Payments MVP" kickoff
```

* Outputs 5–10 **links** to relevant docs (no inline content), notes any missing ToCs.

2. **Product analysis**

```
@pa draft an initial PID for Payments MVP (card-on-file, refunds, receipts).
```

* Writes `/product/analysis/pid.md`.

3. **Architecture**

```
@arch propose two stack options per PID constraints; add to /product/architecture.
```

* Writes `overview.md`, `components.md`, `staging-plan.md` with pros/cons & rollback paths.

4. **Define MVP**

```
@pm define the MVP features from PID + architecture; defer non-essentials.
```

* Writes `/product/planning/mvp.md` (WHAT + rationale), ADRs if needed.

5. **Plan the MVP**

```
@pjm convert MVP into an ordered plan with dependencies and risk notes.
```

* Writes `/product/planning/mvp-plan.md` (Feature ID | Depends On | Effort | Risk | Order | Status).

6. **User stories**

```
@ux write stories for F-001 refunds and F-002 receipts with acceptance criteria.
```

* Writes `/product/ux/features/F-001-*/story.md`, updates `user-stories.md`.

7. **Implementation plan**

```
@le produce an implementation plan for F-001 refunds; prioritize MVP speed.
```

* Writes `/product/ux/features/F-001-*/impl-plan.md`; sets Status in `mvp-plan.md` (e.g., **Ready for Dev**).

8. **Execute the plan (built-in executor)**

```
Follow @le’s impl-plan.md tasks.
```

* Use Claude Code’s **default built-in agent** to actually implement code/tests/docs following the plan.
* If new questions emerge, loop back to the relevant planner agent via @mention.

9. **Close & archive**

```
@bk close the session
```

* Adds ≤200-word recap to `/product/memory/recents.md`, rotates if over size, fixes ToCs, updates `/product/index.md#Next Actions`.

---

## Usage Notes (Claude Code)

* **Where to type:** Use the Claude Code chat or inline comments. Start requests with an @handle.
* **Scope:** Agents do **not** call each other. They only leave `Next Actions` with @mentions in `/product/index.md`.
* **Edits:** All edits are markdown files within `/product/` (no shell, no code execution by planners).
* **Executor handoff:** When `@le` marks a feature **Ready for Dev**, switch to the default built-in agent and implement the tasks from `impl-plan.md`.

---

## Common Commands

```text
@bk init            # validate links & indices in /product
@pa draft PID ...   # create/update pid.md from idea/brief
@arch options ...   # propose 2+ stacks with tradeoffs & rollback paths
@pm MVP ...         # define minimal viable feature set (WHAT + rationale)
@pjm plan ...       # order MVP features with dependencies & risks
@ux stories F-001   # create/clarify user stories & acceptance criteria
@le impl F-001      # write implementation plan & TODOs; set Status
@bk close           # recap, rotate memory by size, update Next Actions
```

---

## Templates

**Change Receipt**

```
## Change Receipt
- Date: YYYY-MM-DD
- Agent: @<handle>
- Summary: <one or two sentences>
- Related: [path1](../..), [path2](../..)
```

**Memory Recents Entry**

```
## YYYY-MM-DD — Session Summary
**Decisions:** D-YYYY-MM-DD-a (if any)  
**Deltas:** files touched + 1-liners  
**Open Questions:** bullets  
**Next Actions:** bullets with @handles and target files
```

---

## Troubleshooting

* **Orphan files or missing ToCs** → `@bk fix link integrity for /product/...`
* **Ambiguous scope/criteria** → `@pm` (scope) or `@ux` (acceptance) will push back or mark **Needs Clarification**
* **Overengineering risk** → `@pm` challenges; `@arch` documents refactor path
* **Dependency cycles** → `@pjm` resolves; `@le` flags if discovered late

---

**That’s it.** Keep `/product/` tidy, invoke one planner at a time, then hand off to the **built-in executor** to do the actual work guided by `impl-plan.md`.
