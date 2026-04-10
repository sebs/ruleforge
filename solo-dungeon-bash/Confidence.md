# Extraction Confidence Report — Solo Dungeon Bash

## Overall Confidence: **91%** (High)

This is an unusually clean extraction. The source rulebook is short (3 pages), structured, and written in imperative voice. Most ambiguities are edge cases that do not affect the core loop.

## Per-Section Breakdown

| Section | Confidence | Notes |
|---|---|---|
| Rules Extraction | **95%** | Nearly every rule is directly quotable. The only residual risk is around implicit behaviours (A-1, A-2). |
| Mechanics Identification | **94%** | Standard taxonomy maps cleanly. "Roll-and-write with king-move grid" is a hybrid, but each component is a known mechanic. |
| Ambiguous Rules | **90%** | Confident the 10 flagged items are *all* the ambiguities in the text; slight residual risk of undiscovered interaction (e.g. max stat caps, duplicate items) — but A-3 and A-4 both cover those. |
| Game Loop | **95%** | Turn sequence is explicit and numbered. The loop is easily represented. |
| Loop Validation | **93%** | All states have exits; termination is guaranteed by grid finiteness. Minor risk on the "blocked" semantics (L-1) — resolved by digital default. |
| Adaptation Gap | **92%** | Low risk. The game has no physical-only components. The only judgement calls are around *opportunities* the digital version can add, which are inherently open-ended. |
| Balance Parameters | **92%** | Every number is on one page; some sensitivity estimates are derived rather than empirically playtested, but the ordinal ranking should hold. |
| Interaction Model | **90%** | The model is simple because the game is simple. A few item-slot semantics depend on A-3's resolution. |
| Onboarding Design | **85%** | Recommendations are extrapolated from the rules + best practices. The tutorial flow is designed, not validated with users. |
| Feature List | **88%** | The must/should/could split is a judgement call; the MVP cut is defensible but another PM might draw the line differently. |
| Architecture | **90%** | Stack choice is suggestive, not prescriptive. The layering is solid. |
| User Stories | **89%** | 37 stories cover all must-have features plus key polish. Fibonacci scoring is judgement-based but internally consistent. |

## Weighted Average
| Section | Weight | Confidence | Weighted |
|---|---|---|---|
| Rules Extraction | 0.20 | 0.95 | 0.190 |
| Mechanics | 0.10 | 0.94 | 0.094 |
| Ambiguous Rules | 0.05 | 0.90 | 0.045 |
| Game Loop | 0.10 | 0.95 | 0.095 |
| Loop Validation | 0.05 | 0.93 | 0.047 |
| Adaptation Gap | 0.05 | 0.92 | 0.046 |
| Balance Parameters | 0.10 | 0.92 | 0.092 |
| Interaction Model | 0.05 | 0.90 | 0.045 |
| Onboarding Design | 0.05 | 0.85 | 0.043 |
| Feature List | 0.10 | 0.88 | 0.088 |
| Architecture | 0.05 | 0.90 | 0.045 |
| User Stories | 0.10 | 0.89 | 0.089 |
| **Total** | **1.00** | | **0.919** |

**Overall weighted confidence: 91.9% → rounded to 91%.**

## Drivers of Lost Confidence

### Where the missing 9% lives
1. **Item stacking semantics (A-3)** — literal vs. slot-based interpretation changes which builds are optimal. Deduct ~3%.
2. **Mid-combat potion timing (A-2)** — strict interpretation affects difficulty significantly. Deduct ~2%.
3. **Unplaytested balance predictions** — the top-5 sensitive parameter analysis is analytical, not empirical. Deduct ~2%.
4. **Judgement in feature/story prioritization** — another extractor might draw the MVP line differently. Deduct ~1%.
5. **Tutorial flow not user-tested** — design recommendation, not validated outcome. Deduct ~1%.

## Sections Above 90% (high-confidence, low manual-review need)
- Rules Extraction, Mechanics, Game Loop, Loop Validation, Adaptation Gap, Balance Parameters, Architecture

## Sections Below 90% (worth a human eyeballs pass)
- Onboarding Design (85%) — user-test the tutorial script.
- Feature List (88%) — run the must/should/could cut past product.
- User Stories (89%) — review Fibonacci scores with the dev team.

## Draft vs. Ready Status
**This extraction is READY for developer handoff.** Overall confidence exceeds the 60% draft threshold by a wide margin. The stages that sit below 90% are design/prioritization work, not extraction work — they're expected to be iterated on by humans.

## Recommended Human Review Checklist
- [ ] Read `AmbiguousRules.md` and ratify or override each of the 10 defaults.
- [ ] Review the Balance Sheet's top-5 sensitive params with the intended difficulty in mind.
- [ ] Validate the MVP cutline in `Features.md` against target timeline and team size.
- [ ] Decide whether accessibility (S033, S034, S035) and art (S037) should be promoted to "must" per product standards.
- [ ] Playtest the tutorial script in a dry run before implementation.

## Update to GDD Section 9
See `GDD.md` section 9. The final entry should read:

> **Overall Extraction Confidence: 91% (High).**
>
> Rules Extraction: 95% · Mechanics: 94% · Game Loop: 95% · Balance: 92% · Adaptation: 92% · Overall: 91%.
>
> Sub-90% sections: Onboarding (85%), Feature prioritization (88%), User stories (89%) — these represent design judgement calls, not extraction uncertainty. The extraction is READY for developer handoff.
