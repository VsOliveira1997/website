---
title: "How to Add Skills and MCPs to AI Agents"
date: 2026-04-16
categories: [AI, Tooling]
tags: [ai, skills, mcp, agents, automation]
image: /assets/img/posts/skills-mcp-linkedin.png
---

Most people still try to solve everything with a prompt. That works for short tasks, but it becomes very confusing when the AI needs to repeat a workflow, follow a pattern, or interact with external systems.

If you want to improve an agent, there are usually two complementary paths:

- `skills`: teach the AI *how* to handle a class of tasks
- `MCPs`: a bridge between the AI and the program that is running, like the Ghidra MCP example

In practice, a skill shapes behavior. An MCP exposes capability.

## What a skill is

A skill is a reusable instruction package. Instead of relying on a long prompt every time, you define a focused operating guide for a specific job:

- when the skill should be used
- which steps the agent should follow
- which constraints matter
- what kind of output should come back

A good skill reduces randomness. If you keep asking the AI to perform the same type of task, that pattern should probably be turned into a skill.

Examples:

- reviewing code for regressions
- triaging bug bounty findings
- writing reports in a fixed structure
- analyzing logs and returning prioritized hypotheses

A simple `SKILL.md` can look like this:

```md
---
name: api-review
description: Review API endpoints for auth, validation, and data exposure issues.
---

Use this skill when the user asks for API review, quick threat modeling, or logic flaw hunting.

Workflow:
1. Map routes and HTTP verbs.
2. Identify authentication and authorization per endpoint.
3. Check for IDOR, BFLA, mass assignment, and weak validation.
4. Prioritize findings by impact.
5. Respond with findings, evidence, and next tests.

Rules:
- Do not assume controls without evidence.
- Separate facts from hypotheses.
- Keep the output concise and direct.
```

That alone already changes how an agent behaves, because it stops improvising from scratch and starts following a playbook.

## What an MCP is

MCP, or `Model Context Protocol`, is a standard way to connect models and agents to external tools and data sources. Instead of building a custom integration for every environment, MCP gives you a consistent interface for exposing things like:

- tools
- resources
- prompts

In practice, that means the AI can fetch live context, call actions, and interact with real systems with less glue code and less ad hoc behavior.

If you built your own MCP, it can act as the bridge between the model and whatever already exists in your environment:

- an internal API
- an admin panel
- a read-only database
- repositories
- documentation
- systems for CRM, ads, observability, or automation

That is the real jump in usefulness: the AI stops depending only on the current conversation and starts working with live context.

## Skills and MCPs solve different problems

This is where a lot of people get confused.

A skill does not replace an MCP, and an MCP does not replace a skill.

- A `skill` defines when to use an approach and how to execute it.
- An `MCP` provides the tools that make the approach actionable.

Together, they make an agent far more consistent.

A simple way to think about it:

- without a skill, the AI may have access to a tool but use it poorly
- without an MCP, the AI may know the process but have no reliable way to act outside the chat
- with both, it knows what to do and has the means to do it

## How to add skills to an AI

The implementation varies by product, but the pattern is usually the same:

1. Create a skill file or skill directory.
2. Write the description, usage triggers, and workflow.
3. Define scope, rules, and output expectations.
4. Test it on real tasks and refine it.

The most common mistake is making the skill too generic. Good skills are narrow enough to activate clearly and broad enough to be reused.

A minimal structure usually includes:

- name
- description
- when to use it
- steps
- constraints
- response format

If your agent supports local skills, the structure often ends up looking like this:

```text
~/.codex/skills/
  api-review/
    SKILL.md
```

or inside the project itself:

```text
.ai/
  skills/
    api-review.md
```

The exact folder name matters less than the quality of the instruction. It needs to be specific, operational, and easy for the agent to apply. It is also worth being careful with the number of skills: depending on the AI, too many skills can consume extra tokens and hurt the quality of the response.

## How to add an MCP to an AI

This tends to be more straightforward than people expect. You usually need three pieces:

1. An MCP server.
2. A client that supports MCP.
3. A configuration telling the AI where that server lives.

Depending on your stack, the MCP server might be a local binary, a Node.js script, a Python process, or a remote service.

One real example is [GhidraMCP](https://github.com/lauriewired/ghidramcp), which connects an AI client to Ghidra for reverse engineering workflows. A configuration can look like this:

```toml
[mcp_servers.ghidra]
command = "python"
args = [
  "/path/to/bridge_mcp_ghidra.py",
  "--ghidra-server",
  "http://127.0.0.1:8080/"
]
```

After that, the AI can discover and use the tools exposed by that server.

In `Claude Code`, the same integration pattern can be shared in the repository with a `.claude.json` file:

```json
{
  "mcpServers": {
    "ghidra": {
      "command": "python",
      "args": [
        "/path/to/bridge_mcp_ghidra.py",
        "--ghidra-server",
        "http://127.0.0.1:8080/"
      ]
    }
  }
}
```

In the case of `GhidraMCP`, this becomes easier to visualize because it exposes operations that map directly to reverse engineering workflows, such as:

- decompiling and analyzing binaries in Ghidra
- automatically renaming methods and data
- listing methods, classes, imports, and exports

That kind of structure makes it easier to understand what the AI can call, which parameters each operation expects, and what kind of result the MCP server will return.

## The best setup: let the skill orchestrate the MCP

This is usually the most effective setup in practice.

The MCP exposes the tools, and the skill tells the agent:

- when to query the system
- in which order to call tools
- how to validate the returned data
- when to ask for user confirmation
- how to summarize the final answer

That is the difference between an improvised assistant and an agent that is actually usable in day-to-day work.

If I were building this for a security or operations workflow, the flow would be something like:

1. The skill identifies the user intent.
2. The skill decides whether MCP access is needed.
3. The agent calls the correct tool.
4. The skill normalizes the result.
5. The final answer is returned in a consistent format.

## Where these integrations usually fail

The same problems show up over and over:

- skills that are too long and too vague
- MCP tools that try to do too much
- poor tool naming
- inconsistent return formats
- weak validation or authentication boundaries
- no rule for when the agent must ask before taking action

If you want something reliable, think like an interface designer, not just like a prompt writer.

## Closing

At the end of the day, the model gets more useful when it has both structure and access. Skills give it a repeatable way to think through a task, and MCPs give it a safe way to reach real tools and real data. That combination is what turns an AI from a better chatbot into a practical working agent.

## References

- [Anthropic: Intro to MCP](https://modelcontextprotocol.io/introduction)
- [Anthropic: Model Context Protocol overview](https://www.anthropic.com/news/model-context-protocol)
- [Subframe Docs: Agent skills](https://docs.subframe.com/guides/skills)
