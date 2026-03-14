---
title: "The Prompting Checklist I Wish I Knew Earlier"
date: 2026-03-10
tags: ["prompting", "engineering", "LLM"]
summary: "10 guidelines applied across 8 real case studies — with before/after examples from 700+ LLM interactions."
ShowToc: true
---

## Introduction

I spend most of my day talking to LLMs. Over the past month, I've sent over 700 messages across 60+ sessions -- prompts for code review, debugging, test generation, CI/CD pipelines, and feature work. My primary model (Claude) is less of a tool and more of an autonomous partner: I set direction, course-correct when needed, and let it run.

After reviewing dozens of prompts -- my own and from engineers across teams -- patterns emerged. Some prompts consistently produced useful output on the first try. Others required multiple rounds of follow-up, burning tokens and time on clarification that should have been in the original ask.

This post distills those patterns into 10 guidelines applied across 8 real case studies, drawing on prompt engineering best practices recommended by Anthropic and OpenAI, and Wei et al.'s research on chain-of-thought prompting. Every before/after example comes from actual engineering work.

<div class="disclaimer-block">
These guidelines are opinionated. They reflect what works for software engineering prompts in my experience. Your domain, model, and use case may differ. Treat these as a starting checklist, not gospel.
</div>

## Guidelines

### 1. Output defined

Know what a good response looks like before you prompt -- format, fields, length, and structure. If you can't describe what success looks like, the model can't achieve it. A prompt without output criteria is a coin flip.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Summarize this PR</td>
<td>Summarize this PR in 3 bullet points: what changed, why, and what to test</td>
</tr></tbody>
</table>
</div>

### 2. Concrete

Replace vague verbs like "look into," "analyze," or "explore" with specific deliverables. The litmus test: if a colleague would ask you a follow-up question, the model will guess instead.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Look into the login bug</td>
<td>Find why login returns 401 for OAuth users when the token has not expired</td>
</tr></tbody>
</table>
</div>

### 3. Field ordering

In structured output, put reasoning fields before simple booleans or enums. Each generated token influences the next -- when the model reasons first, its final yes/no decisions are more accurate.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td><code>{ "is_spam": bool, "reason": str }</code></td>
<td><code>{ "reason": str, "is_spam": bool }</code></td>
</tr></tbody>
</table>
</div>

### 4. Parameters delimited

Use unusual delimiters (`<<<PARAM>>>`) or labeled anchors for input boundaries. Quotes and backticks blend with natural text, making it ambiguous where user input ends and instruction begins. The more distinct the boundary, the less the model confuses data with instructions.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Translate the user's message "hello world" to French</td>
<td>Translate to French:<br><code>&lt;&lt;&lt;hello world&gt;&gt;&gt;</code></td>
</tr></tbody>
</table>
</div>

### 5. Positive framing

Lead with what to do, not what to avoid. A long list of "do not" constraints forces the model to infer intent by elimination. State the desired behavior directly and reserve negation for hard safety boundaries.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Don't use external libraries. Don't write verbose code.</td>
<td>Use only the standard library. Keep functions under 20 lines.</td>
</tr></tbody>
</table>
</div>

### 6. Steps not paragraphs

One complex ask consistently underperforms the same task broken into numbered steps. Research has shown that chain-of-thought prompting significantly improves accuracy on reasoning tasks. Numbered steps also make it easier to spot which step went wrong when output is off.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Review this code for bugs, security issues, and style, then fix them</td>
<td>1) List bugs<br>2) List security issues<br>3) List style violations<br>4) Fix each</td>
</tr></tbody>
</table>
</div>

### 7. Right technique

Use a schema when you need structured output (JSON, tables, fixed formats) and few-shot examples when you need judgment calls (tone, severity, relevance). Mixing them up -- giving examples for a format task or a schema for a judgment task -- degrades both.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid (example for a schema task)</th><th>Better (schema for a schema task)</th></tr></thead>
<tbody><tr>
<td>Output JSON like:<br><code>{ "name": "acme", "score": 92 }</code></td>
<td>Output JSON matching:<br><code>{ name: string, score: int, pass: bool }</code></td>
</tr></tbody>
</table>
</div>

### 8. Testable wording

Even paraphrasing the same instruction changes results -- "summarize" vs. "extract key points" can produce meaningfully different output. Treat prompt text like code: test variants against real data before committing to one.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Write a good summary</td>
<td>Write a summary under 50 words covering who, what, and why</td>
</tr></tbody>
</table>
</div>

### 9. Isolated context

Scope each prompt to a single task. When you mix unrelated requests in one conversation, earlier context bleeds into later responses, degrading quality and wasting tokens on irrelevant information.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>Also while you're at it, update the README and check CI</td>
<td>Update the README to reflect the new API endpoint<br>(separate prompt for CI)</td>
</tr></tbody>
</table>
</div>

### 10. No subjective asks

Ask for deliverables, not feelings. An LLM has no confidence, intuition, or opinion -- subjective questions produce fabricated hedging. "What blocks implementation?" gives you actionable output; "how confident are you?" gives you theater.

<div class="guideline-card">
<table class="avoid-better-table">
<thead><tr><th>Avoid</th><th>Better</th></tr></thead>
<tbody><tr>
<td>How confident are you in this fix?</td>
<td>What edge cases does this fix not handle?</td>
</tr></tbody>
</table>
</div>

## Key Findings from 8 Real Prompts

I applied the checklist to 8 prompts used in day-to-day engineering work. The three most commonly violated guidelines:

| Guideline | Failure Rate |
|-----------|:---:|
| **Steps not paragraphs** | 7/8 |
| **Output defined** | 6/8 |
| **Concrete** | 6/8 |

**Takeaway:** Field ordering and parameters delimited are N/A for most conversational prompts -- they matter for production LLM prompts (system prompts, API calls). The biggest wins come from **steps not paragraphs**, **output defined**, and **concrete**.

## Case Studies

Sorted from worst initial score to best. Each prompt is scored out of 10 -- one point per guideline satisfied. These scores were generated by [prompt-polish](https://github.com/kayhwee/llm-toolkit/tree/main/skills/prompt-polish), a tool I built that automatically checks each guideline and rewrites failures. The before/after pairs below are its output.

### 1. Customer feature disable <span class="score-badge score-low">2 &rarr; 8</span>

<div class="case-study">

Four product acronyms without definitions, three vague questions, and four distinct features crammed into one prompt. Without definitions for any of the acronyms, "allowed" is ambiguous, and there's no guidance on how to structure a response across four separate processes.

```
// Before
I have some customers who either want to disable SSO, opt out of
MFA, disable Webhooks and disable Audit Logs.

What are they allowed? What process do they follow? Any docs?

// After
For each of these features -- SSO, MFA, Webhooks, Audit Logs:
1) Can customers disable or opt out? (yes/no, with any conditions)
2) What is the process they follow?
3) Link to the relevant documentation, if it exists.

Format as a table: Feature | Can Disable? | Process | Docs Link
```

**What changed:**
- **[Output defined]** Defined output format (table with 4 columns)
- **[Concrete]** Replaced "what are they allowed?" with specific yes/no question
- **[Steps not paragraphs]** Decomposed into 3 numbered steps per feature
- **[Isolated context]** Structured as per-feature evaluation instead of mixed blob

</div>

### 2. Ticket confidence check <span class="score-badge score-low">3 &rarr; 8</span>

<div class="case-study">

This prompt asks a subjective question ("how confident are you?") that has no measurable answer for an LLM. "Look into" is vague -- it could mean read the ticket, analyze the code, or draft a solution. There are no steps or success criteria — no definition of done.

```
// Before
Look into PROJ-1234, how confident are you in automating this?
What additional context do you need?

// After
Look into PROJ-1234. For each file that needs to change, list:
1) File path
2) What changes and why
3) Risks (breaking changes, edge cases, missing tests)
4) Blockers preventing implementation

Format as markdown with a heading per file.
```

**What changed:**
- **[Output defined]** Defined output structure with per-file breakdown
- **[Concrete]** Replaced "look into" with specific deliverables
- **[Steps not paragraphs]** Decomposed into 4 numbered steps
- **[No subjective asks]** Replaced "how confident" with "blockers preventing implementation"

</div>

### 3. OAuth test approach <span class="score-badge score-low">3 &rarr; 8</span>

<div class="case-study">

"What do you think how we can approach this?" is a subjective ask -- there are no criteria for what a good approach looks like, no output format, and no fallback instruction for when examples don't exist. Without guardrails, expect a wall of speculation.

```
// Before
With respect to api_integration_tests, is there any existing examples to
verify as a oauth authenticated support end-users? If not what do
you think how we can approach this problem?

// After
I need to write OAuth-authenticated end-user tests in
api_integration_tests.
1) Search the repo for existing OAuth end-user test examples.
2) If examples exist, list file paths and summarize the pattern.
3) If no examples exist, propose a test approach:
   - How to obtain an OAuth token for a support end-user
   - Which endpoints to test
   - How to structure the test file
```

**What changed:**
- **[Output defined]** Defined output for both branches (examples found vs. not found)
- **[Concrete]** Replaced "approach this problem" with specific deliverables
- **[Steps not paragraphs]** Decomposed into 3 numbered steps with sub-items
- **[No subjective asks]** Replaced "what do you think" with concrete investigation steps

</div>

### 4. Code review without criteria <span class="score-badge score-low">4 &rarr; 8</span>

<div class="case-study">

"Review my changes" without criteria is like asking "what do you think?" -- the model doesn't know whether to focus on correctness, security, style, or all of the above. There's no output format, so comments arrive as an unstructured wall of text.

```
// Before
As a reviewer, review my changes in acme/web-console

// After
Review my changes in acme/web-console on the current branch.
Focus on:
1) Correctness
2) Security
3) Maintainability
4) Test coverage
For each issue, cite the file:line and severity (critical/warning/nit).
```

**What changed:**
- **[Output defined]** Defined output format (file:line + severity)
- **[Concrete]** Scoped to current branch
- **[Steps not paragraphs]** Decomposed into 4 focus areas
- **[Right technique]** Used structure (not examples) for a format-oriented task

</div>

### 5. PR review by URL <span class="score-badge score-mid">5 &rarr; 8</span>

<div class="case-study">

The PR URL is unambiguous (parameters delimited -- passes), but "review" alone leaves too many degrees of freedom. The model doesn't know what to look for, how to structure findings, or at what severity level to flag issues.

```
// Before
Review https://github.com/acme/platform/pull/1234

// After
Review the PR at https://github.com/acme/platform/pull/1234.
For each file changed:
1) Check correctness against the stated intent
2) Identify edge cases and boundary conditions
3) Flag security concerns
4) Rate severity: critical/warning/nit
Output as a list of comments with file:line references.
```

**What changed:**
- **[Output defined]** Defined output format (comments with file:line and severity)
- **[Concrete]** "Review" replaced with 4 specific focus areas
- **[Steps not paragraphs]** Decomposed into 4 steps per file
- **[Right technique]** Used structure for format-oriented task

</div>

### 6. Test coverage and feature flags <span class="score-badge score-mid">6 &rarr; 8</span>

<div class="case-study">

The intent is clear and the scope is defined, but "ensure tests pass" and "sufficient coverage" are undefined. How should failures be reported? What counts as sufficient? Without steps, the model may run tests but skip the feature flag verification entirely.

```
// Before
Ensure all relevant tests pass and feature flag gating is properly configured.
Verify test coverage is sufficient.

// After
1) Run the full test suite and report any failures with file:line.
2) Verify feature flag gating: check flag name, default state,
   and rollout config.
3) List any untested code paths or missing edge cases.
4) Confirm CI pipeline passes before marking complete.
```

**What changed:**
- **[Output defined]** Defined what "pass" means (report failures with file:line)
- **[Concrete]** Replaced "sufficient" with "list untested code paths"
- **[Steps not paragraphs]** Decomposed into 4 ordered steps
- **[Testable wording]** Made each step independently testable

</div>

### 7. Address PR comments <span class="score-badge score-high">7 &rarr; 8</span>

<div class="case-study">

This prompt already scores well -- the PR URL identifies the task clearly and "address the comments" is reasonably concrete. The gaps are subtle: no guidance on ambiguous comments (skip or push back?), and no explicit step ordering.

```
// Before
Address the PR comments in
https://github.com/acme/platform/pull/1234.

// After
Address the PR comments in
https://github.com/acme/platform/pull/1234.
1) Read all unresolved comments.
2) Implement requested changes.
3) If a comment is unclear, explain why rather than skipping.
4) Run tests after changes.
```

**What changed:**
- **[Output defined]** Step 3 handles ambiguous comments explicitly
- **[Positive framing]** Positive instruction ("explain why") instead of implicit skip
- **[Steps not paragraphs]** Decomposed into 4 ordered steps

</div>

### 8. Localization bug fix <span class="score-badge score-high">8 &rarr; 8</span>

<div class="case-study">

This prompt scores perfectly across all applicable guidelines. It provides numbered reproduction steps, a specific URL and locale, a clear issue description with expected behavior, a reference to the correct solution (CLDR), and a concrete deliverable ("create a PR"). This is what a well-structured prompt looks like.

```
// This prompt needed no rewriting.

In Split View, the ticket time info is in English format
and not localized.

Steps to Reproduce:
1) Login to staging.example.com/dashboard
2) Change locale to French in user profile
3) Refresh and go to Home page
4) Observe the issue

Issue: Time info shows AM/PM (not used in French).
       Affects multiple locales.

Expected: Localize the time info using CLDR (see this doc).

Create a PR to resolve this issue.
```

**Why it works:**
- **[Output defined]** PASS -- "create a PR" is a concrete deliverable
- **[Concrete]** PASS -- specific URL, locale, and expected behavior
- **[Steps not paragraphs]** PASS -- numbered reproduction steps
- **[Testable wording]** PASS -- "is time localized in French?" is testable

</div>

## Checklist

Before sending a prompt, run through this:

| # | Guideline | Ask yourself |
|---|-----------|-------------|
| 1 | Output defined | Can I describe what a good response looks like? |
| 2 | Concrete | Would a colleague know what to do from this alone? |
| 3 | Field ordering | Do reasoning fields come before booleans in structured output? |
| 4 | Parameters delimited | Are input boundaries unambiguous? |
| 5 | Positive framing | Am I leading with what to do? |
| 6 | Steps not paragraphs | Is the task broken into numbered steps? |
| 7 | Right technique | Schema for format, examples for judgment? |
| 8 | Testable wording | Could I test this wording against a paraphrase? |
| 9 | Isolated context | Is this scoped to a single task? |
| 10 | No subjective asks | Am I asking for deliverables, not feelings? |

## Automate it

The case studies in this post were generated by running prompts through [prompt-polish](https://github.com/kayhwee/llm-toolkit/tree/main/skills/prompt-polish), a Claude Code skill that checks each guideline automatically, rewrites what fails, and cites what changed. If you use Claude Code, you can install it and skip the manual checklist entirely.

## Sources

- [Anthropic: Be Clear and Direct](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct)
- [Anthropic: Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Anthropic: Interactive Tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial)
- [OpenAI: Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [OpenAI Cookbook: Reliability](https://cookbook.openai.com/articles/techniques_to_improve_reliability)
- [promptingguide.ai](https://www.promptingguide.ai/)
- [Wei et al. 2022](https://arxiv.org/abs/2201.11903) — Chain-of-thought prompting
