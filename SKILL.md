---
name: validate
description: Use AUTOMATICALLY after every /advisor output, plan, or architecture recommendation. Also when user types /validate. Spawns a validation subagent that scores findings by severity (BLOCKER/RISK/PREFERENCE) instead of reflexively hunting for blockers. Returns PASS/CALIBRATE/REJECT verdict.
---

# Validate

Spawns a validation subagent to stress-test any recommendation, plan, or architecture before presenting to the user.

**CRITICAL:** This skill MUST run after every `/advisor` output. Never present raw advisor results without validation.

## Critical Calibration Rules (read before every invocation)

The validator's natural failure mode is **reflexive negativity** - finding 6 BLOCKERs in
recommendations that are directionally correct but numerically optimistic. Codex audit
(2026-05-04) found this happens because the prompt was framed as "find every hole" without
calibrating severity, so the model over-indexes on failure modes.

**To counter this, the prompt below FORCES the validator to:**
1. Score every finding by severity: BLOCKER / RISK / PREFERENCE / CORRECTLY-IDENTIFIED
2. Acknowledge when the advisor is directionally right even if the numbers need adjustment
3. Distinguish "wrong number" from "wrong approach" - they require different fixes
4. State the verdict as PASS / CALIBRATE / REJECT, not just PASS / NEEDS-FIXES

## When to Use

- **Automatically** after every `/advisor` run (non-negotiable)
- **Automatically** after any architecture or plan recommendation
- User types `/validate [topic or context]`
- Before presenting any deliverable plan to the user

## Multi-Round Validation (Default: 4 Rounds)

**Empirically validated on 2026-05-11 (Makpatza session):** A single validation round is insufficient for strategic recommendations with legal, financial, or technical commitments. **Run validate up to 4 rounds by default** until verdict is PASS or no new findings emerge between rounds.

**What each round typically catches:**
- **Round 1:** Gross fabrications - invented prices, made-up percentages, wrong product names. Fast wins.
- **Round 2:** Wrong tool/stack names, imprecise legal citations, wrong regulatory thresholds. Subtle but credibility-damaging.
- **Round 3:** Misleading framing of true facts, missing scope items (SLA, ownership, budget), open-ended commitments. Strategic gaps.
- **Round 4:** Confirmation that prior calibrations held + verifying no new issues introduced by patches. PASS or stop.

**When to stop early:**
- Verdict is PASS and the validator explicitly says "no new findings"
- A round produces zero BLOCKERs and zero RISKs (only PREFERENCEs and CORRECTLY-IDENTIFIEDs)
- Validator starts manufacturing trivial PREFERENCEs as RISKs (sign of reflexive negativity)

**When to push past 4 rounds:**
- Recommendation is being sent to a regulator, lawyer, or contains binding legal language
- Money at stake exceeds ₪10K
- Round 4 still finds a BLOCKER (means earlier rounds missed structural issues - rethink approach)

**Round prompt augmentation (rounds 3+):** Tell the validator explicitly "this is round N - earlier rounds caught X, Y, Z. Look ONLY for genuine remaining issues. If draft is essentially shippable, say PASS. Don't manufacture issues to justify another round."

## How to Run

Spawn an Opus subagent with the Agent tool:

```
Agent(
  model="opus",
  prompt="""You are a critical validator. Your natural bias is toward reflexive negativity -
finding holes in everything to look thorough. Counter this bias actively.

Your job is NOT to maximize the count of issues. Your job is to give an accurate severity-scored
audit so the user knows what's truly broken, what's a calibration issue, and what's a preference.

For each finding, classify with one of FOUR labels (use the strictest accurate label):

**BLOCKER**: The recommendation will fail or cause real harm if shipped as-is.
Examples: Wrong tool that doesn't exist. Math error that changes the answer by >50%.
Missing critical step that will break the deliverable. Legal/safety risk.

**RISK**: The recommendation might fail under specific conditions, OR is directionally right
but numerically off by 20-50%.
Examples: Time estimate is 1.5x too low. Cost range is plausible but missing a category.
Approach is sound but assumes traction the user doesn't have.

**PREFERENCE**: You'd do it differently, but the recommendation is defensible.
Examples: Different framing. Alternative tool with similar tradeoffs. Stylistic differences.

**CORRECTLY-IDENTIFIED**: The advisor already addressed this point well. Acknowledge it.

Then, for each BLOCKER and RISK only, give:
- Why it's a problem (1 sentence)
- Concrete fix (1 sentence with specific number/action)

For each PREFERENCE, just note it - don't demand it be fixed.

For each CORRECTLY-IDENTIFIED, briefly affirm. This is critical - if you find nothing
correctly identified, you're hunting for problems instead of evaluating.

Then end with a final verdict from these THREE options:

**PASS**: 0 BLOCKERs, ≤2 RISKs. The advisor's recommendation is shippable, possibly with
minor numerical adjustments.

**CALIBRATE**: 0 BLOCKERs, but 3+ RISKs. The recommendation's direction is correct but the
numbers/scope need adjustment before shipping. List the specific calibrations needed.

**REJECT**: 1+ BLOCKERs. The recommendation has fundamental problems and should be redrawn,
not patched.

Format your response as:

## Findings

| # | Finding | Severity | Fix |
|---|---------|----------|-----|

## Verdict: PASS / CALIBRATE / REJECT

## Required Calibrations (only if CALIBRATE or REJECT)

[Specific concrete changes to make]

The recommendation to validate:

[PLAN OR RECOMMENDATION TO VALIDATE]
"""
)
```

## Output Format

Present validator's findings honestly. If verdict is PASS or CALIBRATE, ship the advisor's
recommendation with the calibrations applied. Only redraft completely if REJECT.

Three rules when presenting validation:
1. Don't pretend BLOCKERs exist when they don't (most things are PREFERENCEs)
2. Don't soften REJECTs into CALIBRATEs to avoid bad news
3. Always show the CORRECTLY-IDENTIFIED items so the user sees what advisor got right

## Integration with /advisor

The correct flow:

```
/advisor → produces draft with KNOWN/ASSUMED/MISSING + ranges + confidence
    ↓
/validate → scores findings by severity, produces PASS/CALIBRATE/REJECT
    ↓
If PASS → present advisor's draft + validator's affirmations
If CALIBRATE → apply calibrations, present updated version + audit trail
If REJECT → redraft from scratch, run /advisor again with fixed framing
```

## Example

User: `/advisor איזו ארכיטקטורה לבוט AI`

→ Advisor returns recommendation with KNOWN/ASSUMED/MISSING and confidence MEDIUM
→ Validator scores: 1 BLOCKER (Railway has no free tier), 2 RISKs (token cost estimate low),
   1 CORRECTLY-IDENTIFIED (auth approach is right)
→ Verdict: REJECT (because BLOCKER exists)
→ Redraft with Hetzner instead of Railway
→ Re-validate, this time PASS with calibrations
→ Present final recommendation to user
