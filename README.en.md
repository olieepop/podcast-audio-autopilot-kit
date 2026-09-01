# YouTube Automation — The Long Way Here

*[繁體中文版 README](README.md)*

Production automation for *The Long Way Here* (Olivia Pan + Nina Tseng), a bi-weekly
podcast/YouTube show. Two things live here:

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
- `templates/edit_style_profile.template.md` — how to read `src/edit_style_model.py`'s
  output; also ported
- `docs/production_playbook.md` — mirror of the Drive production playbook, kept in the
  repo so the scripts' contracts are versioned alongside the code that implements them
- `docs/editing_learnings.md` — living notes on what actually happens between raw
  transcript and published episode, updated after each episode's edit. Append, don't
  overwrite.
- `episodes/<epN>/` — per-episode inputs/outputs (transcripts are gitignored; generated
  content files are checked in as a record)
- `profiles/` — gitignored. `src/edit_style_model.py`'s output is derived from your own
  raw, unedited speech — same privacy rule as the upstream kit: personal data never goes
  in the repo, only the tools that generate it from your own local files.

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
  `docs/editing_learnings.md`). `src/dual_subtitle.py` is ported but not yet run against
  a real ep1 cut.
