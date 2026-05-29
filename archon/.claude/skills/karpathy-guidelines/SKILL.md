---
name: karpathy-guidelines
description: |
  Behavioral guardrails for coding agents. Use when planning, implementing,
  reviewing, or simplifying code so the work stays assumption-aware, minimal,
  surgical, and explicitly verified.
---

# Karpathy Guidelines

This skill translates the viral "Karpathy coding guidelines" into an execution
contract Archon can preload on Claude workflow nodes.

It is not a writing style. It is a failure-mode guardrail for:

- hidden assumptions
- over-engineering
- unnecessary blast radius
- weak or missing verification

Use these rules during planning, implementation, review, and cleanup.

## 1. Think Before Coding

Before changing code:

- State the assumptions that matter.
- If the request could mean multiple things, name the ambiguity instead of
  silently picking one.
- Prefer the simpler interpretation unless the evidence points elsewhere.
- Surface tradeoffs when they affect the design or scope.
- If a critical detail is unclear, stop and ask instead of guessing.

## 2. Simplicity First

Write the minimum code that solves the task.

- Do not add features that were not requested.
- Do not create abstractions for one-off logic.
- Do not add configurability, extension points, or indirection "for later."
- Do not invent defensive code for implausible scenarios.
- If the solution feels larger than the problem, reduce it.

Ask: would a strong senior engineer call this overbuilt? If yes, simplify.

## 3. Surgical Changes

Keep the blast radius tight.

- Touch only the files and lines needed for the task.
- Do not opportunistically refactor adjacent code.
- Match the local style unless the task explicitly includes cleanup.
- Remove only dead code, imports, or variables created by your own change.
- If you notice unrelated problems, mention them separately instead of folding
  them into the current patch.

Every changed line should trace back to the request or to verification required
by the request.

## 4. Goal-Driven Execution

Convert vague work into explicit, verifiable goals.

- Define what success looks like before coding.
- For multi-step work, keep a short plan with a verification check per step.
- Prefer tests or concrete checks over subjective claims like "should work now."
- Do not claim completion until the relevant verification has run.

Use this pattern:

1. [step]
   verify: [specific command, test, or observable outcome]
2. [step]
   verify: [specific command, test, or observable outcome]

## Review Lens

When reviewing code, look specifically for:

- assumptions that were never stated or checked
- abstractions that do not buy enough value
- changes that spill beyond the task boundary
- missing proof that the change actually works

Push the solution toward smaller, clearer, and more easily verified code.
