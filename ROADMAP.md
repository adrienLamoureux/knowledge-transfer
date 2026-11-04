# Roadmap

> Planned 2026-11-04, nothing started yet. Tick the boxes as work lands.
> This file has no frontmatter on purpose, so `build_index.py` leaves it out of `INDEX.md`.

## Finish and publish

**Goal:** the 13 docs move from `draft` to `final` and can be read at a public URL.

- [ ] Review each doc in the reading order of `INDEX.md`. Check the facts. Every claim must carry
      one label: Practice, Notes, Lesson or Case. No employer or client name. Simple English.
      Set `status: final` as each one passes.
- [ ] CI: add `.github/workflows/check.yml`, which checks out `claude-skills` to get the docs-kit
      scripts and then runs `check_docs.py` on every doc listed in `INDEX.md`. Do not run it on
      `README.md`, `INDEX.md` or this file, which have no frontmatter. Also fail the build when
      `build_index.py` would change `INDEX.md` or `index.html` (a stale index).
- [ ] Publish with GitHub Pages from `main`. `index.html` is already the entry point. Pages is not
      enabled yet.
- [ ] Later: once a real agent-evals and AI-code-review pilot has results, add what it taught to
      `ai-retrieval-evals-cost` and `evidence-and-decisions` as general Lessons. Teach the
      method, not the project: name no project and no employer.
- **Done when:** all 13 docs are `final`, CI checks them, and the Pages URL serves `index.html`.
