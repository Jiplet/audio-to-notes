# Install Guide

Full setup for the automated voice-memo-to-Obsidian pipeline.

## What you'll have when you're done

- A Google Drive folder you can drop audio files into (or have your phone auto-sync to)
- The same folder mirrored on your Mac via Drive Desktop
- A `/transcribe` slash command in Claude Code that processes new files
- Each transcription written as a structured markdown note in your Obsidian vault
- Processed audio moved to a `processed/` subfolder so the inbox always shows pending work

## Prerequisites

| Thing | Why | How |
|---|---|---|
| Mac (Apple Silicon recommended) | Local transcription path needs M-series; cloud path works on any Mac | — |
| [Claude Code](https://claude.com/claude-code) | Runs the slash command and skill | `npm install -g @anthropic-ai/claude-code` or download |
| [Google Drive Desktop](https://www.google.com/drive/download/) | Mirrors the inbox folder locally so transcription can read it | Install + sign in |
| [Obsidian](https://obsidian.md) | Output destination | Install + create a vault |
| Python 3.10+ | Runs the transcription script | Usually already on macOS, or `brew install python` |
| `ffmpeg` | Handles oversized files and Samsung's 3GPP wrapper | `brew install ffmpeg` |
| OpenAI API key (cloud path) | Calls Whisper API | [platform.openai.com](https://platform.openai.com) → API keys |
| OR mlx-whisper (local path) | Free, fully local transcription on Apple GPU | `uv tool install mlx-whisper` |

## Step 1: Set up the Google Drive inbox

1. Create a folder in Google Drive — name it whatever you want. Common choice: `99 Voice memos/`

2. **Make the folder available offline.** This is critical. If the folder is "stream-only," files won't be on disk when Claude tries to read them, transcription will hang or fail, and Drive will trigger a download every time. You need the actual bytes locally.

   You have two ways to do this:

   **Option A — Mirror the whole drive (simplest)**
   - Click the Drive Desktop icon in your Mac menu bar → ⚙ (Settings) → **Preferences**
   - Click your Google account in the left sidebar → **My Drive** section
   - Select **Mirror files** (not "Stream files")
   - Click **Save**. Drive will start downloading all your Drive content to disk. Wait for the initial sync to complete (can be hours if your Drive is large).
   - Pro: every file is always offline. Con: takes up local disk space equal to your full Drive.

   **Option B — Stream mode + per-folder offline (less disk usage)**
   - Keep Drive Desktop in **Stream files** mode (the default)
   - In Finder, navigate to: `/Users/<you>/Library/CloudStorage/GoogleDrive-<email>/My Drive/`
   - Find your inbox folder → right-click → **Make available offline** (or in some Drive versions: → **Offline access** → **Available offline**)
   - The icon next to the folder will change to a green checkmark when fully synced
   - Pro: only this folder takes disk space. Con: must repeat per-folder if you have multiple inboxes.

3. **Verify it's actually offline.** Run this in Terminal to confirm files are on disk and not just stubs:
   ```bash
   # Replace with your path
   INBOX="/Users/<you>/Library/CloudStorage/GoogleDrive-<email>/My Drive/99 Voice memos"
   ls -la "$INBOX"  # files should show real sizes, not 0 bytes
   du -sh "$INBOX"  # should match Drive's reported size, not be ~0
   ```

   If files show 0 bytes or near-zero `du`, offline isn't actually on yet. Wait for sync, or repeat Step 1.2.

4. Note the local path. It'll look something like:
   ```
   /Users/<you>/Library/CloudStorage/GoogleDrive-<email>/My Drive/99 Voice memos
   ```

5. (Optional) Set up auto-sync from your phone:
   - **Android**: install [Autosync](https://play.google.com/store/apps/details?id=com.ttxapps.autosync) → point at this folder
   - **iOS**: Voice Memos → Settings → Sync to iCloud, then use a shortcut to copy to Drive; or record straight into the Drive iOS app

6. (Optional but recommended) Tell macOS not to evict these files from disk under storage pressure: System Settings → General → Storage → toggle **Optimize Mac Storage** to **off** (or accept that files may need re-download if your disk fills up).

## Step 2: Set up the Obsidian vault

1. Create a vault, or use an existing one.
2. Inside the vault, create a folder where voice notes will land. Common: `99-Voice notes/` (any name works).
3. Note the absolute path to that folder.

## Step 3: Drop the .claude/ folder into your workspace

Claude Code looks for slash commands in `.claude/commands/` and skills in `.claude/skills/` relative to the working directory you launch it from.

```bash
# Pick a directory you want to use as your "voice-notes workspace"
# This could be your Obsidian vault root, or a separate workspace folder
cd /path/to/your/workspace

# Copy the slash command + skill from this repo
cp -R /path/to/audio-to-notes/.claude .
```

Verify the structure:
```
your-workspace/
├── .claude/
│   ├── commands/
│   │   └── transcribe.md
│   └── skills/
│       └── utility-transcribe/
│           ├── SKILL.md
│           ├── .env.example
│           └── scripts/
│               └── transcribe.py
```

## Step 4: Configure the paths

Two files have placeholder paths you need to replace with yours:

### `.claude/skills/utility-transcribe/SKILL.md`
Find and replace these tokens:

| Token | Replace with |
|---|---|
| `<INBOX_PATH>` | Your local Drive Desktop path from Step 1 |
| `<OBSIDIAN_VAULT_PATH>` | Your vault path |
| `<VOICE_NOTES_FOLDER>` | The subfolder inside the vault from Step 2 (e.g. `99-Voice notes`) |
| `<AREA_ENUM>` | Your project/client tags, comma-separated. Example: `ProjectAlpha,ProjectBeta,Personal,General` |

### `.claude/commands/transcribe.md`
Same `<INBOX_PATH>` token. Replace it.

## Step 5: Install Python dependencies (cloud path)

```bash
# Use a venv if you prefer
pip install openai requests
```

Or, if you'd rather isolate it:
```bash
python3 -m venv ~/.venvs/audio-to-notes
source ~/.venvs/audio-to-notes/bin/activate
pip install openai requests
```

If you use a venv, update the `transcribe.py` invocation in `SKILL.md` Step 1 to source it first.

## Step 6: Add your OpenAI API key

```bash
cd /path/to/your/workspace/.claude/skills/utility-transcribe
cp .env.example .env
# Edit .env — paste your key after OPENAI_API_KEY=
```

The `.env` file is gitignored. Don't commit it.

## Step 7: (Optional) Set up mlx-whisper for free local transcription

Apple Silicon only.

```bash
uv tool install mlx-whisper
# First run downloads the model (~3GB) — be on wifi
mlx_whisper --help
```

The `/transcribe` slash command will use this if available.

## Step 8: Try it

1. Drop an audio file into your Drive inbox folder.
2. Wait for Drive Desktop to sync it locally (usually 5-30 seconds).
3. In Claude Code, in your workspace: type `/transcribe`
4. Claude scans the inbox, lists pending files, asks for confirmation, then transcribes.
5. Open Obsidian — you should see a new file in your voice notes folder.
6. Check Drive — the audio file should now be in `<inbox>/processed/`.

## Troubleshooting

| Problem | Fix |
|---|---|
| `/transcribe` not recognised | Make sure you're in the workspace where `.claude/commands/transcribe.md` lives. `pwd` should show that workspace. |
| "OPENAI_API_KEY not set" | Check `.claude/skills/utility-transcribe/.env` exists and has the key on a single line. |
| "File not found" but Drive shows it | Drive Desktop hasn't synced yet, or the folder is set to "stream" not "mirror". Wait, or change Drive Desktop preferences. |
| Whisper rejects a Samsung `.m4a` | Already handled — the script auto-detects the 3GPP wrapper and re-encodes. If it's still failing, check `ffmpeg` is on PATH (`which ffmpeg`). |
| Transcript is in the wrong language | Add `--language en` (or your code) to the `client.audio.transcriptions.create()` call in `transcribe.py`. |
| Output structure isn't what I want | Edit the section headings in `SKILL.md` Step 2. The skill is the prompt — change it. |

## Privacy note

- **Cloud path (OpenAI Whisper)**: audio is sent to OpenAI's API. Per their terms, API audio is not used for training and is deleted after 30 days. Don't use this for genuinely confidential recordings.
- **Local path (mlx-whisper)**: nothing leaves your Mac. Use this for sensitive recordings.
