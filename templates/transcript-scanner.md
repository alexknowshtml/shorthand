# Transcript Scanner

> **Instructions for Claude to mine session transcripts for intent mapping candidates.**

## Overview

This scanner reads Claude Code session transcripts (JSONL files) and identifies patterns where users express intent in natural language before or instead of using slash commands.

## Where transcripts live

```
~/.claude/projects/*/
```

Each directory corresponds to a Claude Code project. Files are `.jsonl` format, one JSON object per line.

## JSONL structure

Each line contains a message object. The relevant fields are:

```json
{
  "role": "user",        // or "assistant"
  "message": "the text content",
  "timestamp": "..."
}
```

Focus only on `role: "user"` messages.

## What to extract

### 1. Slash command frequency

Count every message that starts with `/` (a direct command invocation).

```
/deploy -> 47 times
/email-overview -> 31 times
/book-room -> 12 times
```

### 2. Natural language -> command pairs

Look for this pattern within a sliding window of 3 consecutive user messages:

```
Message N:   Natural language (no slash command)
Message N+1: (optional: another natural language message or assistant response)
Message N+2: /some-command
```

The natural language message is a trigger candidate for that command.

**Examples:**
```
"I need to check my email"       -> (next command) /email-overview
"book the boardroom for Friday"  -> (next command) /book-room
"let's deploy this"              -> (next command) /deploy
```

**Collect unique phrases and count occurrences.**

### 3. Orphan natural language

Phrases that look like command intent but never result in a slash command:
- Short imperative sentences ("deploy this", "run the tests")
- Phrases containing command-like verbs ("check", "run", "show", "create", "update", "delete")
- These might indicate missing commands OR commands the user invokes differently

## Output format

Return a structured summary:

```
## Command Usage (Top 20)
1. /deploy - 47 invocations
2. /email-overview - 31 invocations
...

## Natural Language -> Command Pairs
/deploy:
  - "deploy this" (5x)
  - "ship it" (3x)
  - "push to production" (2x)

/book-room:
  - "book the boardroom" (3x)
  - "reserve a room" (2x)

## Orphan Intent Phrases (no matching command found)
  - "check the logs" (4x) - no /logs command exists
  - "what's the status" (3x) - could map to /status or /overview

## Recommended Mappings (ranked by impact)
1. /deploy - HIGH (47 uses, 10 natural language attempts)
2. /book-room - MEDIUM (12 uses, 5 natural language attempts)
3. /email-overview - LOW (31 uses, but only 2 NL attempts - users seem comfortable with /)
```

## Performance notes

- Only scan the 10 most recent transcript files per project
- Skip files larger than 10MB (likely very long sessions that would slow scanning)
- Stop scanning if you've found 50+ command invocations — that's enough signal
