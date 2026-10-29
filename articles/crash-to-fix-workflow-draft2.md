# Draft 2: Crash-to-Fix Workflow & Talk Proposal

This draft builds upon [Draft 1](crash-to-fix-workflow-draft1.md) by extending the **Crash-to-Fix Workflow** into a community presentation and adding automated overnight execution concepts.

## 1. Overview & Evolution from Draft 1

In [Draft 1](crash-to-fix-workflow-draft1.md), we outlined the core 6-step Crash-to-Fix pipeline:
1. **Collect** production evidence from Firebase Crashlytics MCP.
2. **Investigate** relevant source code and recent commits.
3. **Generate** minimal reproduction scenarios (unit, integration, or UI tests).
4. **Confirm** the test fails on the unpatched codebase.
5. **Implement** a targeted, minimal fix.
6. **Validate** that the regression test passes and summarize findings.

Draft 2 extends this concept by introducing **overnight automated runs** with safeguards and turning the workflow into a talk proposal.

## 2. Talk Proposal: Stop Debugging Crashes — Let Firebase MCP Fix It!

> **Full Talk Details:** See [`talks/stop-debugging-crashes.md`](../talks/stop-debugging-crashes.md)

### Abstract

What if production crash reports could become automated reproduction tests and verified fixes—without spending hours investigating stack traces?

In this talk, we’ll explore how to combine **Firebase Crashlytics, MCP, and AI models** to build an AI-powered crash-to-fix workflow. We’ll see how AI can retrieve crash data, analyze stack traces, identify root causes, generate automated reproduction scenarios, implement targeted fixes, and run tests to verify them. We’ll also explore running the workflow automatically overnight, with appropriate safeguards and human review.

### Key Takeaways

- Connect AI models to Firebase Crashlytics using MCP.
- Automate crash investigation and root-cause analysis.
- Generate reproducible scenarios and automated tests.
- Run crash-to-fix workflows overnight.
- Implement fixes and regression tests, then verify results.
- Build a repeatable, safer path from production crash to verified fix.
