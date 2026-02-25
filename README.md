# Shorthand

**Teach Claude your shorthand.**

Shorthand scans your Claude Code commands and skills, mines your session history, and builds a natural language routing layer — so you stop typing slash commands and start talking.

Instead of remembering `/book-room` when you want to reserve a conference room, just say "book the boardroom for Thursday." Shorthand maps your natural language to the right command automatically.

## How It Works

Shorthand adds two things to your Claude Code project:

1. **A routing snippet** in your `CLAUDE.md` that tells Claude to check intent mappings before processing each message
2. **A JSON file** (`intent-mappings.json`) that maps natural language triggers to your existing commands and skills

When you type "check my email," Claude matches it against your mappings and invokes `/email-overview` — without you ever typing a slash.

## Quick Start

Open Claude Code in your project and say:

```
Set up Shorthand from https://github.com/alexknowshtml/shorthand
```

Claude will read the [SETUP.md](SETUP.md) in this repo and walk you through everything:

1. Scan your `.claude/commands/` and `.claude/skills/` directories
2. Mine your recent session transcripts for usage patterns
3. Propose a ranked list of intent mappings (most-used commands first)
4. Flag potential conflicts between similar triggers
5. Install the routing snippet and mappings file

No cloning. No npm install. No dependencies. Claude reads the setup instructions and configures everything in your project.

## What Gets Installed

```
your-project/
  .claude/
    data/
      intent-mappings.json    # Your natural language mappings
    commands/
      review-intents.md       # Periodic command to review and update mappings
  CLAUDE.md                   # (modified) Routing snippet added
```

## After Setup

### Review and update mappings

Run `/review-intents` periodically. It re-scans your session transcripts, finds new patterns, and suggests additional mappings.

### Add mappings manually

Edit `.claude/data/intent-mappings.json` directly:

```json
{
  "deploy": {
    "triggers": ["deploy", "ship it", "push to production"],
    "command": "/deploy",
    "description": "Deploy to production"
  }
}
```

### Skill-based intents

Map triggers to skills instead of commands:

```json
{
  "session_resume": {
    "triggers": ["where did we leave off", "what were we working on"],
    "skill": "sessions-api",
    "description": "Resume previous session"
  }
}
```

### URL-based intents

Trigger actions when specific URLs are shared:

```json
{
  "instagram_download": {
    "triggers": [],
    "url_patterns": ["instagram.com/stories/", "instagram.com/reel/"],
    "skill": "instagram-download",
    "description": "Auto-download Instagram content"
  }
}
```

## Honest Tradeoffs

We built this for ourselves and use it daily. Here's what we've learned:

### Token cost is real

Each mapped intent costs ~80-100 tokens of context. The routing instructions add ~1,800 tokens. With 40 intents, that's ~5,300 tokens loaded into every conversation — about 2-3% of a typical session's budget. This scales linearly. Map what matters, skip the rest.

### Claude is probabilistic, not deterministic

Claude pattern-matches triggers in its head. It's not running regex. It will occasionally misroute, especially with similar triggers. The conflict detection in `/review-intents` helps, but it's not perfect. If you need 100% reliability, consider a [UserPromptSubmit hook](https://docs.anthropic.com/en/docs/claude-code) instead.

### You won't map everything — and shouldn't

In our system with 200+ commands and skills, only ~40 have intent mappings (~20%). The rest we invoke directly with `/`. The transcript scanner tells you which commands you actually reach for in conversation vs. which ones you always type as slash commands. No point mapping intents for commands you never try to say naturally.

### Start small

The transcript scanner shows you where to start. You don't need 40 intents on day one. Start with 5-10 high-frequency commands and grow from there.

### Cognitive load on Claude

Every message, Claude mentally checks your trigger list before doing anything else. With a small list this is negligible. With a large list, it adds processing overhead. A shell-based hook would pay zero cognitive cost, but loses Claude's fuzzy matching ability.

## How It Compares to Alternatives

| Approach | Pros | Cons |
|----------|------|------|
| **Shorthand (CLAUDE.md + JSON)** | Zero dependencies, fuzzy matching, easy to customize | Token cost, probabilistic routing |
| **UserPromptSubmit hook** | Deterministic, zero token cost | Requires shell scripting, no fuzzy matching |
| **Claude Code plugin** | Distributable, versionable | Namespaced commands, more complex setup |
| **Hybrid (hook + CLAUDE.md)** | Best of both worlds | More moving parts to maintain |

Shorthand is the simplest starting point. You can graduate to a hook-based system later if your mapping list grows large — the JSON format is the same either way.

## Credits

Built by [Alex Hillman](https://github.com/alexknowshtml) and Andy (his Claude Code assistant).

Born from a real workflow: booking conference rooms, creating invoices, and discovering that natural language is faster than slash commands for the things you do most often.

## License

MIT
