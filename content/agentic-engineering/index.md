---
title: Agentic Engineering
tags:
  - agents
  - tooling
  - workflow
---

Some notes on how I work with coding agents day to day (as of Oct '26).

Currently, my setup is primarily based on how long / much effort is needed to complete the task. I tend to dedicate an autonomous loop that grinds overnight for long tasks, while short tasks get a well-equipped single session where alot of the orchestration work goes into feeding the agent the right context cheaply.

**Model:** GPT 6 Sol/ Claude Opus 5.5

## Long tasks

For anything open-ended (optimise Y, sweep X ideas, improve this metric), I run [gnhf](https://github.com/kunchenguid/gnhf).
It is a autoresearch style orchestrator that keeps agents running while I literally sleep!

It packages [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) pattern which iteratively runs 'research'.
The main idea behind it is basically a glorified loop: read the code, propose one small change, run it, measure, commit if better, roll back if worse, repeat. One good thing is that each successful iteration would result in a own git commit, so usually in the morning I can get a branch I can cherry-pick from and a log of every attempt including the failures.


## Short tasks

For scoped work in a single scoped session, these are the tooling I leverage:

**Code Docs.**
[Context7](https://github.com/upstash/context7) for up to date library documentation. I find it useful when using a niche software, where the LLM may not have been trained on.

**Web.**
[Jina MCP](https://github.com/jina-ai/MCP) to pull webpage content down as clean markdown. [Exa MCP](https://github.com/exa-labs/exa-mcp-server) gives the agent semantic web search and page fetching, plus Exa Agent runs for multi-step research. [Parallel](https://github.com/parallel-web/parallel-agent-skills) for a free search MCP (`web_search`, `web_fetch`) and skills for deep research, data enrichment, entity discovery and web monitoring.

**Browser.**
[Playwright MCP](https://github.com/microsoft/playwright-mcp) for web automation and E2E testing, so the agent can drive a real browser and verify the change.

**Codebase.**
[codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) indexes the repo into a persistent knowledge graph of functions, classes, call chains and routes.
It can answer structural questions (who calls this, what breaks if I change it, what is dead) based on the graph in a few 100 tokens. This is much cleaner as opposed to a `grep` that just dumps 80k LoC into the context.

**Frontend.**
The [shadcn MCP](https://ui.shadcn.com/docs/mcp) plus its React skills, where frontend components get pulled from the registry.

**Cloud and infrastructure.**
For cloud tooling, I use [Supabase MCP](https://github.com/supabase-community/supabase-mcp) for database work, so schema, migrations and queries happen against the project. Often, I also find myself using the [gcloud MCP](https://github.com/googleapis/gcloud-mcp) which covers the GCP side by driving the `gcloud` CLI.


**GitHub.**
I use [gh-axi](https://github.com/kunchenguid/gh-axi), a wrapper around `gh` built on the [AXI](https://github.com/kunchenguid/axi) design principles. The idea is that it emits TOON (Token-Oriented Object Notation) instead of the regular JSON (which agents usually tend to interact with). This is roughly 40% cheaper for the same payload, and returns structured errors with next-step hints.

But don't take my word! Its published benchmarks put it at 100% vs 86% task success against raw `gh`, and 66% cheaper than the GitHub MCP server with half the turns.

**Learning a codebase.**
I find the `grill-me` skill particularly useful where it gets the agent to relentlessly gather and ask me questions about the project I'm working on. I use it often to onboard into a new or unfamiliar codebase.

## Shipping

I run [no-mistakes](https://github.com/kunchenguid/no-mistakes) which is a local git proxy. It basically spins up a disposable worktree, runs an AI naitve validation pipeline, and forwards the branch only after every check/test passes. The pipeline is usually standard with a fixed flow: intent, rebase, review, test, document, lint, push, PR, CI.

**Git worktrees**
I ocassionally spin and orchestrate some git worktrees for multiple agents to work in parallel too. To do that, I tend to keep a reusable `.sh` script that agents invoke themselves to create the worktree and bring the environment up with it, provisioning the database, pulling secrets and setting up the runtime in one command. I find this pretty useful when I have multiple problems to debug or features to implement


## References

| Tool | Repo |
|------|------|
| gnhf | [kunchenguid/gnhf](https://github.com/kunchenguid/gnhf) |
| autoresearch | [karpathy/autoresearch](https://github.com/karpathy/autoresearch) |
| Context7 | [upstash/context7](https://github.com/upstash/context7) |
| Jina MCP | [jina-ai/MCP](https://github.com/jina-ai/MCP) |
| Exa MCP | [exa-labs/exa-mcp-server](https://github.com/exa-labs/exa-mcp-server) |
| Parallel | [parallel-web/parallel-agent-skills](https://github.com/parallel-web/parallel-agent-skills) |
| Playwright MCP | [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) |
| codebase-memory-mcp | [DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) |
| shadcn/ui | [shadcn-ui/ui](https://github.com/shadcn-ui/ui) |
| Supabase MCP | [supabase-community/supabase-mcp](https://github.com/supabase-community/supabase-mcp) |
| gcloud MCP | [googleapis/gcloud-mcp](https://github.com/googleapis/gcloud-mcp) |
| gh-axi | [kunchenguid/gh-axi](https://github.com/kunchenguid/gh-axi) |
| AXI principles | [kunchenguid/axi](https://github.com/kunchenguid/axi) |
| no-mistakes | [kunchenguid/no-mistakes](https://github.com/kunchenguid/no-mistakes) |
