# audio-to-notes

Drop a voice memo into a Google Drive folder → get a clean, structured note in your Obsidian vault. Hands-off after setup.

Phone records audio → Drive Autosync → Drive Desktop syncs to your Mac → Claude Code transcribes via Whisper → structured markdown lands in Obsidian → audio moves to `processed/`. The inbox folder always shows what's pending.

## What this gives you

- A `/transcribe` slash command for Claude Code that scans an inbox and processes new files automatically
- A skill (`utility-transcribe`) that handles the messy bits: Samsung's quirky `.m4a` format, files over 25MB, hour-long recordings
- Structured output: BLUF summary, actions with owners, discussion notes, risks, stakeholder pulse
- Two transcription engines you can pick from: **OpenAI Whisper API** (default, works anywhere, ~$0.006/min) or **mlx-whisper** (free, local, Apple Silicon only)

## Two ways to use this

### Path A: full automation (recommended)
The real thing. Claude Code + slash command + skill + Obsidian. See [INSTALL.md](./INSTALL.md).

### Path B: lightweight (no setup)
Just want to transcribe one file without setting up Claude Code? Open [claude.ai](https://claude.ai), connect Google Drive (Settings → Connectors), attach an audio file, and paste one of the prompts from [`prompts/`](./prompts/). One-shot, no automation.

## How the automation works

1. **Capture**: phone records voice memo → auto-syncs to a Google Drive folder (e.g. via Autosync on Android, Voice Memos sync on iOS, or any drag-and-drop into Drive).
2. **Sync to Mac**: Google Drive Desktop mirrors that folder locally.
3. **Run the slash command**: in Claude Code, type `/transcribe` (or *"any new memos to process"*).
4. **Claude transcribes**: scans the inbox, picks up files without a `.txt` companion, runs Whisper, writes a structured markdown note into your Obsidian vault.
5. **Audio moves to `processed/`**: marks the file as done. Inbox stays clean.

You can run it ad-hoc, or wire it to a cron / hook to run on a schedule.

## Output format

Each transcription becomes one markdown file in your Obsidian vault, named `YYYY-MM-DD - <slug>.md`, with YAML frontmatter for Dataview / search:

```markdown
---
title: "Site visit retro Tuesday"
type: voice-note
date: 2026-04-29
source: "2026-04-29 site visit.m4a"
area: ProjectAlpha
people: [Sam, Priya]
tags: [voice-note, retro, site-visit]
---

## Summary (BLUF)
Two paragraphs leading with the single most important point.

## Actions
- Send revised quote to Priya by Friday — Sam
- Book follow-up site visit week of 5 May — TBC owner

## Discussion & Notes
Cleaned prose, organised by topic.

## Risks & Blockers
- Permit deadline 14 May, no submission yet

## Stakeholder Pulse
- Priya: positive, wants weekly cadence
- Sam: cautious on timeline, asked for slack
```

See [`examples/voice-note-template.md`](./examples/voice-note-template.md) for a full sample.

## Detection logic (why it doesn't reprocess)

A file in the inbox is "new" only when **both**:
1. It's in the inbox root (not in `processed/`)
2. There's no `.txt` companion next to it with the same basename

That `.txt` is the source of truth — moving the audio to `processed/` is secondary. Means you can glance at the inbox folder in Drive on your phone and see what's still pending, without reading any state file.

## Setup

See [INSTALL.md](./INSTALL.md) for full setup. TL;DR:

```bash
# 1. Clone this repo
git clone https://github.com/Jiplet/audio-to-notes.git ~/audio-to-notes

# 2. Drop the .claude/ folder into your Claude Code workspace
cp -R ~/audio-to-notes/.claude /path/to/your/claude-code-workspace/

# 3. Set your paths (inbox + Obsidian vault target) inside the SKILL.md and the slash command
# 4. Install Python deps for the OpenAI Whisper path
pip install openai requests
brew install ffmpeg

# 5. Add your OpenAI API key
echo "OPENAI_API_KEY=sk-..." > /path/to/.claude/skills/utility-transcribe/.env

# 6. In Claude Code, type /transcribe
```

## Customising

- **Output structure**: edit the section headings in `.claude/skills/utility-transcribe/SKILL.md` (Step 2 — Determine output mode). Add or remove sections to match how you actually take notes.
- **Output destination**: change the path in `SKILL.md` Step 3 to point to wherever your Obsidian vault lives. Could be a Notion folder synced via a plugin, or any markdown-compatible target.
- **Inbox location**: change the path in both `SKILL.md` and `.claude/commands/transcribe.md`. Could be a Dropbox folder, an iCloud folder, anything synced locally.
- **Frontmatter taxonomy**: edit the `area:` enum in `SKILL.md` to match your projects/clients/areas.

## Local-only alternative (mlx-whisper)

If you have an Apple Silicon Mac and want fully local, free transcription:

```bash
uv tool install mlx-whisper
brew install ffmpeg
```

Then use the `/transcribe` slash command — it routes to mlx-whisper directly, no OpenAI API key needed. ~5-10 min for a 60-min audio file on M-series GPU.

## License

MIT. Use it, fork it, change it.
