---
name: test-plan
description: Write and execute structured test plans for OpenShift/Kubernetes features and bugs. Inspired by IEEE 829 and Google testing. Use when creating or running test plans. Triggers on "write test plan", "create test plan", "execute test plan", "run test plan", "test plan for".
argument-hint: <write|execute> [description or path-to-plan]
disable-model-invocation: true
allowed-tools: Read, Write, AskUserQuestion, Agent, Bash(git *), Bash(oc *), Bash(kubectl *), Bash(jq *), Bash(grep *), Bash(ls *), Bash(find *), mcp__kubernetes__*
---

# Test Plan — Write & Execute

## Core Principle

**Never guess. Always ask.** If any detail about the feature, bug, environment, or expected behavior is ambiguous, interview the user. A wrong assumption in a test plan changes what gets tested and gives false confidence. Prefer asking 5 clarifying questions over making 1 guess.

## Step 0: Cluster Connectivity Check

**This runs before anything else.** The skill assumes a cluster is already running. Verify connectivity and collect cluster metadata using Kubernetes MCP tools:

1. Call `mcp__kubernetes__nodes_list` to list cluster nodes
2. Call `mcp__kubernetes__configuration_view` to confirm the current context

**If both succeed:** Extract and remember the following for later use:
- Cluster version
- Platform (from node labels or infrastructure resource)
- Network type (from network operator config)
- Node count and roles
- Current context name

Report to the user:
> "Connected to cluster. Version: <version>, Platform: <platform>, Network: <network-type>, Nodes: <count> (<roles summary>). Proceeding."

This detected info is used in Phase 1 to pre-fill environment details, reducing interview questions.

**If either fails:** Stop and tell the user:
> "Cannot connect to the cluster via Kubernetes MCP. Make sure:
> 1. Your kubeconfig is at `~/.kube/config` (or `KUBECONFIG` env var is set)
> 2. The Kubernetes MCP server is configured in your Claude Code settings
> 3. The cluster is reachable from this machine
>
> Once fixed, re-run `/test-plan`."

**Do NOT proceed to any phase until the connectivity check passes.**

---

## Phase Routing

Read the first argument from  to determine which phase to run:

- **`write`** → Go to [Phase 1: Write Test Plan](#phase-1-write-test-plan)
- **`execute`** → Go to [Phase 2: Execute Test Plan](#phase-2-execute-test-plan)

If no argument or unclear, use `AskUserQuestion` to ask:
- Question: "What would you like to do?"
- Header: "Phase"
- Options: "Write a new test plan" / "Execute an existing test plan"

---

## Phase 1: Write Test Plan

### Step 1: Gather Context Through Interview

Before writing anything, understand what you're testing.

**Pre-fill from cluster detection:** Use the cluster metadata collected in Step 0 to pre-populate environment details. Present the detected values and ask the user to confirm or correct:

> "From the connected cluster, I detected:
> - OCP Version: <version>
> - Platform: <platform>
> - Network Type: <network-type>
> - Nodes: <count> (<roles>)
>
> Is this the target environment for the test plan, or are you testing against a different cluster?"

This eliminates most of question 3 below. Only ask about details the cluster doesn't reveal (e.g., specific install-config parameters).

**Remaining information to collect** (interview the user for what wasn't auto-detected):

1. **What is being tested?**
   - Feature name or bug ID/description
   - Link to PR, issue, spec, or BZ if available
   - Is this a new feature, regression, or bug fix?

2. **What is the expected behavior?**
   - For features: what should the feature do?
   - For bugs: what is the correct behavior vs. the broken behavior?

3. **Environment details not auto-detected:**
   - Any specific install-config parameters beyond what was detected
   - Any non-default configuration relevant to the test

4. **What components are involved?**
   - Operators, CRDs, APIs, controllers
   - Any specific node roles, machine sets, or infrastructure

5. **For bugs: can you describe the reproducer?**
   - Exact steps to reproduce the broken state
   - What error messages or symptoms appear?
   - Is it intermittent or consistent?

**Do NOT proceed to Step 2 until all mandatory information is collected.** Ask follow-up questions. It is better to ask 5 questions than to guess once.

### Step 2: Draft the Test Plan

Use the template from [test-plan-template.md](references/test-plan-template.md).

For each section, verify: "Am I stating a fact the user told me, or am I assuming?" If assuming, stop and ask.

Mark anything uncertain as `[TO BE DETERMINED — <what needs clarification>]`.

### Step 3: Review With User

Present the completed test plan and explicitly ask:

1. Are the environment requirements complete and accurate?
2. Are the test cases covering the right scenarios?
3. For bugs: does the reproducer in "Before State" match your experience?
4. Is anything missing or incorrect?

**Do not finalize until the user approves.**

### Step 4: Save the Test Plan

Save the approved test plan as a markdown file:
`test-plan-<feature-or-bug-id>-<short-description>.md`

Place it in the current working directory or a location the user specifies.

After saving, ask the user:
> "Test plan saved. Would you like to proceed to execution now? (`/test-plan execute <path>`)"

---

## Phase 2: Execute Test Plan

### Step 1: Load and Validate the Plan

Read the test plan file from the path provided in the arguments.

**If no path is provided or the file doesn't exist:** Search the current directory for test plan files:
```bash
find . -maxdepth 2 -name "test-plan-*.md" -type f
```
If files are found, present them via `AskUserQuestion` and let the user pick. If none are found, ask the user for the path.

**Validation checks — reject the plan if any fail:**
- [ ] Plan has a valid structure (Introduction, Environment, Test Cases, Pass/Fail Criteria)
- [ ] No `[TO BE DETERMINED]` items remain — if found, list them and ask the user to resolve before continuing
- [ ] Environment requirements are specific enough to verify against the current cluster

If validation fails, tell the user what needs to be fixed and stop.

### Step 2: Verify Environment

Before running any test, confirm the cluster matches the plan's requirements.

Use **Kubernetes MCP tools first**. Fall back to CLI (`oc`, `kubectl`) only when MCP tools don't cover the needed operation.

Check:
- Cluster version matches
- Required operators are installed and healthy
- Required namespaces, CRDs, or resources exist
- Node count and roles match expectations

Present the environment check results to the user:
> "Environment check complete. [N/M] requirements met. [details of any mismatches]"
> "Proceed with execution? (yes / fix environment first / abort)"

**Wait for user approval before continuing.**

### Step 3: Choose Execution Strategy

Before executing, ask the user how they want to run the test cases:

```
AskUserQuestion({
  questions: [
    {
      question: "How should test cases be executed?",
      header: "Strategy",
      options: [
        { label: "Sequential (Recommended)", description: "Run one at a time with approval before each. Best for first runs or when you want control over each step." },
        { label: "Parallel via subagents", description: "Run independent test cases simultaneously using subagents. Faster but requires batch approval upfront." }
      ],
      multiSelect: false
    },
    {
      question: "How should commands be recorded in the report?",
      header: "Recording",
      options: [
        { label: "MCP + CLI (Recommended)", description: "Record both MCP tool calls and CLI equivalents. Verbose but portable — anyone can reproduce with just CLI." },
        { label: "MCP only", description: "Record only MCP tool calls. Concise but requires MCP to reproduce." },
        { label: "CLI only", description: "Record only CLI commands. Most portable, easiest to share." }
      ],
      multiSelect: false
    }
  ]
})
```

Use the selected recording mode throughout all test case execution and in the final report.

### Step 4: Execute Test Cases

#### Sequential Mode

For each test case in the plan:

1. **Present the test case** to the user before executing:
   > "**Test Case TC-XX: <title>**"
   > "Priority: <priority>"
   > "Steps: <summary of what will be done>"
   > "Proceed? (yes / skip / abort all)"

2. **Wait for user approval.** Do not execute without it.

3. **Execute each step**, preferring MCP tools over CLI:
   - Try Kubernetes MCP tool first (e.g., `mcp__kubernetes__pods_list`, `mcp__kubernetes__resources_get`)
   - If MCP tool not available for the operation, fall back to `oc` / `kubectl` CLI
   - Record commands according to the selected recording mode

4. **Record results** for each step:
   - What was executed (per recording mode)
   - Description of what this step does
   - Expected result (from the plan)
   - Actual result (from the execution)
   - Pass/Fail determination

5. **If a step fails or produces unexpected output:**
   - Do NOT guess what went wrong
   - Present the actual output to the user
   - Ask: "This result differs from expected. Is this a bug, an environment issue, or should we adjust the test case?"

6. **Run cleanup** steps if specified in the test case

#### Parallel Mode (Subagents)

Parallel execution dispatches independent test cases to subagents that run simultaneously.

1. **Analyze dependencies:** Review all test cases and identify which ones are independent (no shared state, no ordering requirements). Test cases that create resources used by later tests are NOT independent.

2. **Present the execution plan** to the user:
   > "I identified <N> independent test cases that can run in parallel: TC-01, TC-03, TC-05.
   > These depend on each other and will run sequentially after: TC-02 → TC-04.
   > Approve this batch? (yes / adjust grouping / switch to sequential)"

3. **Wait for user approval** of the full batch before spawning any subagents.

4. **Spawn one subagent per independent test case** using the `Agent` tool. Each subagent receives:
   - The specific test case steps from the plan
   - The cluster context (from Step 0)
   - The selected recording mode
   - Instructions to save results to a structured output (pass/fail per step, actual results, evidence)
   - Instructions to run cleanup steps after completion

   Subagent prompt template:
   ```
   Execute this test case against the connected OpenShift/Kubernetes cluster:

   Test Case: <TC-ID>: <title>
   Priority: <priority>
   Steps: <full step details from plan>
   Recording mode: <MCP+CLI | MCP only | CLI only>

   For each step:
   1. Execute using Kubernetes MCP tools first, fall back to oc/kubectl CLI
   2. Record: command executed, expected result, actual result, pass/fail
   3. If a step fails, record the failure and continue to the next step
   4. Run cleanup steps when done

   Report back: overall pass/fail, per-step results with evidence.
   ```

5. **Collect results** as subagents complete. Present a summary to the user:
   > "Parallel execution complete:
   > - TC-01: PASS (3/3 steps)
   > - TC-03: FAIL (step 2 — unexpected output)
   > - TC-05: PASS (4/4 steps)
   >
   > TC-03 failed. Would you like to review the details?"

6. **Run dependent test cases sequentially** after the parallel batch completes, using the sequential mode described above.

7. **For any failures in parallel mode:** Present the actual output and ask the user for triage, same as sequential mode. The subagent records the failure but does not make judgments about root cause.

### Step 5: Generate Report

After all test cases are executed (or skipped/aborted), generate the execution report.

Use the template from [report-template.md](references/report-template.md).

**Report structure:**
- Execution summary (date, environment, overall result)
- Each test case in its own section with:
  - MCP instructions that were run (tool name + parameters)
  - CLI equivalent commands
  - Description of what each step does
  - Actual vs expected results
  - Evidence (command output)
- **Bugs Found** — separate section at the end with:
  - Bug ID, severity, title
  - Steps to reproduce
  - Related test case
  - Evidence and investigation hints

Save the report as: `test-report-<plan-id>-<YYYYMMDD>.md`

---

## Using AskUserQuestion

Use the `AskUserQuestion` tool whenever you need user input. This provides a structured interactive prompt instead of printing questions as text.

**When to use it:**
- Phase routing (write vs execute)
- Gathering interview context (test type, bug vs feature, platform, etc.)
- Approval gates (environment check, test case execution, report review)
- Failure triage (bug vs environment issue vs test adjustment)

**How to use it well:**
- Group up to 4 related questions in a single call to reduce back-and-forth
- Use `multiSelect: true` when choices aren't mutually exclusive (e.g., "Which components are involved?")
- Add `(Recommended)` to the first option when you have a strong default
- Use the `description` field to explain trade-offs or implications of each option
- The user can always select "Other" to provide free-text input, so don't try to enumerate every possibility — cover the most likely options

**Example — initial interview:**
```
AskUserQuestion({
  questions: [
    {
      question: "What type of testing is this?",
      header: "Test type",
      options: [
        { label: "Bug verification", description: "Verify a bug fix works correctly" },
        { label: "Feature verification", description: "Validate new feature behavior" },
        { label: "Regression", description: "Ensure existing functionality still works" }
      ],
      multiSelect: false
    },
    {
      question: "Which platform is the cluster running on?",
      header: "Platform",
      options: [
        { label: "AWS", description: "Amazon Web Services" },
        { label: "GCP", description: "Google Cloud Platform" },
        { label: "Azure", description: "Microsoft Azure" },
        { label: "Bare-metal", description: "Physical hardware or vSphere" }
      ],
      multiSelect: false
    }
  ]
})
```

**Example — execution approval:**
```
AskUserQuestion({
  questions: [{
    question: "Environment check complete. 4/5 requirements met. Worker node count is 2 (expected 3). Proceed?",
    header: "Env check",
    options: [
      { label: "Proceed anyway", description: "Run tests with current environment" },
      { label: "Fix environment first", description: "Pause while you adjust the cluster" },
      { label: "Abort", description: "Cancel test execution" }
    ],
    multiSelect: false
  }]
})
```

## Interview Principles (Both Phases)

- Use `AskUserQuestion` for all user interactions — never print questions as plain text
- Group related questions (up to 4) in a single call to reduce round-trips
- Repeat back your understanding before acting — "So to confirm, the bug is that X happens when Y, and the fix should make Z happen instead?"
- If the user says "I'm not sure" about something, mark it as `[TO BE DETERMINED]` — do not fill it in with a guess
- Prefer concrete examples over abstract descriptions — "Can you show me the YAML?" is better than "What resource is affected?"

## MCP Tool Priority

When executing commands against the cluster:

1. **First choice:** Kubernetes MCP tools (`mcp__kubernetes__*`)
2. **Second choice:** OpenShift MCP tools if Kubernetes MCP doesn't cover the operation
3. **Last resort:** CLI commands (`oc`, `kubectl`) via Bash

Always record the MCP tool call AND the CLI equivalent regardless of which was actually used.
