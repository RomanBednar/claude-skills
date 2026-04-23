# test-plan

Write and execute structured test plans inspired by IEEE 829 and Google testing practices.

## Install

```
/plugins marketplace add RomanBednar/claude-skills
/plugins install testing-skill@rb-claude-skills
```

## What It Does

**Write phase** — interviews you about the feature or bug, auto-detects cluster environment via Kubernetes MCP, and produces a structured test plan document with environment requirements, reproducers (for bugs), and detailed test cases.

**Execute phase** — loads an existing test plan, verifies the cluster matches the plan's requirements, runs each test case (sequentially or in parallel), and generates an execution report. Each step records MCP tool calls, CLI equivalents, and pass/fail evidence. Bugs found get their own section with reproduction steps.

The skill never guesses — if anything is unclear, it asks before proceeding.

## Usage

### Write a test plan

```
/test-plan write
```

or with a description:

```
/test-plan write OVN egress firewall not blocking traffic on dual-stack clusters
```

The skill will:
1. Connect to your cluster and detect environment details
2. Interview you about the feature/bug
3. Draft a test plan using the IEEE 829-inspired template
4. Ask you to review and approve
5. Save the plan as a markdown file

### Execute a test plan

```
/test-plan execute test-plan-OCPBUGS-12345-egress-fw.md
```

or just:

```
/test-plan execute
```

(it will search the current directory for test plan files)

The skill will:
1. Validate the plan structure
2. Verify the cluster matches the plan's environment requirements
3. Ask you to approve each test case before running it
4. Execute steps using Kubernetes MCP tools (falling back to `oc`/`kubectl` CLI)
5. Generate a report with MCP commands, CLI equivalents, and a bugs section

## Prerequisites

- Kubernetes MCP server configured in Claude Code
- `oc` or `kubectl` CLI available
- Active kubeconfig pointing to the target cluster

## Structure

```
test-plan/
├── SKILL.md                    # Skill instructions
├── README.md                   # This file
└── references/
    ├── test-plan-template.md   # IEEE 829-inspired template
    └── report-template.md      # Execution report template
```
