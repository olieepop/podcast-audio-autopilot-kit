# YouTube Automation — The Long Way Here

Production automation for *The Long Way Here* (Olivia Pan + Nina Tseng), a bi-weekly
podcast/YouTube show. This repo implements the "Claude Code Automation" section of the
[Production Playbook](docs/production_playbook.md): scripts that turn a raw Riverside
transcript into every piece of written content an episode needs.

## Pipeline

```
Riverside recording
  -> raw transcript(s) (per-speaker, .srt/.vtt/.txt)
  -> scripts/prep_transcript.py     (merge tracks, flag/fix language issues)
  -> scripts/script_01_text_outputs.py   (titles, show notes, YT desc, podcast desc, quotes)
  -> CapCut edit (manual, using the trim pattern in docs/editing_learnings.md)
  -> scripts/script_02_buzzsprout_upload.py  (podcast draft)
  -> scripts/script_03_youtube_prep.py       (YouTube upload checklist)
```

Scripts 1–3 mirror the Playbook's spec exactly. `prep_transcript.py` is new — it exists
because ep1's raw transcript exposed a gap the Playbook didn't anticipate (see
[docs/editing_learnings.md](docs/editing_learnings.md)).

## Structure

- `scripts/` — the automation scripts, run manually via `python scripts/<name>.py`
- `docs/production_playbook.md` — mirror of the Drive production playbook, kept in the
  repo so the scripts' contracts are versioned alongside the code that implements them
- `docs/editing_learnings.md` — living notes on what actually happens between raw
  transcript and published episode, updated after each episode's edit. Append, don't
  overwrite.
- `episodes/<epN>/` — per-episode inputs/outputs (transcripts are gitignored; generated
  content files are checked in as a record)

## Setup

```
pip install -r requirements.txt
cp .env.example .env   # fill in ANTHROPIC_API_KEY, BUZZSPROUT_API_KEY, BUZZSPROUT_PODCAST_ID
```

## Status

Scripts 1 (text outputs) is implemented and has been run once by hand against ep1's
transcript — see `episodes/ep1/episode_ep1_content.md`. Scripts 2 and 3 are scaffolded
per the Playbook's spec but untested against live Buzzsprout/YouTube credentials — treat
them as a starting point, not verified.
