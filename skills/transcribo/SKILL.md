---
name: transcribo
description: Transcribe audio and video files to text with speaker labels using Transcribo (transcribo.es), in Spanish and 60 languages in total. Use when the user wants to transcribe or get the text of a recording, interview, meeting, class, podcast, thesis interview or WhatsApp voice note, up to 4 hours long, or says "transcribir", "pasar audio a texto", "transcribe este audio" or "separar hablantes".
---

# Transcribo

Turn long recordings into text with speaker labels through the [Transcribo](https://transcribo.es/) MCP server, then keep working with the text in the same conversation: minutes, summaries, quotes or translations.

## Setup

Check whether the Transcribo tools (`start_transcription`, `create_upload_link`, `get_transcription`…) are available. If they are not, connect the MCP server at `https://api.transcribo.es/mcp`.

**Claude Code**

```bash
claude mcp add --transport http transcribo https://api.transcribo.es/mcp
```

Then ask the user to type `/mcp`, choose **transcribo** and sign in.

**Cursor**: add this to `mcp.json`, and sign in when Cursor asks.

```json
{
  "mcpServers": {
    "transcribo": {
      "url": "https://api.transcribo.es/mcp"
    }
  }
}
```

**Claude (web and desktop), ChatGPT, Grok and other assistants**: the user adds a custom connector with that URL and signs in.

Sign-in is OAuth at transcribo.es, with email or Google. Only the user can complete it. Never ask for a password or token. An account can be created during sign-in.

## Choose how to send the file

| Where the file is | Tools | Max size |
| --- | --- | --- |
| On the user's computer, and you can run shell commands there | `create_upload_link`, upload, `start_uploaded_transcription` | 2 GB |
| Public link: direct download, or Google Drive or Dropbox shared with anyone who has the link | `start_transcription` with `url` | 500 MB |
| Attached in the conversation (ChatGPT) | `start_transcription` with `audio` | 500 MB |

All routes accept up to 4 hours of audio in mp3, m4a, wav, ogg/opus, flac, mp4, webm, mov or mkv. Video files are transcribed from their audio track.

If you cannot reach the file by any of these routes, ask the user for a public link. Chat attachments do not reach the connector in Claude.

## Workflow

### Local file

1. Call `create_upload_link` with `file_name` (including the extension) and, only if you know it, `language`. This spends no balance.
2. Upload only the file the user named, quoting its path:

   ```bash
   curl --fail -sS -T "<path>" "<upload_url>"
   ```

   The upload URL is valid for 15 minutes.
3. Call `start_uploaded_transcription` with the `job_id`. If the file has not arrived yet, nothing changes and you can call it again.
4. Poll `get_transcription` as described below.

### Public link or attachment

1. Call `start_transcription` with `url` or `audio` and, only if you know it, `language`.
2. Poll `get_transcription` as described below.

### Polling

Call `get_transcription` with the `job_id` every 10 to 20 seconds until it reports `completed` or returns an error. An hour of audio usually takes a few minutes. If you have to stop first, give the user the `job_id` and offer to check again later. The job keeps running.

## Results

- The text comes with speaker labels: `Hablante 1`, `Hablante 2`…, in the language of the audio. Omit `language` to detect it automatically.
- Up to 60,000 characters are returned, about an hour of conversation. Longer transcripts return the beginning, `truncated: true` and a link to the full text, so tell the user when the result is partial and do not summarize it as if it were complete.
- The transcript is saved in the user's account at transcribo.es, where they can rename speakers and download Word, TXT or SRT.
- When the user asks for a file, write the returned text to it without rewriting or correcting it.

## Rules

- **Starting a job spends balance.** `start_transcription` creates a new job on every call, so never call it again for a file that is already processing. Poll the existing `job_id` instead.
- Failed transcriptions are not charged.
- YouTube links, streaming sites and web pages do not work, and a link must report its file size. Ask for the file itself instead.
- There are no tools to list or delete transcripts. The user manages them at transcribo.es.
- Use `get_connected_account` when the user asks which account is connected.

## Balance and prices

The first 30 minutes are free for new accounts once the email is confirmed. After that, balance is deducted by the minute:

- 2 € per hour of audio
- 5 hours for 9 €, or 15 hours for 24 €
- A subscription with 20 hours per month for 15 €

Balance is the same on the web and through the assistant. One-off hours do not expire, and subscription hours renew each month. Balance is bought at [transcribo.es/precios](https://transcribo.es/precios/), never through these tools. If the balance is too low, tell the user and link that page.

## Privacy

Audio sent through these tools is uploaded to Transcribo's servers. The transcript and a compressed copy of the audio stay in the user's account until they delete them. The account owner may get an email when a job finishes or fails, unless they turned those notifications off.

Before uploading a recording that looks sensitive, make sure the user is fine with that. For audio that must not leave the device, point them to the free version at [transcribo.es](https://transcribo.es/), which runs in the browser, needs no account and handles up to 30 minutes per file.

If the user shared a public link only for this, remind them they can remove public access once the transcript is ready.

Support: soporte@transcribo.es
