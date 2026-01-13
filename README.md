# cclearn

A Claude Code plugin that analyzes your conversation history and generates personalized instructions so Claude understands how you work.

## What it does

1. **Extracts conversations** from `~/.claude/projects/`
2. **Cleans the data** - Claude identifies and filters noise specific to your usage
3. **Analyzes with 4 parallel agents** - Communication style, code preferences, frustrations, patterns
4. **Generates two files:**
   - `INSIGHT.md` - Brutally honest reflection about you (patterns, style, frustrations)
   - `LEARN.md` - Instructions for Claude (how to work with you)
5. **Optionally updates CLAUDE.md** - Append, merge, or keep separate

## Install

```bash
/plugin marketplace add rsh3khar/cc-plugins
/plugin install rsh
```

## Requirements

- `jq` - JSON processor (Claude will offer to install if missing)

## Usage

```
/learn
```

That's it. Claude reads your history, analyzes it, and generates the files.

## Why?

Every Claude session starts fresh. CLAUDE.md helps, but writing it manually is tedious and you miss patterns you don't notice about yourself.

`/learn` makes Claude analyze your actual conversations - how you communicate, what frustrates you, what you correct Claude on - and generates instructions so future sessions work better.

## Output

**INSIGHT.md** - Honest reflection about you:
- Who you are (role, experience, what you're building)
- Your communication style (with quotes from your conversations)
- What frustrates you (patterns Claude should avoid)
- Honest observations (things you might not notice about yourself)

**LEARN.md** - Instructions for Claude:
- How to communicate with you
- Your code style preferences
- Specific do's and don'ts
- How to recognize and respond when you're frustrated

## License

MIT
