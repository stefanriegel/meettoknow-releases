# Tester guide

Version: 2026.10.08.5

## Requirements

- macOS 26, Apple Silicon (M1 or newer), 8 GB RAM
- ~1 GB disk for required models; on-device AI: +2.8 GB (< 16 GB RAM) or +4.6 GB (≥ 16 GB)

## Install

1. Download the DMG from [Releases](https://github.com/stefanriegel/meettoknow-releases/releases), drag MeetToKnow to Applications.
2. Gatekeeper warning? Stop and report it — builds are notarized.
3. Allow automatic update checks when asked (second launch). Manual: **MeetToKnow ▸ Check for Updates…**

## Permissions

| Permission | Needed for |
|---|---|
| Microphone | required |
| System Audio Recording | the other side of calls |
| Calendar | meeting titles, reminders (optional) |
| Notifications | reminders, "Call ended", "Record this call?" (optional) |
| Screen Recording | screenshot text (optional, off by default) |
| Accessibility | Zoom speaker names (optional, off by default) |
| Local Network | an AI server on your network |

## AI for summaries

- **On-device:** Gemma 4, download in **Settings ▸ AI** (E4B from 16 GB RAM, E2B below).
- **Own server:** see [server-endpoint.md](server-endpoint.md).

## What's in

- Recording of calls (mic and call audio separate) and in-person meetings; pause/resume; crash recovery
- **Record this call?** — bar at the top right when Teams, Zoom or Webex (apps) use the mic
- **Auto-stop after a call** — 15 s after the call app releases the mic, cut back to the hang-up; **Keep Recording** to continue
- Auto-Record, calendar reminders
- Live text and quick notes while recording
- On-device transcription (Parakeet), speaker labels ("Me", "Speaker 1", …), renaming
- Summaries (templates, meeting language), on-device or server
- Chat with citations — server only for now
- Search, audio import, export (Markdown, bundle), Obsidian export
- Screenshot text in search (off by default)
- Problem report (**Help ▸ Report a Problem…**), signed updates

## Please test

- [ ] Teams/Zoom call ≥ 15 min: suggestion appears, auto-stop after hang-up, nothing after hang-up in the transcript
- [ ] With headphones, without, with AirPods/Bluetooth
- [ ] Call with 3+ remote participants: are they separated?
- [ ] In-person meeting, built-in mic
- [ ] Meeting > 1 h
- [ ] German and English summary; chat question with citation (server)
- [ ] Rename speaker, Re-transcribe, Delete Meeting
- [ ] File ▸ Import Audio…
- [ ] Quit during "Transcribing…", reopen: continues
- [ ] Update via Check for Updates…

## Needs more testing

- Real Teams/Zoom/Webex calls: suggestion over full-screen calls, mute, rejoin, device switch
- German and mixed German/English accuracy and summaries
- Several remote speakers; Zoom speaker names
- 8 GB Macs with on-device AI
- Bluetooth headsets, docks, external mics; sleep or lid closed while recording

## Reporting

- Bugs, ideas, questions: [new issue](https://github.com/stefanriegel/meettoknow-releases/issues/new/choose)
- Problem report: **Help ▸ Report a Problem…** → zip on Desktop. Send privately, not in the issue.
- No meeting content (transcripts, summaries, titles, names) in issues.

## Known limits

- Chat needs a server (on-device chat is on the [roadmap](roadmap.md))
- Speaker labels are numbers; one person can show as two (rename to merge) or two as one
- No Teams names yet; Zoom names only in speaker view
- Safari and FaceTime calls: no suggestion, no auto-stop
- Short German phrases less accurate
- Bluetooth headsets may drop to low quality while recording
- Experimental features (Settings ▸ General): off, not part of the test
- No downgrades: newer versions change the database
