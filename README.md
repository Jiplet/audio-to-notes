# audio-to-notes

Turn meeting recordings sitting in Google Drive into clean, structured notes using Claude.

No scripts. No pipelines. Just a connector and a prompt.

## What you need

- A [claude.ai](https://claude.ai) account (free works, Pro is faster)
- A Google account with the audio file in Drive (m4a, mp3, wav, mp4 all work)
- Five minutes for first-time setup

## Setup (one-time, ~2 min)

1. Open [claude.ai](https://claude.ai)
2. Click your profile (bottom-left) → **Settings** → **Connectors**
3. Find **Google Drive** → **Connect** → sign in and approve

That's it. Claude can now read files from your Drive.

## Usage (every time, ~30 sec)

1. Start a new chat in claude.ai
2. Click the **+** (attach) button → **Add from Google Drive** → pick your audio file
3. Paste one of the prompts from [`prompts/`](./prompts/) into the chat
4. Send

Claude transcribes the audio and produces structured notes in one go.

## Which prompt to use

| Recording type | Prompt |
|---|---|
| Team meeting, project meeting, standup | [`meeting-summary.md`](./prompts/meeting-summary.md) |
| 1:1 with a stakeholder, manager, or report | [`stakeholder-1on1.md`](./prompts/stakeholder-1on1.md) |
| Customer interview, user research, discovery call | [`interview-notes.md`](./prompts/interview-notes.md) |

Open the prompt file, copy the whole thing, paste into Claude.

## Tips

- **Long recordings (>1 hour)**: Claude handles them, but the response is faster if you tell it which sections matter most. Add a line like *"Focus on the second half where we discussed pricing."*
- **Multiple speakers**: Claude will try to label them by voice. If you know the names, list them upfront: *"Speakers are Sam, Jess, and Priya."*
- **Bad audio**: If transcription quality is low, Claude will flag it. Don't trust quotes from those sections.
- **Sensitive content**: claude.ai keeps your data private by default and does not train on it. Still, use judgement for highly confidential recordings.

## Customising the prompts

Each prompt is a plain markdown file. Edit the section headings, tone, or output format to match how you actually take notes. Save your version, share it back here as a PR if it's good.

## Troubleshooting

| Problem | Fix |
|---|---|
| Can't see Google Drive in the attach menu | Connector setup didn't complete. Redo step 3 of Setup. |
| Claude says it can't open the file | Some legacy formats (.amr, .opus) need converting first. Use [CloudConvert](https://cloudconvert.com) to make an mp3. |
| Output is too long / too short | Add *"Keep it under 300 words"* or *"Be exhaustive, no summary"* to the prompt. |
| Speaker labels are wrong | Tell Claude the correct names and ask it to redo the labelling. |

## License

MIT. Use it, fork it, change it.
