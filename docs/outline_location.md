# Where episode outlines live

Google Drive → **"The Long Way Here"** folder
(`https://drive.google.com/drive/folders/1y0EmIj2K3cAOHCU-91Lgd--A4D4OH94i`) →
`EP0X_<topic-slug>/01_Preparation/EP0X_Outline_<title>` — one Google Doc per
episode, written by the lead host before recording (per
`docs/production_playbook.md` Phase A).

**Ep1 does not have one.** Every episode from ep2 onward does. This is very
likely part of *why* ep1 needed such heavy tangent-cutting in the edit (see
`docs/editing_learnings.md`) — there was no pre-agreed topic structure to keep
the raw conversation anchored to, so it ran long and wandered. Worth treating
ep1 as an outlier on this basis alone, separate from the transcript-language
issue.

## Outline format (from `EP04_Outline_我媽覺得我還不算成功`, representative of ep2+)

- Title + subtitle + 2 alt title options
- Target length, lead host, proofreader
- **Episode purpose** — one paragraph, written *before* recording, stating
  what the episode is about and the one thing a listener should take away
- Templated cold open + intro (reused show-wide, filled in per episode)
- **3–4 timed segments** (~10 min each), each with bullet talking points
- A recurring "coach's corner" / listener Q&A segment with example questions
- Outro + CTA
- Pre-recording notes, including **pre-marked IG/YouTube clip candidates** —
  moments the host already expects to be shareable, picked before the episode
  is even recorded

## Why this matters for the automation

`src/rough_cut.py`'s tangent-judgment step (2b in its pipeline docstring)
calls for exactly this: diff the raw transcript against the episode's planned
topics, and treat segments that don't map to any planned topic *and* run long
as the strongest tangent signal available — stronger than the mechanical
dead-air/learned-filler heuristics in `propose_cuts()` alone.

This isn't wired up yet — it needs a pre/post transcript pair for an episode
that *has* an outline (ep2+), which doesn't exist in this repo yet. Once one
does: pull the outline doc's segment list, hand both to an LLM per the
TANGENT_JUDGMENT_RECIPE in `src/rough_cut.py`'s docstring, and merge the
result with `propose_cuts()`'s mechanical candidates via `rough_cut.py merge`.
