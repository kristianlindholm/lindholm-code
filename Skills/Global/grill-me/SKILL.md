---
name: grill-me
description: Interview the user relentlessly, one question at a time, until a plan or design reaches shared understanding. Use when intent is vague and must be made concrete, or when another skill needs a requirements interview.
---

# Grill Me

Interview the user relentlessly about every aspect of this plan until you reach a shared understanding. The goal is coverage, not just depth: do not only follow the branches the user nudges toward — actively map the whole design space and surface the branches they have not raised, so nothing is left out at the end.

## Process

1. **Map the design space.** Before and during the interview, maintain an explicit design tree of every decision the plan implies — data, interfaces, edge cases, failure modes, dependencies, non-functional concerns (security, performance, scope). Note branches the user has not mentioned; these are the ones most likely to be missed. Keep this map current as answers arrive.

2. **Ask one question at a time.** Never batch. For each question, state your recommended answer and the reasoning behind it, so the user can confirm or redirect quickly.

3. **Explore before asking.** If a question can be answered by exploring the codebase, explore the codebase instead of asking the user.

4. **Cover breadth, not only the nudged branch.** When the user pulls you down one branch, resolve it, but keep the sibling and untouched branches on the map — return to them rather than letting them drop. Resolve dependencies between decisions one by one: settle the decisions that others depend on first.

5. **Gap check before closing.** When the user-driven branches feel settled, review the design tree for branches that were never visited or are still under-specified. Raise each remaining gap as its own question. Do not treat the interview as done while unvisited or vague branches remain.

6. **Closing loop.** Once the tree is covered, always ask the user whether there is anything else to add. If they add anything:
   - Fold the new information back into the design tree.
   - Re-examine how it fits the decisions already made — flag any it contradicts or reopens.
   - Derive the new questions it raises and return to step 2.

   Repeat this loop until the user has nothing further to add and no open or under-specified branch remains. Only then is the interview complete.

## Push for specificity — do not record vague answers

A vague, category-level, or undefined answer is not an answer. Do not record it and move on. Push once for the concrete version; if it is still vague, reframe constructively — offer a specific candidate answer for the user to confirm or correct — then record what they settle on. This applies to every answer across all the steps above, whatever the interview is about.

| Avoid accepting | Push for |
|---|---|
| A category or group ("the usual ones", "most of them") | A specific, named instance |
| An undefined or subjective term ("fast", "simple", "nice") | A concrete, checkable definition — how you would know it was met |
| A hand-wavy intention ("it should just work") | The exact behaviour, in the specific case |

Take a position rather than validating a non-answer: do not respond to a vague reply with "that's interesting" or "sounds good" — name what is still missing and ask for it. This is the sparring-partner posture applied to interviewing: specificity over social comfort.
