# Test Execution Report Template

Use this template when generating execution reports after running test cases.

---

```markdown
# Test Execution Report: <Test Plan Title>

| Field                | Value                                      |
|----------------------|--------------------------------------------|
| **Report ID**        | TR-<YYYYMMDD>-<short-id>                   |
| **Test Plan ID**     | <reference to the plan>                    |
| **Executed By**      | <name> + Claude Code                       |
| **Execution Date**   | <date>                                     |
| **Cluster**          | <cluster name / API URL>                   |
| **OCP Version**      | <actual version from `oc version`>         |
| **Overall Result**   | **PASS** / **FAIL** / **PARTIAL**          |

---

## Execution Summary

| Metric          | Count |
|-----------------|-------|
| Total Cases     | <N>   |
| Passed          | <N>   |
| Failed          | <N>   |
| Skipped         | <N>   |
| Blocked         | <N>   |
| Bugs Found      | <N>   |

### Environment Verification
<Summary of environment check results. Note any deviations from the plan.>

---

## Test Case Results

### TC-<ID>: <Title>

**Result:** PASS / FAIL / SKIPPED / BLOCKED
**Priority:** Critical / High / Medium / Low

#### Steps Executed

##### Step 1: <Description of what this step does>

**MCP Tool Used:**
```
Tool: mcp__kubernetes__<tool_name>
Parameters: {
  "param1": "value1",
  "param2": "value2"
}
```

**CLI Equivalent:**
```bash
oc <command> / kubectl <command>
```

**Expected Result:** <from the test plan>
**Actual Result:** <what actually happened>
**Status:** PASS / FAIL

<details>
<summary>Command Output</summary>

```
<raw output from the command>
```

</details>

---

##### Step 2: <Description of what this step does>
...

---

> Repeat for each step in the test case.
> Repeat the entire "TC-<ID>" section for each test case.

---

## Bugs Found

> This section lists all bugs discovered during execution.
> If no bugs were found, write: "No bugs found during this execution."

### BUG-<N>: <Short title>

| Field              | Value                                      |
|--------------------|--------------------------------------------|
| **Severity**       | Critical / High / Medium / Low             |
| **Related Test**   | TC-<ID>: <title>                           |
| **Step**           | Step <N>                                   |
| **Reproducible**   | Always / Intermittent / Once               |

**Description:**
<What went wrong — factual, not interpretive.>

**Steps to Reproduce:**
1. <exact steps, including commands>
2. ...

**Expected Behavior:**
<What should have happened.>

**Actual Behavior:**
<What actually happened.>

**Evidence:**
```
<relevant logs, error messages, or command output>
```

**Investigation Hints:**
- <Pointers for where to look: specific pods, logs, controllers, CRDs>
- <Related components or recent changes that might be involved>

---

> Repeat for each bug found.

---

## Conclusion

<1-2 paragraph summary:
- Did the test plan pass or fail overall?
- Which test cases failed and why?
- Are the bugs blocking or non-blocking?
- Recommendation: ready to ship, needs fixes, needs more testing>
```
