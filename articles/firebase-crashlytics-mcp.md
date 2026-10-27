# Firebase Crashlytics MCP: Bringing AI-Powered Debugging into Everyday Mobile Development

> Originally published on [Medium](https://medium.com/@ashrafatef843/firebase-crashlytics-mcp-bringing-ai-powered-debugging-into-everyday-mobile-development-c9425fc93a08) and shared on [LinkedIn](https://lnkd.in/p/dbWhhQWn).

## Introduction

Debugging production issues is an essential part of mobile development, but finding the cause of a crash is rarely just about reading a stack trace. Developers need to understand the affected app version, the execution path, the code behind the failure, recent changes, and whether similar issues have already been investigated.

Firebase Crashlytics provides valuable visibility into crashes and non-fatal exceptions. However, developers often need to switch between Crashlytics, their IDE, source control, task trackers, and test reports to connect all the relevant information.

What if an AI assistant could investigate these issues directly from the development environment?

This is the idea behind **Firebase Crashlytics integration through the Model Context Protocol (MCP)**. It allows compatible AI development tools to access Crashlytics information and use it to help developers prioritize issues, investigate failures, and work toward fixes.

Instead of treating Crashlytics as a dashboard developers visit only when something goes wrong, we can make production stability a regular part of the development workflow.

Official documentation: [AI assistance for Crashlytics via MCP](https://firebase.google.com/docs/crashlytics/ai-assistance-mcp).

## 1. What is MCP, and why does it matter?

The Model Context Protocol is a standardized way for AI applications to access external tools and data sources.

An AI assistant can explain code, suggest fixes, and reason about stack traces, but its usefulness depends on the information it can access. Without integrations, developers must manually copy error messages, paste code, and provide context from other systems.

MCP provides a common interface through which compatible AI tools can access capabilities exposed by external services.

In mobile development, these services might include:

- Firebase Crashlytics for production issues and exception details.
- Git repositories for commits, branches, and code changes.
- Issue trackers for active tasks, acceptance criteria, and bug reports.
- CI systems for build results, test execution, and release metadata.

For example, an AI assistant with access to Crashlytics can retrieve an issue and examine its stack trace. With repository access, it can then inspect the corresponding source code. With task-tracker access, it can connect the investigation to existing engineering work.

MCP does not automatically provide every integration or guarantee that the AI reaches the correct conclusion. It provides the standardized connection that makes this broader workflow possible.

## 2. What does Firebase Crashlytics MCP provide?

Firebase provides an official MCP server that exposes Crashlytics capabilities to compatible AI development environments, including tools such as Antigravity, Claude Code, and Cursor.

The integration supports two main ways of working.

**Guided debugging:** The `crashlytics:connect` workflow helps the assistant prioritize issues, investigate a selected problem, suggest potential fixes, and support the debugging process.

**Conversational debugging:** Developers can ask questions in natural language. The assistant uses available Crashlytics tools to retrieve relevant information and investigate an issue.

The documented toolset includes capabilities to retrieve issue details, query recent events, inspect selected crash or exception events, obtain aggregated reports, read and manage issue notes, and update issue status.

This means the assistant can do more than explain an error message pasted into a chat. It can retrieve actual Crashlytics context and use that evidence during the investigation.

The integration is currently marked **experimental** by Firebase. Its behavior and supported capabilities may change, so teams should verify the current documentation before relying on it for critical workflows.

## 3. Getting started

The basic prerequisites are a working Node.js and npm installation, a compatible AI development tool, a Firebase project using Crashlytics, and suitable Firebase authentication.

The Firebase MCP server can be configured in supported tools using a command such as:

```json
{
  "mcpServers": {
    "firebase": {
      "command": "npx",
      "args": [
        "-y",
        "firebase-tools@latest",
        "mcp"
      ]
    }
  }
}
```

The exact configuration depends on the AI tool being used; Firebase also documents tool-specific installation options and a guided setup workflow.

After configuring the server, authenticate with the appropriate Firebase account or application credentials and confirm that the assistant can access the intended project.

Start the guided workflow by running:

```bash
crashlytics:connect
```

If the selected AI tool does not support the guided command, free-form conversation with the available Crashlytics MCP tools is an alternative.

For the full setup instructions, see the [official Firebase guide](https://firebase.google.com/docs/crashlytics/ai-assistance-mcp).

## 4. Practical daily workflows

The biggest opportunity is to use Crashlytics MCP regularly, rather than waiting for a production incident to become urgent.

### 4.1 Start the day with a stability briefing

Developers and technical leads often spend time reviewing open issues and deciding which one deserves attention first.

The assistant can retrieve relevant Crashlytics issues and reports, summarize the most important findings, and help prioritize the next investigation.

A useful daily briefing might include new issues, growing event counts, frequently affected users, recurring non-fatal exceptions, and unresolved high-impact problems.

**Example prompt:**

> Review our Crashlytics issues and prepare a daily stability briefing. Identify the five most important issues, explain their observed impact, highlight what appears to be getting worse, and recommend what I should investigate first.

This workflow can reduce the time spent collecting information from multiple reports. The ranking should be based on available evidence, rather than treating every new issue as equally important.

### 4.2 Investigate a crash without leaving the IDE

When a crash occurs, the first challenge is often understanding what happened and which part of the application is involved.

Instead of manually searching the Firebase console, copy-pasting stack traces, and looking for relevant source files, the developer can start the investigation conversationally.

**Example prompt:**

> Investigate this Crashlytics issue. Retrieve the issue details and representative events, explain the stack trace, identify the relevant code paths, and suggest the most likely causes. Separate verified evidence from hypotheses and tell me what to inspect next.

The assistant can retrieve available event context and analyze the source code when repository access is available.

The developer remains responsible for evaluating the explanation and validating the suggested fix.

### 4.3 Resume an unfinished investigation

Debugging does not always end with a solution. A developer might discover a likely cause, test several hypotheses, and then need to move to another task.

Without a written record, the next developer may have to repeat the same investigation.

Crashlytics MCP supports reading and adding notes to issues, making it possible to preserve findings in the issue itself.

**Example prompt:**

> Review the notes and recent context for this issue. Summarize what has already been investigated, which hypotheses were ruled out, what remains unresolved, and the next recommended debugging step.

At the end of a work session, the developer can also ask the assistant to attach a summary of verified findings, relevant code references, test results, and open questions to the issue.

This turns individual investigations into reusable team knowledge.

### 4.4 Use production failures to improve code reviews

A useful extension is to compare the current changes with known production failures.

Suppose a pull request modifies an API response mapper that appears in a recurring Crashlytics exception. An assistant with access to both Crashlytics and the repository can flag the connection before the pull request is merged.

**Example prompt:**

> Compare my current Git diff with open Crashlytics issues. Identify changes touching code paths associated with known failures, explain why each match matters, and recommend targeted regression tests.

This workflow requires repository and Git-diff access in addition to Crashlytics MCP. The Crashlytics integration alone does not automatically correlate every issue with the current diff.

The result should be treated as an additional source of review evidence, not as proof that a code change introduced the original bug.

### 4.5 Identify regressions after a release

A successful deployment does not necessarily mean that the release is stable on real devices.

A new version might introduce an issue that appears only on a specific OS version, device family, or unusual execution path.

After a deployment, developers can use the assistant to review relevant issues and reports, compare available release information, and identify changes that deserve investigation.

**Example prompt:**

> Investigate the stability of our latest release. Identify new or increasing crash and non-fatal patterns, compare affected app versions where the data allows, and explain which issues deserve immediate attention.

Release-specific comparisons become more reliable when the team records consistent version, build, and commit identifiers.

For more extensive historical analysis, Firebase also supports exporting Crashlytics data to BigQuery.

### 4.6 Investigate recurring non-fatal exceptions

Not every application failure ends in a crash.

Repeated non-fatal exceptions can indicate failed parsing, unexpected API responses, broken recovery logic, or other problems that deserve attention even when the application continues running.

**Example prompt:**

> Review recurring non-fatal exceptions and group related failures by stack trace, operation, and available metadata. Identify suspicious patterns, possible common causes, and which issues should be investigated first.

Crashlytics MCP can retrieve relevant events and reports. The assistant can then use source-code context to investigate possible underlying causes.

This workflow is especially useful for understanding whether apparently separate symptoms might involve the same component.

It is important to distinguish expected, deliberately handled exceptions from unexpected failures that may affect user experience.

### 4.7 Turn production issues into better regression tests

Every confirmed production bug is an opportunity to improve the test suite.

For example, a crash involving response parsing might reveal that existing tests do not cover missing fields, unexpected values, or empty responses.

A lifecycle-related failure could suggest tests around navigation, activity recreation, asynchronous callbacks, or state restoration.

**Example prompt:**

> Analyze this Crashlytics issue alongside the relevant source code. Identify the condition that could trigger the failure, propose a regression test for that condition, and tell me which existing tests should be run to validate a fix.

The assistant can propose a test and, when connected to the development environment, help implement it. The proposed test must still be checked to ensure it reproduces the actual failure rather than simply testing an assumed cause.

### 4.8 Create a structured bug report

Production issues often lack the information needed for a developer to begin work efficiently.

An assistant can turn the evidence available in Crashlytics into a structured report containing the issue summary, affected versions, representative stack frames, observed impact, investigation notes, and recommended next steps.

**Example prompt:**

> Prepare a bug report for this Crashlytics issue. Include verified details, affected versions, relevant stack frames, reproduction hypotheses, proposed regression tests, and acceptance criteria. Clearly mark anything that still needs confirmation.

An integration with Jira, GitHub Issues, Linear, or another issue tracker can extend this workflow by creating or updating tickets. Such ticket-management actions require the relevant additional integration.

### 4.9 Find device- and OS-specific failures

Mobile applications run across many combinations of operating systems, hardware, and configurations. A failure may be concentrated in one environment even if it is not obvious from the overall issue count.

The assistant can use available event metadata to look for differences between affected and unaffected environments and propose targeted reproduction scenarios.

**Example prompt:**

> Investigate the affected devices and OS versions for this issue. Identify any meaningful patterns in the available event data and recommend scenarios for QA to reproduce the problem.

This can help developers focus testing instead of repeatedly trying to reproduce the issue on a single development device.

### 4.10 Identify recurring architectural weaknesses

Over time, several Crashlytics issues may point toward the same underlying component, such as a networking layer, shared parser, database wrapper, or navigation utility.

Rather than fixing only the visible symptoms, developers can ask the assistant to compare several issues and identify potential common causes.

**Example prompt:**

> Analyze these unresolved Crashlytics issues together. Identify shared components and recurring error patterns, rank plausible common root causes, and suggest which underlying fix could address multiple issues.

This use case benefits from repository search and sufficient historical context. The assistant should explain the evidence for its grouping because similar symptoms do not always have the same cause.

## 5. The next step: connect Crashlytics to active development tasks

The previous workflows are useful on their own, but a more powerful possibility is to correlate production issues with what developers are working on right now.

Consider a developer implementing a new checkout flow. A production non-fatal exception begins appearing in a response-mapping function that the developer is currently modifying.

The assistant could investigate:

1. Which Crashlytics issues involve the affected function?
2. Which app versions and commits contain that function?
3. Does the current Git diff modify the same code path?
4. Is there an existing task or pull request related to the behavior?
5. Which tests could confirm or rule out the suspected connection?

The result might identify a plausible relationship between a production symptom, a recent code change, and an active task.

**Example prompt:**

> Review the open Crashlytics issues relevant to the feature I am working on. Compare them with my current changes and related tasks. Highlight likely connections, show the evidence behind each match, and recommend the next tests or investigation steps.

This is an extension of Firebase's Crashlytics MCP capabilities, not an automatic feature of the server. To implement it fully, the assistant also needs access to the repository, the task tracker, and consistent identifiers connecting builds to commits or pull requests.

The distinction between correlation and causation is essential. A current task may be related to a crash without being responsible for it.

## 6. Build a lightweight daily routine

A practical implementation does not need to automate the entire debugging lifecycle immediately.

A team could begin with the following routine:

- **Start of day:** Ask the assistant for a stability briefing and select one high-priority issue to investigate.
- **During development:** Use conversational debugging when encountering a production report, and check relevant Crashlytics issues before changing code in a fragile area.
- **Before merging:** Compare the current diff with known failures and identify regression tests worth running.
- **After a release:** Review new and increasing issues and determine whether they require further investigation.
- **End of day:** Add a concise investigation note to any issue that remains unresolved, preserving the findings and next steps for the following session.

These actions use the available MCP capabilities alongside ordinary development practices. They do not require a fully autonomous agent or a complex custom platform.

Once the workflow proves valuable, teams can extend it with repository, issue-tracker, CI/CD, or BigQuery integrations.

## 7. How to get more value from Crashlytics data

AI-assisted debugging is only as useful as the data available to it.

Teams should make sure Crashlytics reports contain meaningful logs, appropriate custom keys, useful exception details, and enough application metadata to distinguish relevant execution paths.

Build pipelines should record consistent app versions, build numbers, and Git commit identifiers where possible. This makes it easier to relate production behavior to the source code that generated a particular build.

It is also important to protect user data. Avoid recording sensitive information such as passwords, access tokens, or unnecessary personal details in logs and custom keys.

Finally, give the assistant the smallest relevant set of issues and events needed for the investigation. Focused evidence usually leads to a more useful debugging conversation than an indiscriminate dump of production data.

## 8. Limitations and considerations

Firebase currently describes its Crashlytics MCP functionality as experimental, so its available tools and workflows may change.

The integration is intended for applications that already use Crashlytics; it does not replace the underlying crash-reporting setup.

It also does not automatically provide continuous monitoring, scheduled briefings, task-tracker synchronization, or a guaranteed mapping between crashes and code changes. Those capabilities require additional integrations or automation when they are needed.

Access permissions and data governance deserve attention as well. The Firebase MCP server uses the credentials available in its configured environment, and the AI development tool may have its own data-handling terms.

For these reasons, a sensible starting point is read-oriented investigation, with developer approval for actions such as modifying issue status, changing code, or deploying a fix.

See the [official Firebase documentation](https://firebase.google.com/docs/crashlytics/ai-assistance-mcp) for current setup details, supported tools, and limitations.

## Conclusion

Firebase Crashlytics MCP creates an opportunity to bring production diagnostics directly into everyday mobile development.

Developers can use it to prioritize issues, investigate crashes, understand non-fatal exceptions, preserve debugging knowledge, prepare bug reports, and identify regression tests. With additional repository and task-tracker integrations, the same workflow can connect production failures with the code and engineering tasks most relevant to them.

The most effective approach is to begin with small, repeatable workflows: a daily stability briefing, conversational investigation, and a post-release review. These provide immediate value while establishing the foundation for deeper integrations.

The larger goal is not to let AI make unverified decisions about production stability. It is to reduce the manual work required to collect evidence, understand the problem, and choose the next engineering action.

**MCP makes Crashlytics more than a dashboard to inspect after something breaks. It can become a practical part of the daily feedback loop between production behavior, code changes, testing, and continuous improvement.**

---

### Links & References

- [Medium Article](https://medium.com/@ashrafatef843/firebase-crashlytics-mcp-bringing-ai-powered-debugging-into-everyday-mobile-development-c9425fc93a08)
- [LinkedIn Post](https://lnkd.in/p/dbWhhQWn)
- [Official Firebase Documentation](https://firebase.google.com/docs/crashlytics/ai-assistance-mcp)
