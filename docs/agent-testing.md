# Agent testing

How an AI agent (Claude Code, Codex, …) drives MeetToKnow on a tester's Mac to test features and file bugs.
Version: 2026.10.08.5

## Rules

- Test audio only from `say`, own test files, or calls whose participants agreed.
- No meeting content (transcripts, summaries, titles, names, screenshots) in issues or any output leaving the Mac.
- Database read-only. Don't delete meetings you didn't create.
- Problem report zip: never attach to a public issue.
- Ask the human before sending data off the Mac or changing system settings.

## Setup

- MeetToKnow in `/Applications`, Setup Assistant done, mic allowed.
- Agent's terminal allowed in **System Settings ▸ Privacy & Security ▸ Accessibility**.
- Version: `defaults read /Applications/MeetToKnow.app/Contents/Info CFBundleShortVersionString`

## Menus

| Menu | Item | Shortcut |
|---|---|---|
| Recording | Start Recording / Stop Recording | ⇧⌘R |
| Recording | Pause Recording / Resume Recording | ⇧⌘P |
| File | Import Audio… | ⌥⌘I |
| File | Import Bundle… | ⇧⌘I |
| MeetToKnow | About, Check for Updates…, Settings… | ⌘, |
| Help | Setup Assistant…, Report a Problem… | |

Menu bar icon: Start/Stop/Pause Recording, Keep Recording, Auto-Record Meetings, Suggest Recording When a Call Starts, Capture Screenshots, Open MeetToKnow, Settings…, Quit.

```bash
osascript -e 'tell application "MeetToKnow" to activate' \
  -e 'tell application "System Events" to tell process "MeetToKnow" to click menu item "Start Recording" of menu "Recording" of menu bar 1'
```

## Accessibility identifiers

| Area | AXIdentifier |
|---|---|
| Recording window | `companion-pause`, `companion-stop`, `companion-keep-recording` |
| Record this call? | `suggest-record`, `suggest-not-now` |
| Meeting | `record-toggle`, `pause-toggle`, `meeting-title`, `meeting-progress`, `player-toggle`, `playhead`, `transcript-time-<ms>`, `citation-<ms>` |
| Summary | `summary-regenerate`, `summary-edit`, `summary-save`, `summary-cancel`, `summary-history`, `summary-find`, `summary-writing`, `summary-stale` |
| Chat | `chat-composer`, `chat-stop`, `chat-retry`, `chat-clear` |
| Import | `import-status`, `import-cancel` |
| Settings | `suggest-recording`, `capture-screenshots`, `name-speakers-from-call-app`, `server-base-url`, `notify-recording-notice`, `notify-meeting-reminders`, `notify-reminder-lead`, `model-delete-<name>`, `permission-allow-<pane>` |
| Setup | `setup-continue`, `setup-back`, `setup-skip`, `setup-finish`, `setup-start-test`, `setup-step-<n>` |
| Help | `problem-report-status` |

Press by identifier (also works for floating panels):

```bash
osascript <<'EOF'
tell application "System Events" to tell process "MeetToKnow"
  repeat with w in windows
    repeat with e in (entire contents of w)
      try
        if value of attribute "AXIdentifier" of e is "suggest-record" then
          perform action "AXPress" of e
          return "pressed"
        end if
      end try
    end repeat
  end repeat
end tell
EOF
```

## Test audio

- Room: record, then `say -v Anna "Guten Morgen, wir besprechen das Budget."` / `say -v Samantha "…"` (speakers on, no headphones).
- Import: File ▸ Import Audio…, ⇧⌘G, path, Return, Return.
- Calls: real Teams/Zoom/Webex call. Browser calls and FaceTime don't trigger suggestion or auto-stop.

## Logs

`~/Library/Logs/MeetToKnow/*.log` — no meeting content.

| Line | Meaning |
|---|---|
| `recording started:` / `recording stopped: N frames (S s)` | recording began / ended |
| `suggesting a recording` / `call suggestion: record` / `not now` | suggestion shown / answered |
| `call ended (… let go of the mic); recording stops in 15 s unless kept` | countdown started |
| `recording cut where the call ended: kept A of B frames` | post-call part removed |
| `the user keeps recording` | Keep Recording pressed |
| `enqueued <id>` / `transcribed <id>` | processing |
| `failed`, `error` | problem — quote the line |

## Database (read-only)

```bash
DB="$HOME/Library/Application Support/MeetToKnow/db.sqlite"
sqlite3 -readonly "$DB" "select state, processingStep, durationMs from meeting order by startedAt desc limit 1"
sqlite3 -readonly "$DB" "select count(*) from transcriptSegment where meetingId=(select id from meeting order by startedAt desc limit 1)"
```

`state`: `recording` → `processing` → `ready` | `failed`. Read transcript text locally only.

Settings: `defaults read me.riegel.meettoknow <key>` — `suggestRecording`, `captureScreenshots`, `nameSpeakersFromCallApp`, `autoRecordArmed`, `localModelId`, `llmProvider`.

## Recipes

| Test | Steps | Pass |
|---|---|---|
| Smoke | Start, `say` 2 sentences, Stop, wait for `ready` | segments > 0, no `error` |
| Pause | Start, say A, ⇧⌘P, say B, ⇧⌘P, say C, Stop | A and C in transcript, B not |
| Call end | Real call, `suggest-record`, hang up, keep talking 30 s | countdown + cut in log; nothing after hang-up |
| Keep Recording | As call end, press `companion-keep-recording` | no cut; recording continues |
| Problem report | Help ▸ Report a Problem… | zip on Desktop (keep private) |

## Filing

```bash
gh issue create --repo stefanriegel/meettoknow-releases --label bug \
  --title "…" \
  --body "Version: … · macOS … · Mac, RAM
Steps: 1. … 2. …
Expected: …
Actual: …
Log lines: …
Problem report sent privately: yes/no"
```

Ideas: label `enhancement`. One topic per issue; search first.
