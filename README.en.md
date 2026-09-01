# Podcast/Audio Autopilot Kit — with Style Learning

*[繁體中文版 README](README.md)*

This is the tool that supports the editing pipeline for a podcast called
*The Long Way Here*, hosted by Olivia Pan and Nina Tseng (bi-weekly,
podcast + YouTube). Two things live here:

1. **Podcast publish pipeline** (`scripts/`) — built directly from the show's own
   [Production Playbook](docs/production_playbook.md): raw transcript → titles, show
   notes, YouTube description with chapters, podcast description, Buzzsprout draft,
   YouTube upload checklist.
2. **Edit-style learning tools** (`src/`) — ported from
   [Hao0321/video-autopilot-kit](https://github.com/Hao0321/video-autopilot-kit) by way
   of Olivia's own fork,
   [olieepop/video-autopilot-kit](https://github.com/olieepop/video-autopilot-kit)
   ("creator-voice-autopilot" locally). That fork's original contribution — learning what
   actually gets cut between a raw recording and the published edit, from your own
   pre/post transcript pairs — is the more useful and better-built version of what this
   repo originally tried to hand-build from scratch. Full credit to Hao0321 for the
   underlying framework and to Olivia's fork for `edit_style_model.py`/`rough_cut.py`;
   what's ported here is scoped down to just the transcript/subtitle pieces this show
   actually needs, not the full multi-format kit (Shorts, drama, silent vlog, etc. — see
   the fork if you want those). MIT-licensed; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## What the owner needs to provide

Nothing here runs on its own — it's a kit, not an autopilot in the "hands off"
sense. Four inputs only the owner (Olivia) can supply:

| Input | Where it lives | Used by |
|---|---|---|
| **Voice / brand style** | `templates/voice_profile.template.md`, `templates/style_profile.template.md` → fill in once as `profiles/*.md` (gitignored) | `scripts/script_01_text_outputs.py` (titles, show notes, descriptions) |
| **Edit style** | Not filled by hand — generated from your own raw+published transcript pairs via `src/edit_style_model.py learn` → `profiles/edit_style_profile.md` (gitignored) | `src/rough_cut.py propose`, future automated cut suggestions |
| **Episode outlines** | Google Drive → **"The Long Way Here"** folder → `EP0X_<topic>/01_Preparation/` — see [`docs/outline_location.md`](docs/outline_location.md) for the exact path and format | `src/rough_cut.py`'s topic-adherence tangent judgment (not wired up yet — needs a pre/post pair for an outlined episode first) |
| **API keys** | `.env`, from `.env.example` | `script_01` (Anthropic), `script_02` (Buzzsprout) |

Everything these tools produce is a draft to review, not a publish button —
see each script's own `_readme` field / docstring for what still needs a human
look before it goes out.

## Pipeline

```
Riverside recording
  -> raw transcript(s) (per-speaker, .srt/.vtt/.txt)
  -> scripts/prep_transcript.py          (flag mis-detected-language blocks)
  -> src/edit_style_model.py learn       (accumulate your edit-style profile — optional, informs cuts)
  -> src/rough_cut.py reconstruct        (exact cut list, once a real edit exists — training data)
  -> scripts/script_01_text_outputs.py   (titles, show notes, YT desc, podcast desc, quotes)
  -> CapCut edit (manual — src/dual_subtitle.py for bilingual burned-in captions)
  -> scripts/script_02_buzzsprout_upload.py  (podcast draft)
  -> scripts/script_03_youtube_prep.py       (YouTube upload checklist)
```

Scripts 1–3 mirror the Playbook's spec exactly. `prep_transcript.py` exists because
ep1's raw transcript exposed a gap the Playbook didn't anticipate — see
[docs/editing_learnings.md](docs/editing_learnings.md).

## Structure

- `scripts/` — the podcast publish pipeline, run manually via `python scripts/<name>.py`
- `src/` — edit-style learning + subtitle tools, ported (see attribution above):
  `edit_style_model.py` (learn a cut/retention profile from pre/post transcript pairs),
  `rough_cut.py` (propose/reconstruct/merge/apply a cut list), `dual_subtitle.py`
  (bilingual Traditional Chinese + English burned-in captions), plus their dependencies
  `delivery_media_ops.py` and `media_delivery_qa.py`
- `templates/` — fill-in-the-blank style/voice templates, plus
  `edit_style_profile.template.md` (how to read `edit_style_model.py`'s output); ported
- `docs/production_playbook.md` — mirror of the Drive production playbook, kept in the
  repo so the scripts' contracts are versioned alongside the code that implements them
- `docs/editing_learnings.md` — living notes on what actually happens between raw
  transcript and published episode, updated after each episode's edit. Append, don't
  overwrite.
- `docs/outline_location.md` — where episode outlines live in Drive, their format, and
  why ep1 (which has none) is a process outlier, not just a transcript-quality one
- `episodes/<epN>/` — per-episode inputs/outputs (transcripts are gitignored; generated
  content files are checked in as a record)
- `profiles/` — gitignored. Derived from your own raw, unedited speech and outlines —
  same privacy rule as the upstream kit: personal data never goes in the repo, only the
  tools that generate it from your own local files.

## Setup

```bash
pip install -r requirements.txt
cp .env.example .env   # fill in ANTHROPIC_API_KEY, BUZZSPROUT_API_KEY, BUZZSPROUT_PODCAST_ID
```

`src/` needs `ffmpeg`/`ffprobe` on PATH (for `dual_subtitle.py burn` and `rough_cut.py
propose`/`apply`); `edit_style_model.py learn` and `rough_cut.py reconstruct` are pure
Python, no `ffmpeg` needed.

## Status

- `scripts/script_01_text_outputs.py` — implemented, run once by hand against ep1's
  transcript, see `episodes/ep1/episode_ep1_content.md`. Needs a live
  `ANTHROPIC_API_KEY` to run for real.
- `scripts/script_02_buzzsprout_upload.py`, `scripts/script_03_youtube_prep.py` —
  scaffolded per the Playbook's spec, `script_03` verified against real output,
  `script_02` untested against live Buzzsprout credentials.
- `src/edit_style_model.py`, `src/rough_cut.py` — ported and verified against ep1's real
  transcripts (72.2 min raw → 191 cut ranges, 36.1 min kept — see
  `docs/editing_learnings.md`). Ep1's biggest single cut is flagged as a one-off, not a
  generalizable pattern — ep1 had no pre-recording outline (ep2+ all do), so it's an
  outlier on process, not just transcript quality. `src/dual_subtitle.py` is ported but
  not yet run against a real ep1 cut.
- Episode outlines confirmed to exist in Drive for ep2–ep7 (see
  `docs/outline_location.md`); topic-adherence tangent judgment isn't wired up yet — it
  needs a pre/post transcript pair for one of those episodes first, which doesn't exist
  in this repo yet.
