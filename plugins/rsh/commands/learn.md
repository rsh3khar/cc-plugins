# Learn from my Claude Code history

Analyze my Claude Code conversations and generate personalized insights.

## Step 0: Check dependencies

Check if `jq` is installed (`which jq`). If not, offer to install it:
- macOS: `brew install jq`
- Ubuntu/Debian: `sudo apt install jq`

## Step 1: Extract raw conversations

First, detect the OS (`uname -s`) and adapt commands if needed (e.g., Linux vs macOS differences).

Run this extraction script to get raw user messages:

```bash
OUTFILE="/tmp/cclearn_raw.md"
TMPDIR="/tmp/cclearn_parts"
mkdir -p "$TMPDIR"

# Process files in parallel (8 concurrent jobs)
find ~/.claude/projects -name "*.jsonl" -type f -print0 2>/dev/null | \
  xargs -0 -P 8 -I {} sh -c '
    f="{}"
    session=$(basename $(dirname "$f") | sed "s/-/\//g" | rev | cut -d"/" -f1-3 | rev)
    outfile="$0/$(echo "$f" | cksum | cut -d" " -f1).txt"
    {
      echo "---"
      echo "## Session: $session"
      grep "\"type\":\"user\"" "$f" 2>/dev/null | jq -r "
        (.message.content // .content) as \$c |
        if (\$c | type) == \"array\" then (\$c[0].text // \$c[0] // \"\") else (\$c // \"\") end |
        if type == \"string\" then . else tostring end |
        gsub(\"^\\\\s+|\\\\s+$\"; \"\") |
        select(length > 0) |
        select(startswith(\"{\\\"tool_use_id\\\"\") | not) |
        select(startswith(\"{\\\"type\\\":\") | not) |
        .[:500] | \"USER: \\(.)\"
      " 2>/dev/null
    } > "$outfile"
  ' "$TMPDIR"

# Combine all parts (sorted by modification time)
cat "$TMPDIR"/*.txt > "$OUTFILE" 2>/dev/null
rm -rf "$TMPDIR"
echo "Extracted to: $OUTFILE"
echo "Lines: $(wc -l < "$OUTFILE")"
```

**Important:** After extraction, check the line count. If it's very large (>20000 lines), the file will be too big to read directly. Use `head`, `tail`, or sampling - don't try to read the whole file at once.

## Step 2: Analyze and create cleaning script

First, check file size:
```bash
wc -l /tmp/cclearn_raw.md
```

Sample random lines for analysis:
```bash
shuf /tmp/cclearn_raw.md | grep "^USER:" | head -2000
```

**Note:** The bash output is shown directly - do NOT try to re-read it from tool-results files. Use the output as-is to identify patterns.

Look for noise patterns in the sample:
- Tool results: `{"tool_use_id"`, `{"type":"tool_result"`
- System messages: `<system-reminder>`, `<task-notification>`
- Automated output, logs, error dumps
- Repeated boilerplate specific to this user

Create `/tmp/cclearn_filter.sh` with patterns you identified:

```bash
#!/bin/bash
# Cleaning script - patterns identified for this user
grep "^USER:" "$1" | \
  grep -v 'tool_use_id' | \
  grep -v 'tool_result' | \
  grep -v '<system-reminder>' | \
  grep -v 'PATTERN_FROM_SAMPLE'
# Add more grep -v lines based on what you found
```

Run it:
```bash
chmod +x /tmp/cclearn_filter.sh
/tmp/cclearn_filter.sh /tmp/cclearn_raw.md > /tmp/cclearn_clean.md
echo "Cleaned: $(wc -l < /tmp/cclearn_clean.md) lines"
```

Keep genuine user messages - even short, frustrated, or code-heavy ones.

## Step 3: Launch analysis agents in parallel

Check cleaned file size: `wc -l /tmp/cclearn_clean.md`

Launch 4 agents IN PARALLEL (single message, multiple Task tool calls), each analyzing `/tmp/cclearn_clean.md`.

**Reading strategy for agents:**
- Start with a big initial sample (first 5000 lines) to understand the user
- Then read the rest in 2000-3000 line chunks using offset/limit parameters
- Read safely - don't try to load >3000 lines at once
- Process the ENTIRE file to truly understand the user (not just samples)

### Agent 1: Communication Style
Read the whole cleaned file safely (chunks with offset/limit). Analyze how the user communicates - tone, directness, feedback style, conversation patterns. Return summary (max 500 words) with quotes as evidence.

### Agent 2: Code Preferences
Read the whole cleaned file safely (chunks with offset/limit). Analyze coding preferences - languages, patterns, conventions, things they correct Claude on. Return summary with quotes.

### Agent 3: Frustrations & Corrections
Read the whole cleaned file safely (chunks with offset/limit). Analyze what frustrates the user - "no/wrong" patterns, complaints, repeated corrections. Return summary with quotes.

### Agent 4: Projects & Context
Read the whole cleaned file safely (chunks with offset/limit). Analyze what the user works on - projects, technologies, role, recurring themes. Return summary with quotes.

## Step 4: Synthesize INSIGHT.md

Combine agent findings into `INSIGHT.md` in current directory:

- Who they are (role, projects, tech)
- Communication style
- Code preferences
- What frustrates them
- Patterns & habits

**Tone:** Brutally honest. Don't flatter. Call out inconsistencies. Use quotes as evidence. Mirror, not pep talk.

## Step 5: Generate LEARN.md

Write `LEARN.md` in current directory - instructions FOR Claude on how to work with THIS specific user.

Include:
- Communication: how to talk to them (tone, verbosity, what they hate)
- Coding: their preferences, patterns, conventions
- Do: specific behaviors they appreciate
- Don't: specific things that frustrate them (be detailed, these are critical)
- When frustrated: how to recognize and respond

**Do NOT include:**
- Project-specific context (projects change)
- Generic advice that applies to everyone

**Tone:** Practical, direct. Just instructions. No fluff.

**Goal:** This file should make every future Claude session 100x better for this user. Be thorough and specific. Capture everything that makes this user unique.

## Step 6: Ask about CLAUDE.md

Use AskUserQuestion to ask what they want to do with LEARN.md:

Options:
1. **Append** - Add LEARN.md contents to ~/.claude/CLAUDE.md as-is
2. **Merge** - Intelligently merge with existing CLAUDE.md (avoid duplicates, fill gaps)
3. **Review first** - Let me review LEARN.md before deciding
4. **Keep separate** - Don't touch CLAUDE.md, keep LEARN.md as standalone reference

Execute their choice.

## Step 7: Cleanup

Remove temp files: `rm -f /tmp/cclearn_*.md /tmp/cclearn_*.sh`

## Guidelines

- Launch all 4 agents IN PARALLEL
- Be specific with examples, not generic
- Don't sugarcoat - honest observations only
