# Subagent Router

A lightweight Codex plugin that adds one skill: `subagent-router`.

The skill helps Codex keep the main session small by routing suitable work to
isolated subagents. It is meant for multi-file changes, broad code search,
test triage, independent failures, review work, and `go on` style end-to-end
execution.

## What It Does

`subagent-router` teaches Codex to check whether a task should be delegated
before doing context-heavy work inline.

It favors:

- `explorer` subagents for read-only codebase questions.
- `worker` subagents for bounded implementation, testing, and review tasks.
- Small, explicit subagent prompts with clear scope and return shape.
- Keeping only the result, changed files, verification, and risks in the main
  session.

It avoids delegation for single small edits, design questions that need user
judgment, shared file scopes, secrets, and high-risk global configuration.

## Install

Clone the plugin into the personal plugin location:

```bash
mkdir -p ~/plugins
git clone https://github.com/xxx1766/subagent-router.git ~/plugins/subagent-router
```

Make sure the personal marketplace file contains this entry:

```json
{
  "name": "subagent-router",
  "source": {
    "source": "local",
    "path": "./plugins/subagent-router"
  },
  "policy": {
    "installation": "AVAILABLE",
    "authentication": "ON_INSTALL"
  },
  "category": "Productivity"
}
```

The default marketplace file is:

```text
~/.agents/plugins/marketplace.json
```

Then install the plugin:

```bash
codex plugin add subagent-router@personal
```

Start a new Codex session after installing so the skill list is refreshed.

## Verify

Check that the plugin is installed:

```bash
codex plugin list | rg 'subagent-router'
```

Check that the skill appears in a new prompt context:

```bash
codex debug prompt-input 'test subagent router' | rg 'subagent-router'
```

## Use

The skill can trigger implicitly when a task mentions subagents, delegation,
parallel search, multi-file changes, unrelated failures, tests, reviews, or
`go on`.

You can also invoke it explicitly:

```text
Use $subagent-router while working on this task.
```

Good trigger examples:

```text
Go on: implement the next useful fix, test it, commit it, and push.
```

```text
Find where this API is used and summarize the relevant call sites.
```

```text
Several unrelated test files are failing. Split the investigation cleanly.
```

## Notes

This plugin does not require Ruflo, MCP setup, or hook configuration. It is a
skill-level routing rule for Codex's native subagent capability.

Hooks or MCP orchestration can be added later, but the first version stays
small on purpose: it reminds the main session to delegate at the right time
without adding another runtime.
