# Test Plan Template

Use this template structure when generating test plans. Every section is mandatory unless marked optional.

---

```markdown
# Test Plan: <Title>

| Field              | Value                                      |
|--------------------|--------------------------------------------|
| **Plan ID**        | TP-<YYYYMMDD>-<short-id>                   |
| **Author**         | <name>                                     |
| **Date Created**   | <date>                                     |
| **Last Updated**   | <date>                                     |
| **Status**         | Draft / In Review / Approved / Executed    |
| **Related Issue**  | <link to BZ, Jira, GitHub issue, or PR>    |
| **Type**           | Feature Verification / Bug Verification / Regression |

---

## 1. Introduction

### 1.1 Purpose
<One paragraph: what this test plan verifies and why it matters.>

### 1.2 Feature / Bug Description
<Detailed description of the feature or bug. For bugs, include:
- Reported behavior (what goes wrong)
- Expected behavior (what should happen)
- Root cause if known
- Link to the fix PR/commit>

### 1.3 Scope
**In Scope:**
- <bullet list of what this plan covers>

**Out of Scope:**
- <bullet list of what this plan does NOT cover>

### 1.4 References
- <Links to design docs, specs, upstream KEPs, RFCs, etc.>

---

## 2. Environment Requirements

### 2.1 OpenShift Cluster Specification

| Parameter              | Value                          |
|------------------------|--------------------------------|
| **OCP Version**        | <e.g., 4.17.0>                |
| **Platform**           | <AWS / GCP / Azure / bare-metal / vSphere / etc.> |
| **Network Type**       | <OVNKubernetes / OpenShiftSDN> |
| **Topology**           | <HA / SNO / Compact / Multi-node> |
| **Architecture**       | <x86_64 / arm64 / multi-arch> |
| **Worker Nodes**       | <count and instance type>      |
| **Control Plane Nodes**| <count and instance type>      |

### 2.2 Install Configuration
<Provide the relevant install-config.yaml parameters or the full file if needed.>

```yaml
# Key install-config parameters
apiVersion: v1
metadata:
  name: <cluster-name>
baseDomain: <domain>
platform:
  <platform-specific-config>
networking:
  networkType: <type>
  clusterNetwork:
    - cidr: <cidr>
      hostPrefix: <prefix>
  serviceNetwork:
    - <cidr>
```

### 2.3 Post-Install Setup
<Step-by-step setup required AFTER cluster installation but BEFORE test execution.>

1. <Step 1: e.g., "Install operator X from OperatorHub">
2. <Step 2: e.g., "Create namespace `test-ns`">
3. <Step 3: e.g., "Apply the following CRD/CR...">

For each step, provide:
- The exact command or YAML to apply
- How to verify the step succeeded (expected output or condition)

### 2.4 Required Tools and Access
- <CLI tools needed: oc, kubectl, jq, etc. with minimum versions>
- <Credentials or access levels required>
- <Any external dependencies: registries, DNS, load balancers>

---

## 3. Before State (Bug Verification Only)

> Skip this section for feature verification test plans.

### 3.1 Reproducer
<Step-by-step instructions to reproduce the bug on an UNPATCHED cluster.
Each step must be concrete — exact commands, exact YAML, exact API calls.>

1. <Step 1>
   ```bash
   <command>
   ```
   **Expected (buggy) result:** <what happens>

2. <Step 2>
   ...

### 3.2 Evidence of Bug
<What output, logs, events, or conditions confirm the bug is present?>

```
<example error output or log snippet>
```

### 3.3 Affected Components
- <List of components, pods, operators, or resources affected>

---

## 4. Test Cases

### Test Case <ID>: <Title>

| Field           | Value                                    |
|-----------------|------------------------------------------|
| **Priority**    | Critical / High / Medium / Low           |
| **Type**        | Positive / Negative / Boundary / Regression |
| **Prerequisite**| <any setup specific to this test case>   |

**Objective:** <What this test case verifies in one sentence.>

**Steps:**

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1    | <what to do — exact command or action> | <what should happen> |
| 2    | ... | ... |
| 3    | ... | ... |

**Cleanup:** <any teardown needed after this test case>

---

> Repeat "Test Case" section for each test case.

---

## 5. Pass / Fail Criteria

### 5.1 Overall Pass Criteria
- <condition 1: e.g., "All Critical and High priority test cases pass">
- <condition 2: e.g., "No regressions observed in related functionality">

### 5.2 Overall Fail Criteria
- <condition 1: e.g., "Any Critical test case fails">
- <condition 2: e.g., "Bug reproducer still succeeds after fix is applied">

---

## 6. Risks and Assumptions

### 6.1 Assumptions
- <e.g., "Cluster is healthy and all operators are in Available state before testing">

### 6.2 Risks
| Risk | Likelihood | Mitigation |
|------|------------|------------|
| <risk description> | High/Medium/Low | <how to handle> |

---

## 7. Open Questions

> List anything marked [TO BE DETERMINED] during the interview.
> These MUST be resolved before the test plan status moves to "Approved".

- [ ] <question 1>
- [ ] <question 2>
```
