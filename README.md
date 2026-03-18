# Meeting Prep Skill for Claude Code

A comprehensive meeting preparation skill that produces professional HTML briefings. Supports 8 meeting types with type-specific research, hypothesis generation, question crafting, and objection preparation.

## What It Does

Give it a person's name, company, and meeting purpose. It will:

1. **Research** the person and company using web search (LinkedIn, company website, news)
2. **Map their orbit** -- current company, past roles, board positions, advisory roles
3. **Generate a hypothesis** about the opportunity and key unknowns
4. **Craft questions** organized by blocks (prior context, discovery, strategic)
5. **Prepare objection responses** based on persona and research
6. **Produce an HTML briefing** you can open in any browser or print

## Supported Meeting Types

| Type | What It Prepares |
|------|-----------------|
| **Discovery** | Full orbit mapping, pain hypothesis, demo-led questions |
| **Follow-up** | Deal status, open threads, blocker-focused questions |
| **Investor** | Portfolio analysis, answer bank (you're being asked), hard question prep |
| **Advisory** | Dilemma framing, advisor-specific questions, decision points |
| **Partnership** | Mutual value analysis, integration feasibility, deal structure options |
| **Customer Success** | Account health, value delivered, expansion opportunities |
| **Internal** | Decision framing, stakeholder positions, discussion guide |
| **Hiring** | Candidate fit analysis, role-specific questions, assessment rubric |

## Multi-Person Support

Works with meetings involving multiple attendees on either side:
- Individual dossiers per attendee
- Group dynamics mapping (decision maker, influencer, blocker, champion)
- Per-person question allocation (10-12 total, tagged by respondent)
- Role assignment for your side (who covers what, handoff moments)

## Requirements

- **Claude Code** (that's it -- no other dependencies)
- The skill uses Claude Code's built-in `WebSearch` and `WebFetch` tools for research
- No API keys, databases, or external services needed

## Installation

1. Create the skill directory in your Claude Code skills folder:

```bash
mkdir -p ~/.claude/skills/meeting-prep/references
```

2. Copy the skill files:

```bash
cp SKILL.md ~/.claude/skills/meeting-prep/SKILL.md
cp references/meeting-types.md ~/.claude/skills/meeting-prep/references/meeting-types.md
cp references/briefing-quality-checklist.md ~/.claude/skills/meeting-prep/references/briefing-quality-checklist.md
```

3. That's it. The skill is now available in Claude Code.

## Usage

In Claude Code, say any of these:

- "Prep for my meeting with Jane Smith at Acme Corp"
- "Meeting prep: investor pitch with Sarah Chen from Sequoia"
- "Prepare for my advisory call with Dr. Miller"
- "Brief me on the partnership discussion with Stripe"

Claude will ask for any missing details (meeting type, desired outcome, topics) and then run the full pipeline.

### Providing Prior Context

For follow-up meetings, you can provide prior meeting notes:

- Paste notes directly: "Here are my notes from our last call: ..."
- Point to a file: "My notes are in ~/meetings/acme-notes.md"
- Give a summary: "We last spoke in January about their compliance workflow"

This unlocks follow-up-specific question blocks and avoids repeating what you already know.

## Output

The skill produces a single HTML file you can open in any browser. Default location: `~/meeting-prep/YYYY-MM-DD-person-name.html`

The briefing includes:
- Executive summary with hypothesis and key unknowns
- Conversation hooks and style signals
- Career background (relevant highlights only)
- Entity orbit map with fit scoring
- 10-15 questions organized by block, with must-ask and if-time-allows markers
- Objection preparation with response strategies
- Quick reference card (print-friendly)
- Source URLs for all research

## Customization

The skill adapts based on what you tell it. Over time, it can learn your preferences:
- Preferred briefing depth (concise vs. detailed)
- Recurring contacts and companies
- Topics you always want covered
- Output directory preferences

## File Structure

```
meeting-prep/
  SKILL.md                                  # Main skill definition
  references/
    meeting-types.md                        # Type-specific pipeline overrides
    briefing-quality-checklist.md           # Quality gates and confidence scoring
  README.md                                 # This file
```

## License

MIT
