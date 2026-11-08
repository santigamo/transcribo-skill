# Transcribo for AI agents

[Transcribo](https://transcribo.es/) turns audio and video files into text with speaker labels, in Spanish and 60 languages in total, up to 4 hours per file. This package connects your agent to the Transcribo MCP server, so you can transcribe a recording and keep working with the text in the same conversation: minutes, summaries, quotes or translations.

Your first 30 minutes are free. After that it costs [2 € per hour of audio](https://transcribo.es/precios/), deducted by the minute.

## Install as a skill

```bash
npx skills add santigamo/transcribo-skill --skill transcribo
```

## Install in Hermes Agent

```bash
hermes plugins install transcribo
```

The first time a Transcribo tool loads, a browser opens so you can sign in at transcribo.es, with email or Google.

### What this plugin does

- **Network.** Every tool call goes to Transcribo's hosted server at `api.transcribo.es` and acts on your Transcribo account. The tool set comes from that server at runtime and is not covered by the pinned commit.
- **Your audio.** Files are uploaded to Transcribo's servers to be transcribed. The transcript and a compressed copy of the audio stay in your account until you delete them. See the [privacy policy](https://transcribo.es/privacidad/).
- **Balance.** `start_transcription` and `start_uploaded_transcription` use audio time from your Transcribo balance. Failed transcriptions are not charged.
- **Email.** The account owner may get an email when a job finishes or fails, unless they turned those notifications off.
- **Shell commands.** To send a file from your computer, the skill has the agent run `curl` to upload that file to a presigned URL.
- **Credentials.** There are none in this package or in Hermes's config. Sign-in is OAuth 2.1 with PKCE. It runs the pinned `mcp-remote@0.1.49` through `npx` (Node.js 20.18.1+ and npm required), which opens a browser and a local callback port on first use and stores tokens under `~/.mcp-auth`.
- **Approvals.** The package marks the server as untrusted, so Hermes asks before every tool the server does not mark read-only and refuses them in unattended runs.
- There is no code, binary, telemetry or self-updater in this package.

## Other agents

Claude Code:

```bash
claude mcp add --transport http transcribo https://api.transcribo.es/mcp
```

Any assistant that supports remote MCP connectors with OAuth can use `https://api.transcribo.es/mcp`. See the full [guide for Claude, ChatGPT and Cursor](https://transcribo.es/blog/transcribir-audio-con-claude-chatgpt-cursor-mcp/).

For audio that must not leave your device, use the [free version in the browser](https://transcribo.es/), which needs no account and handles up to 30 minutes per file.

Support: soporte@transcribo.es

## License

MIT
