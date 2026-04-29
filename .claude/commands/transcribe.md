---
description: Scan the voice-memo inbox and transcribe new audio files locally with mlx-whisper, or via the utility-transcribe skill for cloud transcription
---

# /transcribe — scan inbox and transcribe new files

Scan the voice-memo inbox, identify files that haven't been transcribed yet, and transcribe them. Two engines available — pick based on what's installed.

## Inbox to scan

`<INBOX_PATH>`

> Replace `<INBOX_PATH>` with your local Drive Desktop mirror path before using.

Scan only the **root** of the inbox. The `processed/` subfolder holds completed work — never reprocess from there. Ignore `.DS_Store` and other dotfiles.

## "New file" detection

A file in the inbox is **new** when BOTH conditions hold:

1. It is in the inbox root (not in `processed/`)
2. No `.txt` (or `.md`) file with the same basename sits next to it

The companion file is the primary marker. Moving the audio to `processed/` after success is the secondary marker. Both together let the inbox stay self-evident — you can glance at the folder and see what's pending without reading any state file.

Audio extensions to consider: `.m4a` `.mp3` `.mp4` `.wav` `.webm` `.ogg` `.opus` `.aac` `.flac` `.mpeg` `.mpga`

## Modes

| Trigger | Behaviour |
|---|---|
| `/transcribe` (no args) | **Scan + confirm.** List pending files with size and duration estimate; wait for "go" before transcribing |
| `/transcribe --auto` | **Scan + auto-process.** No confirmation — chew through the inbox |
| `/transcribe <path>` | **Single-file.** Transcribe that specific file, ignore the inbox |
| `/transcribe --wiki` | After transcription, also invoke the `utility-transcribe` skill to write structured notes to your Obsidian vault |

`--auto` and `--wiki` can combine.

## Engine selection

Check what's available, prefer local if both are installed:

| Engine | When to use | Command |
|---|---|---|
| `mlx_whisper` | Apple Silicon Mac, available on PATH | Local, free, fast (~1/12 of audio duration) |
| `utility-transcribe` skill | OpenAI API key configured, any platform | Cloud, ~$0.006/min, more robust on edge cases |

Detect with `which mlx_whisper`. If absent, fall back to the skill.

## Steps

### 1. Enumerate pending files

```bash
INBOX="<INBOX_PATH>"

# List audio files in root (not in processed/), where no .txt or .md companion exists.
# Use find -maxdepth 1 to stay out of subfolders. Filter out dotfiles.
```

For each candidate, report: filename, size (MB), duration (use `ffprobe -v error -show_entries format=duration`).

If the inbox has no pending audio, say "Inbox empty — nothing to transcribe" and stop.

### 2. Confirm (unless `--auto`)

Show the pending list. Ask for confirmation. Warn if total estimated runtime is large.

### 3. Transcribe each file

**If using mlx-whisper (local):**

```bash
mlx_whisper "<audio_path>" \
  --model mlx-community/whisper-large-v3-turbo \
  --output-dir "<inbox_dir>" \
  --output-format txt \
  --language en
```

Output lands as `<basename>.txt` next to the original file (the companion that signals "done").

**If using the skill (cloud):**

Invoke the `utility-transcribe` skill, which handles 3GPP detection, oversized files, and chunking automatically. The skill writes a structured markdown note directly to your Obsidian vault — no separate `.txt` step.

### 4. Move audio to `processed/`

After a successful transcription:

```bash
mkdir -p "<inbox_dir>/processed"
mv "<audio_path>" "<inbox_dir>/processed/"
```

**Convention**: move only the audio file. Keep the `.txt` (or wiki note) where it lands so transcripts are visible at a glance.

### 5. Report back

```
Transcribed:
- meeting recording.m4a (62 min → 10,634 words) → audio moved to processed/
- 2026-04-29 site visit.m4a (18 min → 3,200 words) → audio moved to processed/

Skipped:
- duplicate (1).m4a (companion .txt already exists)
```

### 6. (Optional) Wiki compile

If `--wiki` was passed, OR if the user asks to log it, invoke the `utility-transcribe` skill on each new transcript. The skill produces a structured markdown note (BLUF, Actions, Discussion, Risks, Stakeholder Pulse) and writes it to your Obsidian vault.

## Tooling assumptions

- `mlx_whisper` (optional): install via `uv tool install mlx-whisper` (Apple Silicon only — uses Apple GPU via MLX).
- `ffmpeg` / `ffprobe`: install via `brew install ffmpeg`.
- Drive Desktop is syncing the inbox path locally — transcription writes to the local copy and Drive Desktop syncs back up.
- For the cloud path, see `.claude/skills/utility-transcribe/SKILL.md` for setup.

## Rules

- **Never reprocess** files already in `processed/` — the move is the one-way gate.
- **Never delete** original audio. Move only.
- **Never overwrite** an existing companion file — if one exists, the file is not new and is skipped.
- **Quiet mode** if inbox empty — one line, no ceremony.
- **Background long jobs** — for queues that will exceed 10 min total runtime, run in the background and poll. Don't block the chat.
- **Cost / privacy:** mlx-whisper is fully local — nothing leaves the Mac. The cloud path sends audio to OpenAI's API. Pick the right one for the content.
