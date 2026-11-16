+++
author = "marclederoff"
title = "How Hivebrite Makes Quality Visible at Scale (Part 2): When AI Makes Tests Cheap, Intent Becomes the Bottleneck"
date = "2026-11-16"
description = "A year of AI-assisted Quality Engineering at Hivebrite: what worked, what broke, and why we now write down the product's promises before we write the tests."
tags = [
    "QA", "automation", "AI", "engineering practices", "spec-driven development"
]
categories = [
    "tech"
]
canonicalUrl = "https://hivebrite.io/blog/how-hivebrite-makes-quality-visible-at-scale-2/"
thumbnail = "/img/thumbs/hivebrite.webp"
#featureImage = "hivebrite.webp"
#featureImageAlt = "Hivebrite's engineering team!"
+++

*A year of AI-assisted Quality Engineering at Hivebrite: what worked, what broke, and why we now write down the product’s promises before we write the tests.*

## TL;DR

AI made code and tests much cheaper to produce at Hivebrite. In the last quarter alone, we added **1,249 new hermetic Playwright tests**. A curated suite of roughly **190 browser tests** runs systematically through our CI/CD Quality Gates, while the broader hermetic pool is available when a change makes it relevant.

But more automation did not automatically mean more confidence. We even saw an AI-generated test pass while validating the wrong product behaviour.

Our response has been to move towards **Spec-Driven Development**: define the product promise first, connect it to evidence through Spec Gate, and make that intent accessible to Product, Support, developers, QA and AI agents.

The goal is no longer simply to generate more tests. It is to know **what we promised, what proves it, and which evidence needs to run when the product changes.**

## From visible Quality to executable Quality

In [Part 1](/blog/how-hivebrite-makes-quality-visible-at-scale-1/), we described how Hivebrite reorganised Quality around shared ownership, early alignment, risk-based decisions, visible automation and Quality Gates engineers can trust.

We ended with an open question: as teams adopt AI-assisted workflows and ship faster, how does Quality Engineering keep up without becoming the bottleneck?

Over the past year, we have been answering that question in practice.

We introduced AI assistants into QA workflows, encoded testing conventions into prompts and rules, moved from Cursor to Claude Code, and made reusable engineering capabilities available to both QA and developers.

And it worked.

Creating automation became significantly easier.

But that success exposed a different problem. When code and tests are both cheap to generate, producing automation is no longer the hard part.

The hard part is knowing whether we are proving the right thing.

## The first wave: AI as a QA copilot

Our first experiments focused on augmenting existing QA workflows.

Using Dust, our internal AI-assistant platform, we built assistants for common QA activities: analysing bug data and tickets, searching the test knowledge stored in Qase (our test management tool), and drafting new test cases.

The goal was simple: spend less time gathering information and more time making decisions.

We then applied the same idea closer to automation.

With Cursor, we wrote down our browser-test conventions (we use Playwright) as rules the AI could follow: how to structure a test, which shared helpers to reuse and which patterns to avoid.

Instead of starting every automated test from an empty file, a developer or QA Analyst could describe the scenario and let the agent draft most of the test within our existing framework.

The model was still fairly traditional:

**human provides context → AI creates an artefact → human reviews it**

AI made individual QA tasks faster, but the broader operating model had not changed yet.

## The second wave: QA capabilities move into Engineering

Moving to Claude Code changed the model itself.

Instead of maintaining isolated prompts, we started packaging repeatable workflows as **skills**: reusable instruction sets an AI agent can invoke to perform a task according to our engineering conventions.

A skill can explain how to write a Playwright test in our framework, where our shared helpers live, how to use Qase, how repositories are organised, and what our Quality expectations are.

That matters because QA know-how no longer has to live only inside the QA team. It can travel with the engineering workflow.

Our QA Analysts became more comfortable not only using these capabilities, but also creating and improving them. At the same time, developers started contributing more directly to automation.

Our goal has never been to turn every developer into a QA Automation Engineer. It is to remove unnecessary boundaries around Quality.

If a developer introduces a new happy path, they should be able to provide the automated evidence for it without waiting for QA to translate the implementation into a separate test later.

QA can then focus more of its effort where its perspective adds the most value: risk, ambiguity, edge cases, exploratory testing, accessibility and the gaps nobody else has noticed yet.

None of this works without what platform teams often call a **paved road**: shared foundations that make the right way the easy way.

Shared helpers, stable test architecture, review conventions and reliable CI matter even more when many more people — and now AI agents — can create tests.

We have applied the same principle to code review. A Claude-based pull request review bot follows a version-controlled `review.md` rubric, classifying feedback as **Blocker**, **Consider** or **Nit**.

Making that rubric explicit matters. An automated reviewer that treats every observation as critical quickly becomes noise. Because the criteria live in version control, we can evolve them like any other engineering standard: transparently, collaboratively and with a shared view of what should actually block delivery.

## When more automation becomes a Quality problem

For years, the software testing industry has talked about the test pyramid: many fast tests at the bottom, fewer integration tests in the middle, and a relatively small number of expensive end-to-end tests at the top.

AI can distort that model surprisingly quickly.

If generating a browser test takes minutes instead of hours, the temptation is to generate another one, and another.

Over the last quarter, this capability helped us add **1,249 hermetic Playwright tests**, which forced us to decide where each one belongs.

Hermetic tests are browser tests that run against an isolated copy of the application with external services controlled or faked. They give us realistic workflow coverage without depending on a full shared environment.

The risk is obvious: if every new automated test is simply added to the default CI path, the test pyramid can start looking more like an hourglass — lots of tests close to the code, lots of browser automation at the top, and insufficient thought about where each behaviour really needs to be proven.

**The cost has not disappeared. We have simply moved it.**

It appears later as execution time, maintenance, test data complexity, flaky signals and slower feedback.

That led us to a principle we now use when choosing the level of automation:

**Every behaviour should be proven at the cheapest level that still proves the risk.**

It also reinforced a principle already present in our first article:

**automation should follow risk, not volume.**

Roughly **190 browser tests** currently form the curated suite that runs systematically through our CI/CD Quality Gates.

Those tests and the broader hermetic suite use the same underlying automation technology. The difference is **when and why we execute them**.

Our Quality Gates run at different stages of delivery. The tests that continuously protect those gates are deliberately selected according to the criticality of their functional area or sub-area.

Tests covering our most critical functional areas can be promoted into our `@core` suite, which runs on our QA environment just before each daily deployment to confirm we are safe to deploy.

The rest of the hermetic tests remain available as a much larger pool of evidence.

Putting everything into the default CI path would simply move the bottleneck: tests would become cheap to create, while every push became expensive to validate.

But execution cost was not the only problem created by easier automation.

A more fundamental issue appeared when we looked at how AI-generated tests decide what the expected behaviour should be.

## When a green test proves the bug

One recent authentication example made the problem particularly clear.

We asked an agent to generate a hermetic Playwright test for a logout flow.

The agent inspected the existing implementation, understood how logout currently behaved, and wrote a coherent test around it.

The test passed.

The problem was that the implementation itself was wrong: the flow reported a successful logout while the user was still effectively signed in to Hivebrite.

Because the agent learned what to expect from the code, it reproduced the same incorrect assumption in the test.

We had produced automated evidence that proved the bug.

This exposed an important limitation when both implementation and test derive their truth from the same source.

If the code tells the agent what the product is supposed to do, the generated test can end up proving only that:

**the code behaves like the code.**

That is very different from proving that the product behaves as intended.

The direction we want is therefore:

**specification → scenario → implementation-aware test**

rather than:

**implementation → generated test → implementation validated**

What the product should do must exist independently from both the code and the test.

That realisation is why Spec-Driven Development has become central to our work.

## From documentation to product promises

Mature products usually have plenty of documentation, and Hivebrite is no exception.

We have technical documentation, tickets, Qase cases, customer-facing Knowledge Base articles, code comments, ADRs (Architecture Decision Records explaining technical choices), and years of implementation history.

But documentation and specifications are not the same thing.

Documentation can explain how something works.

An ADR can explain why we made a technical decision.

A specification should answer a simpler question:

**What does the product promise?**

Our first AI-assisted attempts at extracting those promises showed the difference very clearly.

In one authentication area, an initial pass produced **69 candidate rules**.

They were detailed, but many mirrored implementation branches instead of expressing durable product behaviour.

After human review against our rule-writing principles, those 69 candidates became **14 product-level rules**.

AI is excellent at producing volume.

The harder work is deciding which rules deserve to become durable product contracts.

## Our Rule-Writing Manifesto

To make those decisions more consistent, we created a small Rule-Writing Manifesto.

Our six principles, in priority order:

1. **The intended target over what the code does today.** A specification describes what the product should do, not what the current implementation happens to do.
2. **Observable behaviour over implementation details.** Rules describe what a user, client or external system can observe.
3. **One outcome per rule.** No “and then also”: one rule describes one result that can be checked independently.
4. **One observer per rule.** If a user, backend and log each observe something different, write separate rules.
5. **Generic rules over one rule per test case.** The rule captures stable behaviour; individual variants belong in tests.
6. **The contract over where it is tested.** A rule should remain valid whether its proof lives in a unit test, an integration test, Playwright, Qase or somewhere else.

We use a lightweight syntax inspired by EARS (Easy Approach to Requirements Syntax), where one sentence describes one observable behaviour.

For example:

> IF a request carries an expired token, THEN THE platform SHALL refuse it without calling the protected service.

The syntax matters less than the content: the sentence describes something an outside observer can verify.

It does not mention the controller, React component, Ruby class, Go package or database query that happens to implement the behaviour today.

That makes the rule much more durable.

One principle has become especially useful:

**Write the target.**

If the current product does not yet behave the way we have decided it should, we do not weaken the specification to match the implementation.

The rule describes the desired behaviour and remains explicitly pending until implementation and proof catch up.

We also avoid turning defects into specifications.

**A failure is only a rule when the input is wrong. A failure on valid input is a bug.**

## Spec Gate: connecting promises to evidence

Writing down product promises only solves part of the problem.

A specification sitting in a Markdown file and disconnected from delivery will eventually become another stale document.

This is where **Spec Gate**, built by our Developer Efficiency & Architecture team, comes in.

Spec Gate connects product rules to the evidence that proves them. Each rule receives a stable identifier, and tests cite the rules they prove.

The direction matters:

**the rule does not point to the test; the evidence points back to the rule.**

That keeps the behavioural contract independent from how we choose to validate it.

A rule might be proven by a request test today and by a browser integration test tomorrow without changing the product promise itself.

Rules that require automated proof are marked accordingly. Today, if a required rule has no accepted evidence, **the Spec Gate CI check fails unless that rule is explicitly declared as pending**.

Missing proof is no longer something we have to discover manually after the fact. It becomes part of the delivery signal.

Delivery, however, is rarely binary.

A behaviour may be decided but not fully implemented, hidden behind a feature flag, blocked by a known bug, or best tested in another repository.

Spec Gate makes those situations explicit:

- **Pending** — the behaviour is decided but not yet fully proven, with a date and reason.
- **Proven** — accepted evidence exists.
- **Left to another project** — another repository is better placed to prove the promise.

This allows ownership and proof to remain distributed without losing traceability.

## Making product intent queryable

Spec Gate is not designed only for CI.

Its most valuable property is that different people can use the same behavioural model to answer different questions:

- **Product:** What behaviour did we actually agree for this feature?
- **Developers:** Which product promises could this change affect?
- **QA:** Which required promises are still missing evidence?

For Support, the question is even simpler:

**“Is the product supposed to do that?”**

If the rule exists, the agreed behaviour is visible.

If no rule exists, that can also be valuable.

The behaviour might simply be an internal detail that does not deserve a product contract.

But if it matters to customers and nobody agrees on the expected outcome, the gap is not primarily a testing gap.

It is a product decision that was never made explicit.

### A Spec Gate MCP proof of concept

We are also exploring this through a **Spec Gate MCP proof of concept**.

MCP (Model Context Protocol) is a standard way for AI assistants to query tools and structured data.

Because the Spec Gate model is stored as structured YAML, the same source can answer each role at the right level of detail.

For coding agents, this opens a new possibility:

**before changing code, the agent can first ask what the product promises in that area.**

Instead of discovering intent from the implementation, the agent starts with product intent in its context, before the implementation can influence the answer.

Once product intent becomes structured and queryable, specifications stop being documents people must remember to read.

They become part of the engineering system.

## Deciding behaviour before code decides for us

This reconnects directly with practices we described in Part 1, particularly **Three Amigos**: Product, QA and a developer reviewing behaviour together before or during implementation.

For sufficiently complex work, a short **Example Mapping** session (a structured exercise capturing rules, examples and unanswered questions) leaves the group with three useful outputs:

- **Rules** — what the product should promise.
- **Examples** — concrete cases showing what those rules mean.
- **Questions** — things nobody can answer yet.

The last category can be the most valuable.

Without that discussion, unanswered questions have a tendency to become implementation decisions.

A developer makes a reasonable choice because the ticket is ambiguous. A test is written around that behaviour. Months later, everybody assumes the product was intentionally designed that way.

Making questions visible early helps prevent accidental product design through implementation.

### Bugs become feedback on the contract

The same model changes how we approach bugs.

When a bug arrives and a relevant rule already exists, the fix has four steps:

1. Find the promise that was broken.
2. Reproduce the failure with evidence.
3. Fix the implementation.
4. Restore the proof.

If no rule describes the expected behaviour, we first ask whether one should.

Not every technical defect needs a product rule. Nobody needs a product specification explaining how an internal cache is populated.

But when the behaviour is visible to customers — especially when Product, Support and Engineering disagree about what should happen — the bug has exposed missing product intent.

The fix should not only repair the code.

It should close that gap.

### A test of our tests

Specifications also give us a useful test of our tests.

When a developer restructures code without changing what users observe, the specification should remain unchanged, and ideally so should the behavioural evidence.

If a test has to be rewritten every time internal code moves, it is worth asking whether it was proving the contract or the implementation.

## From “run everything” to “prove what changed”

Risk-based Quality Gates answer one question:

**How critical is this behaviour?**

**Crystal Ball**, our internal focused-testing approach, answers another:

**What could this particular change affect?**

We use it today to identify the functional areas or sub-areas touched by a change and trigger the relevant hermetic tests on demand.

This gives us deeper regression confidence without forcing the whole hermetic suite into every CI run.

Spec-driven traceability can take that model further.

Today, focused testing starts from **the code that changed**.

Longer term, we want to connect:

**changed code → impacted product rules → evidence that proves those rules**

At that point, test selection becomes more than a performance optimisation.

It becomes explainable.

Instead of saying:

> “We skipped most of the suite because our selection algorithm decided it was safe.”

we want to be able to say:

> “This change can affect these product behaviours, and this is the evidence we selected to prove they still work.”

AI made tests cheaper to create.

**The next challenge is deciding which evidence needs to run for a given change.**

## Connecting the Quality knowledge graph

Longer term, we want each product rule to become the thread connecting knowledge currently scattered across different tools:

- **Code** — where the behaviour is implemented.
- **Automated tests** — whether the behaviour still works.
- **Qase cases** — how we validate or explore it manually.
- **Knowledge Base articles** — how the behaviour is explained to customers.
- **Production monitoring** — whether critical journeys still behave correctly after deployment.
- **Support incidents** — where reality challenged our assumptions.

Once those assets share the same behavioural language, we can ask better questions.

Which critical promises have no automated proof? Which Knowledge Base articles describe behaviour that changed? Which incidents affected areas we believed were covered?

That is a different view of test coverage.

It is not:

**How many tests do we have?**

It is:

**How much confidence do we have in the promises that matter?**

## What we have learned so far

Several parts of this work are still evolving, but a few lessons are already clear.

**1. AI is great at volume, and volume is rarely the bottleneck.**

Generating code, tests or candidate rules is becoming cheap. The hard work remains deciding which behaviours matter, removing duplication, spotting ambiguity and choosing the right level of proof.

The reduction from **69 candidate rules to 14 product-level rules** in one area is a good example: generation was easy; deciding what deserved to survive implementation changes was the difficult part.

**2. Specifications pay off before the first test runs.**

Reviewing behavioural rules has already exposed contradictions between documentation and implementation, assumptions nobody was sure about anymore, and behaviours that required an explicit Product decision.

For example, in one authentication area we uncovered an unresolved question around token lifetime: should a refresh token remain valid for six months, or expire after 30 days?

The code could implement either answer.

The valuable part was discovering that the product decision itself needed to be made explicit.

**3. More automation does not mean more confidence.**

A green test that checks the wrong expectation can be worse than having no test at all because it creates false confidence.

The logout example taught us that automation needs an independent source of intent.

**4. Quality work is moving upstream.**

QA still tests software.

But Quality Engineering also helps make sure what we build is understood, observable, deliberate and provable before implementation hardens assumptions into behaviour.

## What comes next: deterministic proof and agentic exploration

Critical promises should still be proven with **deterministic tests**: scripted checks that give us an exact, repeatable answer.

If a rule says that an unauthorised user must not gain access to something, we want a precise assertion.

But scripted tests only check what we already thought to ask.

**Agentic exploration** offers a different layer: an AI agent can use the product more like a user, navigate unexpected paths and challenge behaviours our scripted suite never encoded.

**Deterministic automation protects the contracts we know. Agentic exploration challenges what we may have forgotten.**

Hybrid approaches are starting to emerge where deterministic steps and agent-driven exploration can coexist inside the same verification workflow.

We have not yet validated where that model provides enough reliability and value for our stack, so we treat it as an area for experimentation rather than a platform decision.

The question we care about is:

**Where can agentic testing give us incremental confidence that deterministic automation cannot provide economically?**

## Closing thoughts

In Part 1, we wrote that Quality is not a phase, a checklist or a role.

A year of rapid AI adoption has reinforced that view.

AI makes implementation faster, tests easier to create, and previously specialised workflows available to many more people.

None of that removes the need for Quality Engineering.

It changes where its value sits.

As implementation becomes cheaper, **intent becomes more valuable**.

We need to preserve the connection between what we decided, what we promised, what we built, what proves it, and what customers actually experience.

That requires tooling, but it also requires governance.

It requires Product, Engineering, QA and Support to share a language for behaviour.

And it requires us to resist one of the most tempting failure modes of AI-assisted engineering: generating code and tests from the same assumptions and mistaking agreement between them for correctness.

Our goal is therefore no longer simply to automate more.

It is to keep three questions answerable:

**What did we promise?**

**What proves it?**

**And if it changes tomorrow, will we know?**
