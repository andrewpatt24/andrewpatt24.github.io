---
date: 2026-05-20
title: A Cursor skill that interviews you, then writes the blog post
description: >-
  andys-got-skills ships write-technical-blog-post: a ~5 min voice setup, a ~10 min
  blog interview, repo context, and a full draft with anti-AI-slop rules baked in.
tags:
  - ai
  - tools
image: /images/posts/cursor-blog-skill.jpg
---

I kept starting blog posts about side projects and stopping at the same place: enough context in my head, too little on the page, and a draft that sounded like every other AI article. Outlines help. So does [Rizèl Scarlett’s guide to technical blogging](https://dev.to/blackgirlbytes/the-ultimate-guide-to-writing-technical-blog-posts-5464). Neither tells Cursor *how I write* or forces the boring interview before the prose.

I packaged the workflow as a Cursor Agent Skill in [andys-got-skills](https://github.com/andrewpatt24/andys-got-skills). The first skill, **write-technical-blog-post**, runs a short voice setup when needed, a structured blog interview, then writes the full post from your answers and the repo README.

## What a Cursor skill is here

A skill is a folder with `SKILL.md` (YAML frontmatter + instructions) and supporting files. Cursor loads it when you ask for that workflow. You install from git:

```bash
git clone https://github.com/andrewpatt24/andys-got-skills.git
ln -sf "$(pwd)/andys-got-skills/skills/write-technical-blog-post" \
  ~/.cursor/skills/write-technical-blog-post
```

Then in chat:

> Use the write-technical-blog-post skill to write a post about [project]. Repo: https://github.com/…

The repo is a **skills library**: one folder per workflow. More skills land under `skills/<name>/` as I add them.

## Two interviews, one draft

| Phase | Time | Purpose |
|-------|------|---------|
| Voice setup | ~5 min | Fill `config-tone.txt` with *your* prose (or paste samples) |
| Blog interview | ~10 min | Topic, hook, beats, repo, tags, CTA |
| Agent work | after "go" | README/code scan, full Markdown draft, edit passes |

**Voice first.** If `config-tone.txt` is empty, the skill runs [voice-interview.md](https://github.com/andrewpatt24/andys-got-skills/blob/main/skills/write-technical-blog-post/voice-interview.md): you explain something technical in your own words, write a short paragraph about a project, pick opener style and words you avoid. The agent saves your answers verbatim. No inventing a fake “sample” voice.

**Blog second.** Three rounds: shape (topic, post type, motivation, repo), substance (hook, before/after, key beats, proof), delivery (title, length, publish target, tags, exclude list, CTA). You get a brief capture and outline; reply **go** to draft.

You can say **skip voice setup** if you are in a hurry. The skill notes that and relies on rules only.

## Where your voice lives

| File | Role |
|------|------|
| `config-tone.txt` | **Samples:** your cadence (not rules) |
| `best-practices.md` | Punctuation, bans, blog formatting |
| `human-style.md` | Anti-AI-slop, burstiness, fact checks |
| `reference.md` | Rizèl-style outlines (how-to, explainer, etc.) |
| `voice-interview.md` | Script when samples are missing |

Resolution order for voice samples:

1. `<project>/.cursor/config-tone.txt`
2. `~/.cursor/skills/write-technical-blog-post/config-tone.txt`

Project file wins when it has real text. On a blog repo, that keeps site voice separate from other work.

Rules include things I actually care about: no em dashes, no “In this post we will explore”, no contrastive “it’s not X, it’s Y” spine sentences, active voice, concrete hooks. [human-style.md](https://github.com/andrewpatt24/andys-got-skills/blob/main/skills/write-technical-blog-post/human-style.md) adds burstiness, banned phrases (delve, leverage, Moreover), and a mandatory edit pass after the first draft.

## What the agent does with your repo

During the blog interview it can pull README, commands, and architecture in parallel. After **go** it:

1. Picks an outline from `reference.md` (how-to, explainer, opinion, listicle)
2. Drafts in your voice samples + rules
3. Discovers post path and frontmatter from **sibling posts** in the workspace (Jekyll on my site; not hardcoded in the skill)
4. Runs human-style and best-practices scans before handoff

It should not treat old AI-rewritten posts as voice reference unless you paste them into `config-tone.txt` yourself.

## Repo layout

```text
andys-got-skills/
├── README.md
└── skills/
    └── write-technical-blog-post/
        ├── SKILL.md
        ├── best-practices.md
        ├── human-style.md
        ├── voice-interview.md
        ├── reference.md
        └── config-tone.txt
```

MIT license. Fork, symlink, adjust `best-practices.md` for your own bans.

## Try it

1. Clone and symlink as above (or copy the folder into `~/.cursor/skills/`).
2. Open a project with a README or GitHub URL.
3. Ask Cursor to use **write-technical-blog-post** for that project.
4. Complete voice setup once; replace `config-tone.txt` later with older writing you prefer.

I use this for posts like the golf coach and GoPro migration write-ups on this site. The skill does not replace editing; it replaces staring at a blank file and forgetting to ask yourself the obvious questions.

Issues and PRs: [github.com/andrewpatt24/andys-got-skills](https://github.com/andrewpatt24/andys-got-skills).

---

*Cursor skills evolve; install path and discovery may change with Cursor updates. Check the repo README.*
