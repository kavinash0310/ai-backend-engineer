# Prompt design and structured model interaction

## What a prompt is

A prompt is the input context supplied to a model for one inference call. It can include system/developer instructions, user content, conversation history, retrieved evidence, tool results, examples and output-format requirements. The exact message format is model/provider-specific.

Prompt engineering is careful interface and experiment design. It does not make a model deterministic, factual or secure. Backend code must enforce authorization, validation, quotas and tool policy.

## 1. Write the task contract first

Define:
- the user's goal and allowed scope;
- what information is authoritative;
- required output schema and fields;
- what the model should do when evidence is missing or ambiguous;
- what it must not do;
- examples of acceptable and unacceptable behavior;
- evaluation cases that represent expected use and abuse.

A vague instruction such as “be helpful” is hard to evaluate. A task contract might say: “Extract the invoice number and total from the supplied page image. Return null when unreadable. Do not infer missing digits. Include a page reference.”

## 2. Context organization

A useful conceptual order:
1. stable task policy and output constraints;
2. authenticated application context that is safe to provide;
3. user request, labeled as untrusted input;
4. retrieved evidence, each with source ID and boundaries;
5. tool outputs, identified as untrusted observations;
6. explicit response schema and stop/abstention behavior.

The actual provider's role semantics and hierarchy differ. Do not assume plain delimiters create hard isolation. Avoid inserting secrets or irrelevant personal data just because the model has context space.

## 3. Specific instructions and examples

Good instructions use concrete verbs, scope, criteria and output contract. Examples can clarify ambiguous formats:
- zero-shot: instruction without examples;
- few-shot: a small set of input/output examples in context.

Examples should be representative, diverse and checked for contradictions. They consume tokens and can bias the output toward an unintended pattern. Test with examples removed and changed.

Use constrained output modes or JSON schemas when supported. Still parse and validate every field in code. Enforce enums, ranges, identity and business rules. If a required field is missing, return a safe error or one bounded repair attempt; do not recursively ask the model until it complies.

## 4. Separate generation from decisions

A model should propose content or a typed action candidate. The application then:
- validates shape;
- checks authorization using trusted identity;
- checks current database state;
- applies policy and budgets;
- performs the operation idempotently;
- records observable action metadata;
- handles errors and rollback.

Never run generated SQL, shell, Python or arbitrary URLs as trusted instructions. A strict output schema is one layer, not the whole security model.

## 5. Grounding, citations and refusal

For evidence-based work:
- provide source excerpts and stable IDs;
- ask the model to tie factual claims to evidence;
- require a defined response when support is absent;
- validate citation IDs against retrieved results;
- evaluate whether evidence actually supports each claim.

“Cite your sources” alone does not ensure genuine citations. The model may invent one or attach a real passage to an unsupported claim.

A useful assistant can ask a clarifying question, abstain or escalate. Define these states explicitly in the API rather than treating every non-answer as an error.

## 6. Prompt injection boundaries

User messages and retrieved/tool content can include hostile instructions. Label them as data and instruct the model not to treat them as authority, but rely on external controls for security. Limit tool capability, enforce permissions in code, never expose credentials in context, and test malicious text from both direct and indirect sources.

Prompt wording is not a substitute for tenant filtering, output escaping, least privilege or confirmation.

## 7. Iteration as an experiment

Change one factor at a time where possible. Keep:
- prompt/template version;
- model and tokenizer identifier;
- decoding parameters;
- retrieval/index version;
- evaluation dataset/rubric;
- latency, token usage and failure categories.

Use a fixed development set while iterating and retain a holdout for final comparison. Check quality by input type, language and no-answer cases. A prompt that improves handpicked examples can reduce behavior elsewhere.

## 8. Prompt length and context budget

Estimate the combined tokens of all instructions, history, retrieved chunks and expected answer. Define truncation/selection rules. Preserve high-priority policy and the current question; retrieve a smaller relevant evidence set rather than truncating unpredictably.

Longer prompts can increase latency/cost and distract the model. Repeated conversation history should be summarized or selectively retained under an explicit policy, while preserving important user intent and provenance.

## Questions and answers

**Q: Is prompt engineering programming?**  
A: It is part of the model interface and deserves versioning/evaluation, but it cannot replace ordinary code for authorization, state changes and invariants.

**Q: Do role labels prevent prompt injection?**  
A: They can shape model behavior but do not create a hard security barrier. Enforce permission and tool rules outside the model.

**Q: Does JSON mode guarantee correct data?**  
A: It can constrain syntax or shape. Values may still be fabricated, out of range, unauthorized or inconsistent with current state.

**Q: Should every prompt include many examples?**  
A: No. Examples consume context and can overfit behavior. Add them only when evaluation shows improvement on the target distribution.

## Practice

1. Rewrite a vague support-ticket prompt as a precise task contract with a no-answer behavior.
2. Produce a JSON schema for an action proposal, then list checks the schema cannot enforce.
3. Test an indirect prompt injection embedded in retrieved documentation.
4. Compare a prompt with/without few-shot examples on a fixed evaluation set.
5. Estimate a request token budget and define a deterministic evidence-truncation rule.

## Completion check

You can write a testable prompt contract, organize evidence, validate structured results, manage context budgets and keep security decisions in trusted application code.