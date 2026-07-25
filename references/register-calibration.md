# Register calibration — measured diction, tense, and connector norms

Read this once during the Phase 1 whole-paper read and draw on it when composing options. It is **calibration, not law**: the numbers below were measured on one corpus — 20 open-access 2025 *Nature Communications* CS/AI articles — and the author's field norms and profiled voice always win. Never copy example wording into the paper.

_Distilled and reworded from the corpus notes in [Yuan1z0825/nature-skills](https://github.com/Yuan1z0825/nature-skills) (`nature-polishing`, Apache-2.0)._

## Boosters: the `significantly` rule

`significantly` appeared **zero times** in the corpus. Treat every bare booster ("significantly better", "greatly improved") as a claim with a missing number: either the statistic exists — then attach it ("improved by 23%", "p < 0.01") — or it doesn't, and the booster is slop (principle 4). Softer alternatives (notably, substantially, markedly) still need a number or a test behind them; swapping the adverb is not the fix.

## Connectors

Measured preference: `However` (51) ≫ `Furthermore` (22) > `Therefore` (19) > `Overall`/`Notably` (16) > `In addition` (13) > `In contrast` (6) > `Moreover` (4).

- `However` does the real work of turns and gap-openings; don't scatter exotic substitutes where it is the idiom.
- For addition, `Furthermore` over `Moreover` — and better than either, no connective at all when given-new flow already carries the logic.
- Reserve `Notably` / `Importantly` / `Interestingly` for the paragraph's one key finding; as routine sentence openers they are decoration, and a tell.
- Demonstrative-led openers ("This suggests...", "These results...") — about one per paragraph. Past that, link by restating the noun ("Such heterogeneity...", "The resulting gradient...") or let zero-connective progression work.
- Always the smallest connective that does the job.

## Tense triad

- Standing property or background fact → **present** ("the model captures long-range order").
- A specific operation you performed → **past** ("we annealed the sample at 600 K").
- Figure description → **present** ("Fig. 2 shows").

Flag accidental past tense on a capability ("the method demonstrated..." when stating what it *does*) — an easy second-language slip.

## Evidence-strength verb ladder

- **Strong** (design and data justify it): show, demonstrate, establish, reveal, identify.
- **Moderate** (plausible, not definitive): suggest, indicate, are consistent with, point to.
- **Speculative** (beyond direct observation): may reflect, could arise from, appears to.

The verb is part of the claim: moving a sentence up or down the ladder changes meaning, so propose such a move only as an explicitly flagged option.

## Hedge placement

Hedges cluster in interpretation sentences, not observation sentences. The observation stays assertive and numeric ("the gap widens by 12 meV under strain"); the `may` lives in the meaning sentence ("this may reflect enhanced coupling"). A hedge sitting on an observation usually marks either misplaced caution (offer the move, flagged) or genuinely indirect data (then it must stay).

## Honest comparison

When the result is weaker than a baseline, the corpus pattern is `comparable` plus an explicit concession ("despite the smaller training set, comparable accuracy...") — never a quiet upgrade to `superior`. State the weakness; frame the trade.

## Two micro-heuristics

- The **last sentence of a paragraph** is where drafts sag — often the longest and weakest. Check it at each paragraph checkpoint.
- A sentence past ~20 words earns a **one-job check** (the job lens in SKILL.md): more than one main proposition is a reason to split, not to polish cosmetically. No hard length caps — the author's profiled rhythm wins.
