# Draft 2: Crash-to-Fix Workflow for Claude Code

I think this is a much more powerful idea than a normal Crashlytics MCP integration. I would build it as a **Crash-to-Fix workflow for Claude Code**: an AI-assisted debugging process that starts with a real production failure, attempts to reproduce it automatically, creates a regression test, and proposes a targeted fix.

The key is to make Claude prove as much of its diagnosis as possible through executable tests, rather than simply reading a stack trace and generating a plausible code change.

There is already a good foundation: Firebase provides the `crashlytics:connect` guided MCP workflow for prioritizing, investigating, and fixing issues in the codebase. Your custom skill would add a disciplined reproduction-and-validation pipeline around it.

## 1. The proposed tool: Crash-to-Fix

1. **Collect production evidence**
   Crashlytics issues, events, stack traces, breadcrumbs and available logs
2. **Investigate the source**
   Find relevant code, dependencies, build version and recent changes
3. **Generate a reproduction**
   Build a minimal failing test or a UI/device automation scenario
4. **Confirm the failure**
   Run the test against the original code and capture the result
5. **Implement a minimal fix**
   Address the diagnosed cause without unrelated refactoring
6. **Validate and report**
   Pass the regression test, run relevant checks, summarize evidence and remaining risks

The distinction between steps 3 and 4 is especially important. Claude should not just write a test that looks reasonable. It should first demonstrate that the test fails on the original implementation, when reproducible, and then demonstrate that it passes after the proposed fix.

Some production crashes cannot be reproduced from Crashlytics data alone. In those cases, the workflow should report the missing evidence instead of fabricating a reproduction or claiming success.

## 2. How I'd implement it in Claude Code

I would use three pieces, rather than putting everything into one enormous prompt.

### A. Firebase MCP — the data source

Use Firebase's official MCP server to retrieve Crashlytics issues, representative events, exception information, and available issue context. Its documented workflow also supports investigation notes and issue-status management.

### B. A custom Claude Code skill — the investigation logic

Create `.claude/skills/crash-to-fix/SKILL.md`. It defines the investigation phases, required evidence, test-first process, validation criteria, and reporting format. Claude Code supports project-level skills in this directory structure.

### C. Test and reproduction tooling — the execution layer

Use the repository's existing build and test commands, emulators, integration-test framework, and debugging tools. Add small helper scripts only where they provide reliable, repeatable functionality.

I would start with Android and Kotlin if that's your stack, then extend the approach to iOS or Flutter. Each platform needs its own strategy for reproducing failures, controlling the environment, and executing tests.

## 3. What should happen in each phase?

| Phase | Claude should do | Required outcome |
| --- | --- | --- |
| **Collect** | Retrieve the selected issue and relevant events | An evidence package |
| **Diagnose** | Trace stack frames into source code and check build identity | Ranked root-cause hypotheses |
| **Reproduce** | Choose a unit, integration, UI, or device-level test | A repeatable scenario, or a documented blocker |
| **Prove** | Run the test before changing production code | A confirmed failure, where reproducible |
| **Fix** | Make the smallest justified code change | A focused patch |
| **Verify** | Run the regression test and related checks | Recorded results, including failures |
| **Report** | Summarize the cause, code changes, and validation | A review-ready report |

The evidence package should include the Crashlytics issue identifier, stack trace, affected app version, available breadcrumbs and custom keys, relevant source locations, and any limitations in the collected telemetry.

One subtle but important point: Crashlytics events are not automatically a complete device log or a recording of every interaction. Your app must have captured useful logs, keys, or breadcrumbs, and other logging or session-replay systems may be necessary for deeper reproduction.

## 4. The most valuable feature: automatic reproduction strategies

I would make this the central intelligence of the skill.

Instead of always generating the same kind of test, Claude should select a strategy based on the failure.

### Null pointer, parsing, or business-logic exception

Generate a focused unit test with the input conditions that trigger the failing method. Prefer this when the failure can be isolated without running the full app.

### Networking or backend-response failure

Use a deterministic mock server, fixture, or mocked repository to reproduce malformed responses, timeouts, empty data, or unexpected status codes.

### Navigation, lifecycle, or UI crash

Generate an instrumented UI test that performs the relevant user actions, navigation, background/foreground transitions, or activity recreation.

### Device- or OS-specific crash

Select a compatible emulator or device configuration, where available, and reproduce with the affected OS version, screen settings, or other relevant environmental conditions.

### Insufficient evidence

Identify missing logs or inputs and recommend additional instrumentation. Do not invent user actions or report an unverified scenario as a successful reproduction.

This makes the skill more valuable than a crash-summary generator: it can produce an executable artifact tailored to the actual failure.

## 5. Define the skill's contract

I'd give the skill an explicit invocation, such as:

```bash
/crash-to-fix <Crashlytics issue ID or URL>
```

Its instructions should require Claude to:

- Inspect the repository's existing state and avoid overwriting unrelated changes.
- Retrieve only the relevant Crashlytics data and redact sensitive information from reports.
- Inspect the affected code and its dependencies before selecting a root cause.
- Choose the smallest viable reproduction strategy and create a regression test where possible.
- Run the reproduction against the original implementation before applying the fix, when feasible.
- Preserve the original failure evidence, implement a minimal fix, and rerun the test.
- Report exactly which commands ran, which tests passed or failed, and what remains unverified.
- Request approval for destructive actions, changes to remote issue status, or deployment.

The skill should not require certainty at every stage. It should have explicit outcomes such as `reproduced`, `fix validated`, `partially validated`, and `unable to reproduce`.

## 6. The final report Claude should produce

Every run should leave behind a useful engineering artifact. For example:

### Crash-to-Fix investigation report

**Issue:** Checkout response exception  
**Root cause:** The response mapper does not handle a missing optional field. This finding is confirmed only if supported by the captured stack trace and source inspection.  
**Reproduction:** A regression test uses a response fixture with the relevant field missing. The test result against the original code is recorded.  
**Proposed fix:** Handle the missing field explicitly while preserving the intended behavior for valid responses.  
**Validation:**  
- Regression test: result recorded from execution  
- Related tests: results recorded from execution  
- Build: result recorded from execution  

The assistant must fill in results from actual tool output. It must never mark a test as passed merely because the test was generated.

## 7. What I'd build first

Start with an MVP that accepts one Crashlytics issue and performs the full investigation locally. Make it generate a report and a regression test before trying to automate everything else.

The first milestone is not autonomous fixing. It is this: given a real production crash, Claude can locate the relevant code, produce a credible reproduction, demonstrate the failure, propose a focused fix, and verify the change.

Once this works reliably, add automatic issue selection, Git history correlation, pull-request creation, and post-release checks.

That sequence will give you a much more reliable engineering workflow than immediately trying to fix every incoming crash automatically.

The next concrete artifact should be the project's `SKILL.md`, together with a reproducible test workflow tailored to the mobile framework. That is where this concept becomes a repeatable tool your team can use every day.

---

## 🎤 Talk / CFP Submission

# Stop Debugging Crashes — Let Firebase MCP Fix It!

What if production crash reports could become automated reproduction tests and verified fixes—without spending hours investigating stack traces?

In this talk, we’ll explore how to combine **Firebase Crashlytics, MCP, and AI models** to build an AI-powered crash-to-fix workflow. We’ll see how AI can retrieve crash data, analyze stack traces, identify root causes, generate automated reproduction scenarios, implement targeted fixes, and run tests to verify them. We’ll also explore running the workflow automatically overnight, with appropriate safeguards and human review.

### Key Takeaways
- Connect AI models to Firebase Crashlytics using MCP.
- Automate crash investigation and root-cause analysis.
- Generate reproducible scenarios and automated tests.
- Run crash-to-fix workflows overnight.
- Implement fixes and regression tests, then verify results.
- Build a repeatable, safer path from production crash to verified fix.
