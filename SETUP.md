# Shorthand Setup Instructions

> **These instructions are written for Claude Code to follow.** When a user asks you to "set up Shorthand," read this file and execute each step.

## Prerequisites

Confirm these exist before proceeding:
- A `.claude/` directory in the project root (this is a Claude Code project)
- A `CLAUDE.md` file in the project root

If either is missing, tell the user this project needs to be set up for Claude Code first.

## Step 1: Scan existing commands and skills

Read the user's available commands and skills:

```
Glob pattern: .claude/commands/**/*.md
Glob pattern: .claude/skills/**/SKILL.md
```

For each command/skill found:
1. Read the file to understand what it does
2. Note the invocation name (filename or skill name)
3. Categorize by domain (email, git, deployment, meetings, etc.)

Store this inventory — you'll use it in Step 3.

## Step 2: Scan session transcripts for usage patterns

Session transcripts are stored as JSONL files in:
```
~/.claude/projects/*/
```

Each line is a JSON object. Look for `user` role messages.

**What to find:**

1. **Direct slash command usage**: Lines where the user typed `/command-name`. Count frequency of each command.

2. **Natural language before slash commands**: Cases where the user said something in natural language, then in a subsequent message (within 3 messages) invoked a slash command. These are intent candidates.

   Example pattern:
   - User: "I need to book the conference room for Friday"
   - User (2 messages later): "/book-room"
   - This suggests "book the conference room" should map to `/book-room`

3. **Repeated natural language patterns**: Phrases the user says frequently that relate to available commands but never invoke them directly.

**Scan the 10 most recent transcript files.** This gives enough signal without taking too long.

**Output format for each finding:**
```
Command: /book-room (used 12 times)
Natural language attempts:
  - "book the room" (3 times)
  - "reserve the boardroom" (2 times)
  - "I need a room for Thursday" (1 time)
```

## Step 3: Generate proposed intent mappings

Using the command inventory (Step 1) and usage patterns (Step 2):

1. **Rank commands by usage frequency** — most-used commands get mapped first
2. **Propose trigger phrases** based on natural language found in transcripts
3. **Add obvious triggers** for commands whose names clearly suggest natural language (e.g., `/deploy` -> "deploy", "ship it")
4. **Group by domain** and flag potential conflicts:
   - If two commands could match similar triggers, note the conflict
   - Suggest disambiguation strategies (add context words, use specific verbs)

Present the proposed mappings to the user in groups:

```
## Email (3 commands)
- /email-overview -> "check email", "inbox", "show inbox"
- /email-filter -> "filter email", "run filters"
- /email-send -> "send email", "compose email"

## Meetings (2 commands)
- /book-room -> "book a room", "reserve the boardroom"
- /prep-meeting -> "prep for meeting", "meeting prep"

## Potential conflicts:
- "book" could match /book-room or /book-flight — suggest using "book a room" vs "book a flight"
```

**Ask the user to approve, modify, or remove mappings before proceeding.**

## Step 4: Create the intent mappings file

Create `.claude/data/intent-mappings.json` using the approved mappings.

If `.claude/data/` doesn't exist, create it.

**Format:**

```json
{
  "intent_name": {
    "triggers": ["trigger phrase 1", "trigger phrase 2"],
    "command": "/command-name",
    "description": "What this intent does"
  }
}
```

For skill-based intents:
```json
{
  "intent_name": {
    "triggers": ["trigger phrase 1"],
    "skill": "skill-name",
    "description": "What this intent does"
  }
}
```

**Rules for trigger phrases:**
- Lowercase
- No punctuation
- Use the phrases actually found in transcripts when possible
- Include 3-6 triggers per intent (more for high-frequency commands)
- Avoid single-word triggers unless very specific (they cause false positives)

## Step 5: Add the routing snippet to CLAUDE.md

Add the following to the user's `CLAUDE.md` file. Find a logical place — after any existing "Commands" or "Skills" section, or at the end if no obvious location.

```markdown
## Shorthand Intent Detection

When the user's plain language matches triggers in `.claude/data/intent-mappings.json`, automatically invoke the mapped command or skill.

**How to match:**
1. Load intent mappings from `.claude/data/intent-mappings.json`
2. Compare user's message against all triggers (case-insensitive, partial match)
3. If match found:
   - If `command` is specified: invoke using the Skill tool with the command name
   - If `skill` is specified: invoke using the Skill tool with the skill name
4. If multiple intents could match, prefer the most specific trigger (longest match)
5. If ambiguous, ask the user which they meant

**Review mappings:** Run `/review-intents` to scan recent sessions and update mappings.
```

## Step 6: Install the review-intents command

Create `.claude/commands/review-intents.md` with the contents from the `templates/review-intents.md` file in this repo.

If `.claude/commands/` doesn't exist, create it.

## Step 7: Verify installation

1. Confirm `.claude/data/intent-mappings.json` exists and is valid JSON
2. Confirm `CLAUDE.md` contains the Shorthand routing snippet
3. Confirm `.claude/commands/review-intents.md` exists

Tell the user:

```
Shorthand is set up. You have [N] intent mappings across [N] categories.

Try saying one of your mapped phrases naturally — like "[example from their mappings]" — instead of typing the slash command.

Run /review-intents anytime to scan recent sessions and update your mappings.
```

## Troubleshooting

**No session transcripts found:**
- Transcripts are stored after sessions end. If this is a new project, there won't be any yet.
- Skip the transcript scanning step and propose mappings based on command/skill names only.
- Tell the user to run `/review-intents` after a week of use to mine their actual patterns.

**No commands or skills found:**
- The project doesn't have custom commands or skills yet.
- Tell the user Shorthand works best once they have commands to map. Suggest they create a few commands first, use them for a while, then come back to set up Shorthand.

**CLAUDE.md is very large:**
- Add the routing snippet as a concise reference and point to the JSON file.
- Don't duplicate the full mappings in CLAUDE.md.
