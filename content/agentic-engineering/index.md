---
title: Agentic Engineering
tags:
  - agents
  - tooling
  - workflow
---

Notes on how I work with coding agents day to day.

My setup splits based on how long / much effort is needed to complete the task. Long tasks get an autonomous loop that grinds overnight, while short tasks get a well-equipped single session where most of the effort goes into feeding the agent the right context cheaply.

## Long tasks

For anything open-ended (optimise this, sweep these ideas, improve this metric), I run [gnhf](https://github.com/kunchenguid/gnhf).
It is a autoresearch-style orchestrator that keeps agents running while I literally sleep.

It packages [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) pattern so it can point at any project.
The loop is the whole idea: read the code, propose one small change, run it, measure, commit if better, roll back if worse, repeat.
Each successful iteration lands as its own git commit, so in the morning I get a branch I can cherry-pick from and a log of every attempt including the failures.


## Short tasks

For scoped work in a single session, these are the tooling I leverage:

**Docs.**
[Context7](https://github.com/upstash/context7) for up to date library documentation, which kills the "confidently wrote an API that was deprecated two versions ago" failure mode.

**Web.**
Websearch for discovery, and the [Jina MCP](https://github.com/jina-ai/MCP) to actually pull page content down as clean markdown when I need the full source rather than a snippet.

**Browser.**
[Playwright MCP](https://github.com/microsoft/playwright-mcp) for web automation and E2E testing, so the agent can drive a real browser and verify the change instead of asserting that it should work.

**Codebase.**
[codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) indexes the repo into a persistent knowledge graph of functions, classes, call chains and routes.
Structural questions (who calls this, what breaks if I change it, what is dead) get answered from the graph in a few hundred tokens instead of a grep that dumps 80k.
This is the single biggest token saver in the setup.

**Frontend.**
The [shadcn MCP](https://ui.shadcn.com/docs/mcp) plus its React skills, so components get pulled from the registry and wired up properly rather than hand-rolled from memory.

**Cloud and infrastructure.**
For cloud tooling, I use [Supabase MCP](https://github.com/supabase-community/supabase-mcp) for database work, so schema, migrations and queries happen against the project. Often, I also find myself using the [gcloud MCP](https://github.com/googleapis/gcloud-mcp) which covers the GCP side by driving the `gcloud` CLI.


**GitHub.**
[gh-axi](https://github.com/kunchenguid/gh-axi), a wrapper around `gh` built on the [AXI](https://github.com/kunchenguid/axi) design principles.
It emits TOON (Token-Oriented Object Notation) instead of JSON, which is roughly 40% cheaper for the same payload, and returns structured errors with next-step hints.
The published benchmarks put it at 100% vs 86% task success against raw `gh`, and 66% cheaper than the GitHub MCP server with half the turns.
Same idea as the knowledge graph: the win comes from the output format, not from a smarter model.

**Learning a codebase.**
I use the `grill-me` skill which gets the agent to relentless gather and ask me questions so that it has a full scope of requirements without ambiguity. I found it useful for onboarding onto unfamiliar codebase

## Shipping: the gate

I run [no-mistakes](https://github.com/kunchenguid/no-mistakes) which is a local git proxy. It spins up a disposable worktree, runs an AI-driven validation pipeline, and forwards the branch to the configured push target only after every check passes.

The pipeline is fixed and opinionated: intent, rebase, review, test, document, lint, push, PR, CI.

**Git worktrees**
I ocassionally spawn git worktrees for multiple agents tow ork in parallel too. To do that, I keep a reusable `.sh` script that agents invoke themselves to create the worktree and bring the environment up with it, provisioning the database, pulling secrets and setting up the runtime in one command.


## References

| Tool | Repo |
|------|------|
| gnhf | [kunchenguid/gnhf](https://github.com/kunchenguid/gnhf) |
| autoresearch | [karpathy/autoresearch](https://github.com/karpathy/autoresearch) |
| Context7 | [upstash/context7](https://github.com/upstash/context7) |
| Jina MCP | [jina-ai/MCP](https://github.com/jina-ai/MCP) |
| Playwright MCP | [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) |
| codebase-memory-mcp | [DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) |
| shadcn/ui | [shadcn-ui/ui](https://github.com/shadcn-ui/ui) |
| Supabase MCP | [supabase-community/supabase-mcp](https://github.com/supabase-community/supabase-mcp) |
| gcloud MCP | [googleapis/gcloud-mcp](https://github.com/googleapis/gcloud-mcp) |
| gh-axi | [kunchenguid/gh-axi](https://github.com/kunchenguid/gh-axi) |
| AXI principles | [kunchenguid/axi](https://github.com/kunchenguid/axi) |
| no-mistakes | [kunchenguid/no-mistakes](https://github.com/kunchenguid/no-mistakes) |
