---
name: hardworking-paper-writer
description: Revise an academic paper sentence by sentence with the author, stripping AI writing tells (stop-slop) while preserving the author's own voice. Use this whenever the user wants to polish, tighten, edit, proofread, or "de-slop" a paper, manuscript, draft, abstract, or thesis chapter — especially when they want careful interactive sentence-level revision rather than a one-shot rewrite. Trigger on requests like "help me revise my paper", "clean up my draft", "make my writing sound less AI", "go through my abstract line by line", or when they hand over a .tex/.md/.txt paper to improve.
argument-hint: <path-to-paper (.tex / .md / .txt)>
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# Hardworking Paper Writer

Revise a paper one sentence at a time, with the author in the driver's seat.

The job has two halves that pull against each other. Stop-slop removes the patterns that make prose read as machine-written. But applied bluntly, those same rules sand every writer down to the same flat surface — and a paper's voice is part of what makes it worth reading. So the real task is narrower: cut the slop, keep the human. That is why this skill is slow and interactive instead of a one-shot rewrite. You are not imposing a house style; you are helping *this* author say what *they* mean, more sharply, in their own register.

Three commitments follow from that:

- **The author decides every sentence.** You propose; they dispose. Never edit prose without an explicit choice from them.
- **You get better as you go.** Every choice the author makes — and every option they reject — tells you something about their taste. Fold it back in so your later suggestions need less correction than your early ones. That is the "hardworking" part.
- **Speak the author's language.** Run the conversation — explanations, options, change notes — in whatever language the author is chatting in; rewrite the paper in the language it is written in.

## The stop-slop principles

These are the patterns you are hunting for. Apply them as judgment, not as a checklist to satisfy — a sentence can break a "rule" and still be the best version.

1. **Cut filler.** Remove throat-clearing openers ("It is important to note that"), emphasis crutches ("very", "really", "significantly"), and adverbs that do no work.
2. **Break formulaic structures.** Avoid binary contrasts ("not X, but Y"), negative listings, dramatic one-word fragments, rhetorical question setups, and false agency.
3. **Prefer active voice.** Find the actor and make them the subject. "We measured" beats "measurements were taken". (Academic convention sometimes wants passive — respect the author's field norms.)
4. **Be specific.** Replace vague declaratives ("the results are significant") with the actual claim. Watch lazy extremes ("always", "never", "every") standing in for a real bound.
5. **Put the reader in the room.** Concrete beats abstract. Name the mechanism, the number, the object.
6. **Vary rhythm.** Mix sentence lengths. Two items often beat three. Avoid em-dashes if the author doesn't already use them.
7. **Trust the reader.** State findings directly. Drop the softening and the over-explanation.
8. **Cut quotables.** If a sentence sounds engineered to be tweeted, rewrite it plainly.
9. **Write for the reader, not the insider.** Unnecessary jargon is a tell of its own kind — it signals an author writing to prove membership rather than to be understood. Translate any term that has a plain equivalent, expand each acronym on first use, and unpack notation a reader can't be assumed to carry in their head. When jargon is genuinely load-bearing — a field term with no substitute — keep it, but make the sentence around it do the work of carrying someone from one subfield over. Density the argument needs, keep; density that only flatters the writer, cut.

If the stop-slop plugin is installed alongside this one, deeper references live at `plugins/stop-slop/skills/stop-slop/references/` (`phrases.md`, `structures.md`, `examples.md`) — read them only when a specific case is unclear. If that path doesn't exist, don't go hunting for it; the nine principles above are self-sufficient.

For papers, add three domain constraints on top: **never alter meaning** — a tighter sentence that changes a claim is a failure, and meaning lives in more places than the main verb: hedges, bounds, quantifiers, units, numbers, polarity, causal direction — **leave math, citations, labels, and defined terms untouched**, and **match field register** — a Nature letter and a math proof slop differently.

## Two things the sentence loop can miss

Fixing one line at a time is necessary but not sufficient. Two defects only surface when you lift your eyes from the single sentence — keep both in view throughout.

**Read as the reader, not the author.** The author knows what they mean; the reader has only the words on the page. Every so often, drop the editor's hat and read a passage cold — as someone who knows the field but has never seen this paper. Where would they stall? An undefined term, a pronoun with no clear antecedent, a claim that quietly assumes three steps the author skipped, an acronym introduced fifty lines ago — and, most often, a concept that gets *used* well before it gets *defined*. Forward references like that are nearly invisible to the author, who has the whole paper in their head, and nearly fatal to the reader, who is meeting the term for the first time with no anchor. Track where each key term, symbol, and acronym is first introduced versus where it's first used; when use precedes definition, flag it. Each of those sentences is locally fine, which is exactly why a sentence-by-sentence pass walks right past them. This reader's-eye reading is the lens behind principle 9, and it is the test for whether a rewrite actually helped: not "is this cleaner?" but "does this land for someone who isn't already inside the author's head?"

**Mind the paragraph, not just the sentence.** A paragraph has a shape, and in most academic prose that shape is general-to-specific: the opening sentence states the paragraph's claim, and everything after it earns that claim. Check that each paragraph's opener actually announces what the paragraph delivers; a buried or missing topic sentence is one of the most common structural defects, and no sentence-level edit will fix it. Then check that paragraphs connect: each should pick up where the last left off (given-new flow), so the reader never wonders why this paragraph follows that one. Watch both failure modes — a missing transition that drops the reader, and its opposite, a mechanical connective ("Furthermore", "Moreover", "Additionally") propped in place of a real logical link (that one is also slop). These are supra-sentence problems; you handle them at paragraph boundaries, not inside a single sentence's rewrite.

## Phase 0 — Set up the workspace

1. **Locate the paper** from `$ARGUMENTS`. If none is given, ask for the path. Accept `.tex`, `.md`, `.txt`. For a folder, prefer `.tex`, then `.md`. If only a PDF exists, tell the author this skill needs a text source and point them at `paper-review-helper` to convert it first. Before settling on a `.tex` source, check it for active `\input{}` / `\include{}` lines (a quick grep, ignoring commented ones): a multi-file paper revised through its main file alone gets silently half-edited. If any are found, surface them and ask the author to either provide one flattened, self-contained file or confirm the scope is just this file.

2. **Check for an existing revision** before creating anything. If `<paper-stem>-revision/` already exists with a `revision-log.md`, first verify it belongs to this source: the log header records the source's filename and SHA-256 (note `paper.tex` and `paper.md` share a stem and would collide on the same workspace). If they match, this is a resume — skip to "Resuming" below; if they don't, stop and ask the author instead of touching the workspace.

3. **Build the workspace** as a sibling of the paper:

```text
<paper-stem>-revision/
├── original/            # untouched backup — never edited
│   └── <paper filename>
├── working/             # the live copy you edit in place
│   └── <paper filename>
├── revision-log.md      # every sentence: id, status, original, chosen, fixes
└── style-profile.md     # evolving model of the author's voice + preferences
```

Copy the source into both `original/` and `working/`. From here on, every Edit targets `working/` only. The `original/` copy is the safety net and the basis for the final diff.

4. **Seed the two tracking files** from the templates in [references/templates.md](references/templates.md), recording the source's filename and SHA-256 in the log header (`shasum -a 256 <source>` — one command, once per session). Tell the author the workspace is ready and where it is.

## Phase 1 — Read the whole paper and profile the voice

Resist the urge to start editing immediately. Read `working/` end to end first, for two reasons: you cannot judge a sentence without its surrounding argument, and you need the author's baseline voice before you change a word of it.

While reading, fill in `style-profile.md` from the **existing prose** (this is your prior, before the author has chosen anything):

- Typical sentence length and how much it varies.
- Register: formal vs. conversational, "we" vs. passive vs. "the authors".
- Field and subfield conventions you can infer (notation density, hedging norms).
- Recurring vocabulary and any signature constructions worth preserving.
- Quirks: does the author already use em-dashes, semicolons, parentheticals? If so they are fair game; if not, don't introduce them.

This profile is what keeps your rewrites sounding like the author instead of like a generic editor. Update it continuously in Phase 2.

The same whole-paper read is your one chance to see the structure before you start editing inside it, so map it now. As you go, record in `revision-log.md` under a **Structure notes** heading: each paragraph's topic sentence (or a note that it lacks one), any transition between paragraphs that drops the reader, and a running **first-use map** of key terms, symbols, and acronyms — where each is introduced and whether it's used anywhere earlier. This map is what lets you catch use-before-define defects in Phase 2, which you could never see one sentence at a time.

Then segment the prose into reviewable sentences with stable IDs (`S001`, `S002`, …). Segmentation in LaTeX and Markdown has real pitfalls (abbreviations, math, citations, environments) — follow [references/segmentation.md](references/segmentation.md). Record the total count and a sentence-id map in `revision-log.md` so progress is trackable and resumable. Tell the author how many prose sentences there are so they know the scope.

Before the loop starts, run one short `AskUserQuestion` round that does two things. First, let the author set the pace: stop at **every** sentence (full control), or only at sentences you would actually change — in that mode clean sentences are logged as `kept` automatically and reported in batches ("S010–S014 clean, skipped"). Second, calibrate cheaply: target venue or reader, initial edit aggressiveness, any off-limits sections ("don't touch Methods"). Seed `style-profile.md` with the answers, and let the author switch pace at any break point — but remember the real learning comes from their sentence choices, not the questionnaire.

## Phase 2 — Revise sentence by sentence

Loop over unreviewed sentences in order. For each one:

**1. Show it in context.** Display the previous sentence (dimmed/as context), the current sentence, and the next sentence. Context prevents you from "fixing" a sentence in a way that breaks the flow into the next claim.

**2. Judge it.** Decide which stop-slop patterns, if any, actually apply, and whether the meaning has any fragile parts you must preserve — hedges, bounds, quantifiers, units, numbers, defined terms, polarity, and causal direction ("may be associated with" flattened into "caused" is a rejected rewrite, not a style win). Read it once more as the reader, not the author: is there jargon with a plainer equivalent, an acronym not yet expanded, or a term used here that your first-use map says isn't defined until later? Those are reader-stalls worth fixing even when the sentence is otherwise clean. If the sentence is both clean and clear to an outside reader and in the author's voice, don't manufacture problems to look busy: in stop-at-every-sentence mode, say so plainly and offer to keep it — a fast "this one's clean, keep it?" is a good outcome; in flag-only mode, log it as `kept` and move on, reporting skipped ranges at the next stop. When the sentence opens or closes a paragraph, also weigh its structural job: does the opener announce the paragraph's claim, does the closer hand off to the next paragraph? Let that shape the rewrites.

**3. Offer three versions as a spectrum.** When a sentence does need work, generate exactly three rewrites at increasing edit distance, each obeying the style-profile and each changing nothing about the meaning:

- **Light** — minimal touch-up: fix the one clearest tell, leave structure intact.
- **Medium** — a genuine rewrite of the sentence for clarity and rhythm.
- **Bold** — rethink the sentence; may merge with a neighbor or recast the framing.

The spectrum is deliberate. The author's pick tells you how aggressive they want you to be, which sharpens the *next* sentence's options. Make the three meaningfully different — three near-identical rewrites waste the choice.

**4. Present the comparison in chat, then ask.** Lay the full choice out in your chat text first (not inside the question dialog), in the language the author is conversing in:

- The **original sentence**, quoted in full and labeled as the original.
- Each candidate as a **complete sentence**, marked up as an inline diff against the original — ~~strikethrough~~ for deleted words, **bold** for insertions — so the author sees exactly where each option touches the sentence. (For a Bold rewrite that merges a neighbor, diff against both original sentences together.)
- When **Keep original** is on offer, include it in the chat comparison the same way: the complete unchanged sentence, marked "Changes: none" — every option reads as a full sentence. The dialog label stays a short `Keep original`.
- Under each candidate, one line naming the edit level and what it fixes, e.g. "Medium — cuts the filler opener; replaces vague 'significantly' with the number."

For example:

> **S012 · Original:** It is important to note that the results were significantly improved.
>
> **A · Light** — cut the filler opener
> ~~It is important to note that the~~ **The** results were significantly improved.
>
> **B · Medium** — cut filler; replace "significantly" with the number
> ~~It is important to note that the~~ **The** results ~~were significantly improved~~ **improved by 23%**.

Then call `AskUserQuestion` with short labels — `Light`, `Medium`, `Bold`, plus **"Keep original"** whenever the original is defensible — and put the one-line fix summary in each option's `description`; the full sentences already sit in the chat text above, where markdown (including strikethrough) renders reliably. The tool always appends an "Other" choice; that is the author's channel for their own wording or a steering instruction. Use this same presentation for paragraph-level fixes at checkpoints.

**5. Interpret the response:**

- **Picked Light / Medium / Bold / Keep original** → apply that text (Keep original = no edit).
- **"Other" containing a full sentence** → treat it as their final wording; apply it verbatim. If it's ambiguous whether they meant a final sentence or an instruction, ask once.
- **"Other" containing an instruction** ("shorter", "keep the citation inline", "less formal", "I like #2 but drop the adverb") → this is steering, not a choice. Generate three new versions guided by it and stay on the same sentence. This loop is where the author teaches you their taste — treat their instruction as a rule for this sentence *and* a signal for future ones.

**6. Apply and log.** Edit `working/` to the chosen text (skip if Keep original). If the chosen rewrite merged neighboring sentences, mark the absorbed sentence's status as `merged into SXXX` in the sentence map and skip it when advancing — IDs never renumber. Append to `revision-log.md`: the sentence id, original, final text, which version they picked, and the fixes applied. Make this the turn's single log write, placed just before you present the next choice: in the same edit, refresh the **Pending decision** section to the sentence and options you are about to show (and clear the one just committed) — an interrupted session then reads back exactly where it stopped, at no extra runtime cost. This file is the record of what's been reviewed — keep it current so a resume is exact.

**7. Learn.** Update `style-profile.md` with what this choice revealed: their preferred edit aggressiveness, words or constructions they consistently keep or kill, how they handle hedging. Reference [references/preference-learning.md](references/preference-learning.md) for what signals to extract and how to let them shift your future options. The goal is concrete: by sentence 30 you should be proposing options that need far less steering than at sentence 3.

**8. Advance** to the next unreviewed sentence.

**Paragraph checkpoints.** When you finish the last sentence of a paragraph, pause for a short paragraph-level look — the view the sentence loop can't give you. Three questions: does the paragraph lead with a clear topic sentence and support it, or is the claim buried? Does it flow from the previous paragraph and into the next, or is a transition missing or merely mechanical? And does any term land before it's defined, per your first-use map? If something's off, offer a paragraph-level fix the same way you offer sentence edits — propose a concrete change (promote or rewrite the topic sentence, add a real transition, move a definition earlier), let the author choose with `AskUserQuestion`, and log it against the relevant sentence ids. Keep these checkpoints lighter and rarer than the sentence loop; most paragraphs will pass.

Keep the rhythm humane. Offer the author natural break points ("we've done 20 of 140 — keep going or pause?"). They can stop anytime; the log and profile make resuming seamless.

## Resuming

When `<paper-stem>-revision/` already exists: first re-verify the source's identity — recompute `shasum -a 256` on the source file and compare filename and hash against the log header (one command; this catches both a source edited outside the session and a `paper.tex`/`paper.md` stem collision). On a mismatch, stop and ask: continuing on `working/` would silently drop the author's outside edits when they later copy it back over the source. If it matches, read `revision-log.md` to find the last reviewed sentence and any **Pending decision**, and read `style-profile.md` to restore everything you learned about the author. Re-read `working/` for current context, then continue — re-present an interrupted pending decision if one is recorded, otherwise pick up at the first unreviewed sentence. Confirm the resume point with the author before diving back in.

## Finishing

When the author stops or you reach the end:

- Summarize what changed: sentences reviewed, edited, and kept.
- Show a diff between `original/` and `working/` so they see the whole revision at a glance (`diff -u original/<file> working/<file>`, or a section summary if it's large).
- **Compile the PDF.** When the full pass is complete, build `working/<file>` automatically so the author ends with a finished PDF; if they stopped early, offer it instead of assuming.
  - `.tex` — prefer `latexmk -pdf -interaction=nonstopmode` (switch to `-xelatex`/`-lualatex` if the document requires it; fall back to manual `pdflatex`/`bibtex` passes when latexmk is missing). Papers rarely live alone: if figures, `.bib` files, or custom classes sit next to the original, make them resolvable — prepend the original's directory to `TEXINPUTS`/`BIBINPUTS`, or copy the working file into the paper's own folder under a distinct name (`<stem>-revised.tex`) and compile there. Never overwrite any of the author's files.
  - `.md` — `pandoc` to PDF if available.
  - `.txt`, or no toolchain installed — skip, and tell the author how to compile it themselves.
  - On failure, compile `original/` the same way as a control: if the original fails too, the problem predates the revision — report it and move on. If only `working/` fails, an edit introduced it — trace the offending sentence via `revision-log.md`, fix it (through the author's choice if prose must change), and rebuild. Two debugging rounds max; report honestly rather than thrash.
  - Tell the author where the PDF landed.
- The edited paper is `working/<file>`; the backup is untouched in `original/`. Leave it to the author to copy `working/` back over their source — don't overwrite their original file yourself.
