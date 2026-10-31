# Testing AI backends

## Two kinds of correctness

A backend with AI has deterministic software behavior and probabilistic model behavior. Test them separately, then test the complete path.

A **unit test** checks a small function, such as token-budget calculation. An **integration test** checks collaborating parts, such as API, database, and worker. A **contract test** checks that a service follows the request and response schema expected by clients. An **end-to-end test** exercises a complete user workflow. A **load test** measures behavior under concurrency and volume.

For AI, an **evaluation set** is a collection of representative inputs with expected properties or reviewer judgments. It can measure task success, factual support, refusal when evidence is absent, citation correctness, and formatting. It is not necessarily a set with one exact string answer.

## Keep tests stable without hiding failures

Mock a provider in unit tests so network cost and random responses do not make the test suite flaky. Then run a smaller integration suite against a real provider or staging environment when appropriate. Store secrets outside source control and use low-risk test data.

Avoid asserting that a generative answer equals one exact sentence. Assert stable requirements: response schema is valid, the answer cites supplied evidence, the service abstains for a no-evidence case, and the output respects an allowed category. Human review may still be needed for semantic quality.

Do not update expected results automatically after a failure. Review whether model behavior changed, evaluation data changed, or the expected behavior was wrong.

## Failure tests

Test a provider timeout, invalid JSON, rate limiting, client cancellation, database outage, queue retry, duplicated job, missing document, revoked permission, and oversized prompt. Verify that the service returns a safe error, does not leak internal details, and does not repeat side effects.

For RAG, test retrieval and generation separately. A wrong answer may come from missing source text, wrong permissions, poor ranking, or unsupported generation. Stage-level metrics make the cause visible.

## Coverage and human review

Build cases from realistic user requests, not only happy paths. Include ambiguous, adversarial, out-of-scope, and unanswerable inputs. Track data version, model identifier, prompt version, and scoring rubric. Protect private evaluation data. Review cases across languages and user groups relevant to the application.

Questions: Why not test only the model? The API, permissions, database, queue, and UI can fail even if the model performs well. Why mock providers? To make most tests fast, cheap, deterministic, and offline. What does a passing evaluation prove? It gives evidence for the tested cases and rubric, not every future input.

Practice: create 20 test cases for an internal knowledge assistant: supported answer, absent evidence, conflicting docs, access denial, injection text, provider timeout. Define pass criteria before running the system.

Further reading: [FastAPI testing](https://fastapi.tiangolo.com/tutorial/testing/) and [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/).
