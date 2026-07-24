# Chapter 11 — Testing GenAI services

Grounding for **Area 6**. Source: *Building Generative AI Services with FastAPI*, Ch 11.

## When testing is warranted (calibration)

Testing trades time and effort for confidence. The book is explicit that the right amount depends on stakes:
- A prototype in fast iteration needs little or no automated testing.
- A **minimum sellable product**, or anything that handles **sensitive data** or **processes payments**, must be tested.
- Also test when there are multiple contributors over time, when external dependencies change, as the number of components grows, or when bugs start appearing.

Grade Area 6 against this. Thin tests on an experiment are not a fail; thin tests on a production or sensitive service are.

## Standard testing concepts

Test types, increasing in size and cost:
- **Unit:** isolated components/functions, atomic, no external systems.
- **Integration:** interaction between components/subsystems; validates data flows and interface contracts; scoped to a subset, often pairwise.
- **E2E:** whole system, real usage start to finish; highest confidence, hardest and most brittle to maintain.

Static checks first (for example `mypy`) catch type/syntax errors, style issues, misuse, security vulnerabilities, dead code. As you move static -> unit -> integration -> E2E, tests get more valuable but slower, costlier, and more brittle. E2E nondeterminism comes from lack of isolation, async behaviour, remote services, and resource leaks.

Structure tests with **Given-When-Then** (pytest's arrange-act-assert-cleanup): set up fixtures, exercise the system under test, assert outputs, optional cleanup.

**Do not test implementation details.** Test observable behaviour (black-box). Telltale signs of testing internals: tests break on a refactor that changes no behaviour (false positives), or pass when you introduce a breaking change (false negatives). Test what `count_tokens(text)` returns, not how it builds the count.

Biggest challenge: deciding what to test and what to mock, fake, or keep real. Plan in advance, find breaking points, imagine the user's steps, turn them into tests.

## Why GenAI testing is different

- **Output variability (flakiness):** same input, different output, because models sample probabilistically (temperature affects variance). Deterministic exact-match tests are too flaky for CI/CD. Approach statistically: sample a **representative, legitimate distribution of inputs** and verify output quality; or score variable outputs with a discriminator model against a tolerance/threshold.
- **Performance and cost:** multi-model/statistical tests are slow and burn tokens. Mitigate with mocking and patching, dependency injection, statistical hypothesis tests, reduced model-test frequency, and small fine-tuned discriminators.
- **Regression and model drift:** the behaviour of the "same" hosted model changes over time (concept drift, data drift). Plan regression testing and monitoring, especially with external providers; address drift at the app layer (validation, RAG) or model layer (retrain/fine-tune).
- **Bias:** gender/racial/age bias, especially serious for LLM-as-judge tools (marking, interview assessment). Detect with model self-checks and AI discriminators.
- **Adversarial attacks:** jailbreak/prompt injection, token manipulation, sensitive-info disclosure, data poisoning, model theft, DoS, excessive agency. Internet-facing services need safeguarding layers (see OWASP Top 10 for LLM apps). Adversarial tests should also verify the authn/authz guards. (Implementation detail in Ch 9 / Area 5.)
- **Unbounded coverage:** the input space is effectively infinite, so unit tests cannot reach full coverage. Use behavioural testing on output properties instead.

## Behavioural / property-based testing (the core technique)

Treat the model as a black box and check output **properties** over a representative range of inputs, not exact strings. Aim for statistical confidence, not full coverage. Example properties: sentiment, response length, readability, factual "I don't know" when unanswerable, coherence, relevance, toxicity, correctness, faithfulness to policy.

The landmark paper breaks behavioural tests into three categories (use these names):

- **Minimum Functionality Tests (MFTs):** basic correct behaviour on simple, well-defined inputs, including failure modes. E.g. grammar, well-known facts, zero toxicity, rejecting clearly inappropriate inputs, readable/professional output. (Example in the book: assert a `textstat` Flesch reading-ease score falls in an expected band.)
- **Invariance Tests (ITs):** output stays consistent under irrelevant input changes (case, whitespace/special characters, typos, synonyms, number formats, reordering chunks). Tests robustness and sensitivity.
- **Directional Expectation Tests (DETs):** output moves in the right direction as input changes (negative-sentiment prompts addressed, detailed questions answered with more specificity, a more complex prompt yielding a longer/more complex response).

**Auto-evaluation tests:** use a discriminator/evaluator model (an LLM or classifier) to score outputs on hallucination, toxicity, correctness, relevancy, etc., returning structured JSON for parsing, asserted against a threshold. Powerful but adds API calls and cost.

TDD with behavioural tests is a good way to tune prompts and settings (temperature, top-p).

## E2E specifics

An E2E test boundary spans multiple components and external services. Invoking an endpoint is an E2E test (the controller orchestrates several services), not unit or integration. Manual E2E (for example through a UI) still finds things automation misses; automate the ones worth automating with an API test client or headless browser, and run E2E less often than unit/integration. If an E2E test fails but no unit/integration test does, there are blind spots or emergent behaviour. Vertical E2E covers one feature across layers (UI to DB); horizontal E2E covers many scenarios across integrated systems.

## Test dimensions and data

Dimensions: scope (what is tested, with boundaries), coverage (how much), comprehensiveness (how deep). Test data types: valid, invalid, boundary, and huge (for stress).

## Applying this when grading (Area 6)

- Is there a test suite proportional to the stakes (see calibration)? Are static checks (`mypy`) in place?
- Are LLM and external calls **mocked** (monkeypatch, `respx`, fixtures) so the suite is deterministic and does not hit a paid API in CI? Is DI used to substitute models in tests?
- For dynamic model output, is there **behavioural/property-based testing** (MFTs, ITs, DETs) or auto-evaluation against thresholds, rather than exact-match assertions that will be flaky?
- Is there a regression/monitoring plan for model drift, especially with hosted providers?
- For internet-facing services, are there adversarial tests that also exercise the auth guards?
- Do tests check behaviour rather than implementation detail?
