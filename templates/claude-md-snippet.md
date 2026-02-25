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
