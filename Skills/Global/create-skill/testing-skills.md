---
name: testing-skills
description: Reference for testing skills before deployment — pressure scenarios and wording checks that verify a skill works and resists rationalization.
disable-model-invocation: true
---

# Testing Skills

Load this when creating or editing a discipline skill — one that enforces a rule against an incentive to skip it (test-first, verify-before-claiming, root-cause-before-fix). Pure reference and technique skills do not need it: they have no rule to violate, so the lighter self-test in `create-skill` Step 6 is enough.

Testing a skill is test-driven development applied to documentation. **The core principle: if you did not watch a subagent fail without the skill, you do not know the skill prevents the right failure.**

## The cycle

| Phase | For a skill | What you do |
|-------|-------------|-------------|
| RED | Baseline | Run the scenario in a fresh subagent without the skill; watch it fail |
| Verify RED | Capture | Record the choices and rationalizations verbatim |
| GREEN | Write | Address the specific failures observed, nothing hypothetical |
| Verify GREEN | Pressure-test | Run the same scenario with the skill; confirm compliance |
| REFACTOR | Close loopholes | Capture each new rationalization, add a counter, re-test |

## RED — baseline first

Run the scenario without the skill so you see what the agent does by default. Always include this no-guidance control: if the agent already does the right thing without the skill, there is nothing to fix — stop, and do not write the skill. Capture the exact rationalizations word for word; they become the rationalization table.

## Writing pressure scenarios

An academic prompt ("what does the skill say?") only makes the agent recite. A useful scenario gives it a reason to break the rule.

- Combine three or more pressures. A single pressure is easy to resist; stacked pressures expose the real failure.
- Force a concrete choice (A/B/C), not an open-ended answer.
- Use real constraints — specific times, named paths, actual consequences.
- Make the agent act ("what do you do?"), with no easy out such as deferring the decision.

| Pressure | Example |
|----------|---------|
| Time | Deadline, deploy window closing, production down |
| Sunk cost | Hours of work that it feels wasteful to delete |
| Authority | A senior or the user says to skip it |
| Exhaustion | End of day, already tired, wanting to stop |
| Social | Looking dogmatic or inflexible |
| Pragmatic | "Being pragmatic, not dogmatic" |

## Micro-testing the wording

Full pressure scenarios are the final gate, but they are slow per iteration. Check the wording itself first with cheap, fast samples:

- One fresh-context sample per call. The system prompt is the realistic context the guidance will live in (the full skill, not the line in isolation); the user message is a task that tempts the failure.
- Always include a no-guidance control. No failure in the control means nothing to fix.
- Five or more reps per variant — single samples lie.
- Read every flagged match yourself. Quoted counter-examples and template echoes masquerade as hits; automated counts alone mislead.
- Treat variance as a signal. When guidance binds, reps converge on the same shape. Five different interpretations across five reps means the wording is not binding yet — tighten the form before adding words.

## REFACTOR — close each loophole

When the agent complies in some runs but rationalizes its way out in others, plug the specific gap rather than adding generic sternness. For each new rationalization:

- Add an explicit negation that names the workaround ("do not keep it as reference" beats "do not cheat").
- Add a row to the rationalization table: the excuse, and the reality that defeats it.
- Add a red flag so the agent can self-catch the thought.
- Add the symptom to the `description` so the skill fires when the agent is about to violate.

Match the form of the counter to the failure — see "Matching form to failure" in [`writing-great-skills.md`](writing-great-skills.md). Generic counters do not bind; specific ones do.

## When the agent ignores a clear skill

If it chose wrong despite reading the skill, ask it directly how the skill should have been written to make the right choice unambiguous. Three answers, three fixes:

- "The skill was clear, I ignored it" — not a wording problem; add a foundational principle ("violating the letter is violating the spirit").
- "It should have said X" — a wording gap; add X verbatim.
- "I did not see section Y" — an organization problem; make the point more prominent and earlier.

## Done

The skill is ready when, under the pressure that made the baseline fail, the agent chooses correctly, cites the skill, and acknowledges the temptation it resisted. It is not ready while it still finds new rationalizations, argues the skill is wrong, or invents hybrid approaches to evade it.
