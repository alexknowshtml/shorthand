# Review Intents

Scan recent session transcripts and update your Shorthand intent mappings.

## What This Does

1. Reads your current intent mappings from `.claude/data/intent-mappings.json`
2. Scans your 10 most recent session transcripts for usage patterns
3. Identifies gaps: commands you invoke with `/` that don't have natural language triggers
4. Identifies candidates: natural language phrases you use before invoking slash commands
5. Flags conflicts: triggers that could match multiple commands
6. Proposes additions and modifications

## Steps

### 1. Load current mappings

Read `.claude/data/intent-mappings.json` and note all currently mapped commands/skills.

### 2. Scan session transcripts

Session transcripts are JSONL files in `~/.claude/projects/*/`

Scan the 10 most recent files. For each, look for:

**Slash command frequency:**
- Count how often each `/command` appears in user messages
- Note which mapped commands are used most (validates existing mappings)
- Note which unmapped commands are used frequently (candidates for new mappings)

**Natural language -> command patterns:**
- Find cases where user said something conversational, then typed a slash command within the next 3 messages
- These natural language phrases are trigger candidates
- Example: "I should check my email" followed by `/email-overview` -> add "check my email" as a trigger

**Repeated phrases without commands:**
- Phrases the user says often that relate to available commands but never trigger them
- These suggest the user expects natural language to work but it doesn't yet

### 3. Report findings

Present findings in groups:

```
## Current Mapping Health
- [N] intents mapped, [N] commands/skills total
- Most used: /deploy (47 invocations), /email-overview (31 invocations)
- Least used mappings: /some-command (0 natural language triggers found)

## Suggested New Mappings
1. /deploy (used 47 times, no mapping)
   Suggested triggers: "deploy", "ship it", "push to prod"

2. /run-tests (used 23 times, no mapping)
   Suggested triggers: "run tests", "test it"

## Suggested Trigger Additions
- /email-overview: add "inbox status" (said 4 times before invoking)

## Potential Conflicts
- "deploy" could match /deploy or /deploy-staging
  Suggestion: use "deploy to prod" vs "deploy to staging"

## Unused Mappings (consider removing)
- /old-command: mapped but never invoked or triggered in last 10 sessions
```

### 4. Apply approved changes

After the user reviews and approves:
- Add new mappings to `intent-mappings.json`
- Add new triggers to existing mappings
- Remove mappings the user wants to drop
- Note any conflicts that need manual resolution
