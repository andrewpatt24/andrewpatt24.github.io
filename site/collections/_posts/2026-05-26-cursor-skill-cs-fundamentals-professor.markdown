---
date: 2026-05-26
title: A Cursor skill that teaches CS fundamentals from your actual code
description: >-
  andys-got-skills ships cs-fundamentals-professor: first-principles explanations,
  trade-offs, ASCII data flow, code citations from your repo, and a pop quiz, without
  rewriting your app.
tags:
  - ai
  - tools
  - learning
image: /images/posts/cs-fundamentals-professor.jpg
---

Cursor can add Redis, JWT middleware, or a vector store in one session. My repo compiles. My mental model often does not. I wanted explanations tied to **my** files, with the rigor of a systems course, not another generated patch.

The second skill in [andys-got-skills](https://github.com/andrewpatt24/andys-got-skills) is **cs-fundamentals-professor**. It reads your code first, then teaches in a fixed five-part shape: concept, diagram, trade-offs, quoted lines from your tree, and one question you answer yourself.

## The problem it targets

Fast LLM-assisted building creates a specific gap: **velocity outruns understanding**. You accept a rate limiter, an auth layer, or a Prisma schema because the agent said it works. When something breaks at scale, you are debugging patterns you never internalized.

This skill does not replace your editor. It does not refactor unless you ask. It explains **why** a pattern likely exists in **your** setup, names the underlying CS problem (networking, memory, concurrency, crypto, algorithms), and stops with a Socratic check.

## What a response looks like

Every answer follows the same section order (headings included):

| Section | Job |
|---------|-----|
| **1. The Fundamental Concept** | One paragraph: the CS problem, without circular jargon |
| **2. Mental Model / Data Flow** | ASCII diagram or indented map (client → LB → store, etc.) |
| **3. Trade-Off Analysis** | Pros/cons, what breaks at ~100× scale, complexity where it matters |
| **4. Code Context** | 2–3 quoted lines from **your** codebase with file paths |
| **5. Professor's Pop Quiz** | One trade-off question; **no answer** until you reply |

The repo ships [example-response.md](https://github.com/andrewpatt24/andys-got-skills/blob/main/skills/cs-fundamentals-professor/example-response.md) for Redis rate limiting: distributed counting, atomic `INCR`/`EXPIRE`, fail-open vs fail-closed, hot-key bottlenecks. Use it as the canonical shape, not as copy-paste text for your app.

Tone is SICP-flavored: precise, calm, encouraging through clarity. No "Great question!" filler. If the agent has not opened your files, it should say so instead of inventing line numbers.

## Four pillars (what the agent optimizes for)

1. **First principles:** strip framework magic; name the real systems problem.
2. **Why, not just what:** latency vs consistency, fail-open vs fail-closed, operational cost at scale.
3. **Mental model:** something you can picture on a whiteboard or in a terminal.
4. **Socratic check:** one punchy question at the end; you supply the reasoning.

For AI-heavy stacks, the skill still grounds in systems CS: embeddings → geometry and ANN indexing; tool calling → RPC and trust boundaries; prompt caching → memoization and invalidation; streaming → backpressure and partial I/O. Model behavior is separated from systems design unless you ask for ML depth.

## When it runs (and when it should not)

**Good triggers:** "Why did Cursor add Redis here?", "Explain our auth middleware", "I don't get this Prisma schema", "Activate CS fundamentals", "MIT CS skill", or any explain/why/ELI5 on **your** architecture.

**Out of scope:** bugfixes from stack traces, refactors, tests, vendor/model shopping, greenfield framework comparisons, writing blog posts or other skills, Cursor install help. Those jobs belong elsewhere (including [write-technical-blog-post](https://github.com/andrewpatt24/andys-got-skills/tree/main/skills/write-technical-blog-post) for posts like this one).

Before teaching, the agent is instructed to search your workspace for the symbol you care about (`redis`, `jwt`, `rateLimit`, etc.) when you did not give a path.

## Install

Same pattern as the blog skill; symlink into `~/.cursor/skills/`:

```bash
git clone https://github.com/andrewpatt24/andys-got-skills.git
ln -sf "$(pwd)/andys-got-skills/skills/cs-fundamentals-professor" \
  ~/.cursor/skills/cs-fundamentals-professor
```

Restart Cursor or open a new agent chat. Then in a project with real code:

> Use cs-fundamentals-professor: Cursor added Redis for rate limiting in `middleware.ts`. Why?

Or:

> Activate CS fundamentals: explain our auth middleware

Depth knobs: **go deeper**, **ELI5**, or ask for only trade-offs (all five sections stay; shorter where you do not need length).

## Repo layout

```text
andys-got-skills/
└── skills/
    └── cs-fundamentals-professor/
        ├── SKILL.md
        ├── example-response.md
        └── evals/          # trigger/description optimization (optional)
```

MIT license. Fork, symlink, edit `SKILL.md` if you want a different section order or quiz style.

## Pair it with the blog skill

I use **write-technical-blog-post** to ship write-ups and **cs-fundamentals-professor** to close the understanding gap on the code those posts describe. One produces public prose from interviews and repo facts; the other keeps your head aligned with the repo while you are still building.

## Try it

1. Symlink the skill as above.
2. Open a repo where an agent recently added something you accepted but do not fully own.
3. Point at a file or symbol; ask *why* it is there.
4. Answer the pop quiz in chat before asking for the solution.

Issues and PRs: [github.com/andrewpatt24/andys-got-skills](https://github.com/andrewpatt24/andys-got-skills).

---

*Cursor skills evolve; install paths may change with Cursor updates. Check the repo README.*
