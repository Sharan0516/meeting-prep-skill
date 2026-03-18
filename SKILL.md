---
name: meeting-prep
version: "1.0"
description: >
  Comprehensive meeting preparation skill for Claude Code. Supports 8 meeting
  types (discovery, follow-up, investor, advisory, partnership, customer success,
  internal, hiring) with type-specific hypothesis, questions, and briefing
  sections. Handles multi-person meetings with group dynamics mapping and
  per-person question allocation. Researches contacts via web, performs
  multi-entity orbit mapping, and produces professional HTML briefings.
triggers:
  - manual
dependencies:
  skills: []
  files: []
output:
  - meeting-prep-briefing-html
---

# meeting-prep

You are preparing a meeting briefing using a comprehensive meeting preparation pipeline. This skill adapts to **8 meeting types** (discovery, follow-up, investor, advisory, partnership, customer success, internal, hiring) and handles **multi-person meetings** as a first-class concept.

Core pipeline: gather attendees -> check for prior notes -> research attendees -> type-specific hypothesis -> type-specific questions/answers -> HTML briefing.

**Trigger phrases:** "prep for meeting", "meeting prep", "prepare for call", "brief me on", "who am I meeting", "prepare for my meeting with", "meeting brief", or any reference to preparing for an upcoming meeting or call.

---

## Step 0: Verify Environment

No external dependencies are required. This skill uses:
- **WebSearch** (Claude Code built-in) for researching people and companies
- **WebFetch** (Claude Code built-in) for reading web pages
- **Claude's reasoning** for hypothesis generation, question crafting, and analysis

If WebSearch or WebFetch are unavailable, inform the user and proceed with whatever information they provide directly.

## Step 1: Get Meeting Details & Build Attendee List

Ask the user for:
- **Who** they are meeting (names, companies -- may be multiple people)
- **Meeting purpose/type** (discovery, follow-up, investor, advisory, partnership, customer success, internal, hiring)
- **Desired outcome** (what do you want to walk away with?)
- **Specific topics** to cover or prepare for
- **LinkedIn URLs** or other links for research (optional)
- **Who from your side** is attending (if multiple -- for role assignment)
- **Any prior meeting notes or context** (optional -- paste or point to a file)

### 1a: Build the Attendee List

Structure attendees as a list. Single person = list of 1.

```
attendees_their_side = [
  {name, company, title, linkedin_url, role_in_meeting, is_primary},  # primary = most important
  ...
]
attendees_our_side = [
  {name, title, role_in_meeting},  # e.g., "CTO handles technical deep-dive"
  ...
]
is_multi_person = len(attendees_their_side) > 1 or len(attendees_our_side) > 1
primary_contact = attendees_their_side[0]  # the one marked is_primary
```

**Single attendee (most common):** `attendees_their_side` has 1 entry. Pipeline runs as normal.
**Multiple their-side attendees:** Research each person. Build group dynamics map. Allocate questions per person.
**Multiple our-side attendees:** Add "Who Covers What" section to briefing. Plan handoff moments.

If the user provides partial info (just a name), proceed with what is available. Web research may fill in company and other details.

### Step 1b: Input Validation Gate

Before proceeding to research, validate that enough information exists to produce a useful briefing:

| What We Have | Verdict | Action |
|---|---|---|
| Person name + company | **Proceed** | Full pipeline |
| Person name only (no company) | **Proceed with warning** | Search web for company. Print: "No company specified. Searching web..." |
| Company only (no person) | **Proceed as company-focused** | Skip Steps 3, 5 (person research, orbit map). Produce company briefing with industry context. Print: "Company-only prep. Skipping person research." |
| LinkedIn URL only | **Proceed** | Extract name + company from URL slug or page. |
| Nothing provided | **Stop** | Ask: "Who are you meeting? I need at least a name, company, or LinkedIn URL." |

Also detect **meeting type** using heuristics from `references/briefing-quality-checklist.md` Section 5. If ambiguous, ask: "This looks like a [type] meeting. Correct, or is it something else?"

Set `meeting_type` variable for use in Steps 6, 7, and 8.

## Step 2: Prior Context Detection

Determine whether this is a follow-up meeting.

### 2a: Check for User-Provided Prior Context

Ask the user or check if they provided:
- Prior meeting notes, transcripts, or summaries
- A file path containing notes from previous meetings
- Any context about prior interactions ("we last spoke in January about X")

### 2b: Synthesize Prior Context

**If the user provides prior meeting notes or transcripts:**
- Read and synthesize the key points: what was discussed, what was promised, what changed
- Build prior call context showing relationship status and meeting count
- Set `is_followup = true`

**If no prior context is available:**
- Set `is_followup = false`, `meeting_number = 1`
- Proceed as first meeting -- zero impact on pipeline

### 2c: Company-Level Intel from Prior Notes

If prior notes mention **other people at the same company**, surface that intel:
- What was learned from other contacts at this organization
- Anonymize sources: say "based on prior conversations" not specific names

## Step 3: Research Attendees

**Loop over `attendees_their_side`.** Primary contact gets full research. Additional attendees get lighter research (name, title, role in meeting, likely concerns).

**For multi-person meetings (is_multi_person = true):**
After researching all attendees, build a **Group Dynamics Map**:
- **Decision maker:** Who has final authority?
- **Influencer:** Who shapes the decision maker's opinion?
- **Champion:** Who's pushing for this internally?
- **Blocker:** Who might resist?
- **Evaluator:** Who's doing technical/operational assessment?
- **Conflicting agendas:** Where do attendees' interests diverge?
- **Common ground:** What do all attendees agree on?

**For multi-person our-side:**
Build a **Role Assignment** section:
- Who covers what topic (e.g., "CTO handles architecture, CEO handles commercial")
- Planned handoff moments
- Unified messaging check (same numbers, same positioning)

### Primary Contact Research

### Follow-Up Conditional (is_followup = true)

If Step 2 detected prior context:

**DO NOT re-research their background from scratch.** Prior context exists. Focus on:
- **Check for role/company changes:** Quick web search to detect if they've moved
- **Company news since last contact:** Web search limited to recent developments
- **Open action items status:** Check if any commitments have public evidence of completion
- **Agenda focus:** Decide what to cover this call based on prior context

### First Meeting (is_followup = false)

**If no person info is available** (company-only prep), skip to Step 4 and produce a company-focused briefing.

**If LinkedIn URL is provided**, use WebFetch to read the profile page, then use WebSearch for additional context.

**If no LinkedIn but person details are provided** (name, title, etc.), use WebSearch to find public info.

Search for:
- Person's name + company + title
- Their LinkedIn posts, articles, conference talks
- Prior companies, career trajectory, notable moves
- Published content, thought leadership

**Build the Person Dossier:**

| Field | Description |
|-------|-------------|
| Name & Title | Full name, current role |
| Company | Current employer |
| Career Arc | Key roles, how they got here, years in industry |
| Domain Expertise | What they know deeply |
| Public Voice | Things they've said, written, or shared publicly |
| Conversation Hooks | Mutual connections, shared interests, recent activity (2-3 specific openers) |
| Style Signals | Formal/casual, data-driven/narrative, technical/strategic |

## Step 4: Research the Company

Use WebSearch and WebFetch on the company website. Research:
- Core business: what they do, products/services, value proposition
- Industry and sub-sector classification
- Size signals: employee count, revenue range, locations
- Technology and tools (check careers pages, blog posts, case studies)
- Recent news: funding, acquisitions, leadership changes, product launches
- Regulatory context: what rules/compliance they navigate
- Industry headwinds: current challenges facing their sector

## Step 5: Multi-Entity Orbit Mapping

**Read `references/meeting-types.md` for type-specific overrides to this step.** The default pipeline below applies to `discovery` type. Other types (investor, advisory, partnership, customer-success, internal, hiring) have their own Step 5 overrides that replace or modify the logic below.

This is the core analytical step for discovery/sales meetings. Instead of assessing fit for one company, map the person's entire professional orbit and screen each entity.

### 5a: Extract Meaningful Entities from Person's Background

From LinkedIn profile and research, build a structured map of every company/organization this person has meaningfully touched:
- Current company (primary)
- Past companies where they held director+ level roles, or spent 2+ years
- Board positions, advisory roles
- Companies they founded or co-founded
- Portfolio companies, subsidiaries, or affiliated entities mentioned in role descriptions
- Industry associations or groups where they're actively involved

**Exclude:** Internships, junior positions, short stints (<1 year unless C-suite), passive memberships.

**Output format:**
```
Entity Map:
1. [Current] Company A -- CEO (2025-present) -- Industry description
2. [Portfolio] Company B -- via Company A -- Sub-industry
3. [Advisory] Company C -- EIR (2025-present) -- Industry
4. [Board] Company D -- Board advisor -- Industry
5. [Previous] Company E -- VP (2015-2024) -- Industry description
```

### 5b: First-Pass Fit Screen on Each Entity

For each meaningful entity, do a quick fit screen based on research:

| Entity | Industry Match? | Size Signal | Pain Indicators | Fit |
|--------|----------------|-------------|-----------------|-----|
| Company A | Matches your target market | Mid-market | Strong pain signals | High |
| Company B | Adjacent market | Unknown | Possible fit | Medium |
| Company C | No match | N/A | Network value only | Low |

**Scoring criteria:**
- **High fit**: Industry matches your target market, company size aligns, role suggests relevant pain
- **Medium fit**: Industry matches but size is edge case, or emerging market segment
- **Low fit**: No industry match, but relevant for network value or domain knowledge
- **No fit**: Completely unrelated industry

### 5c: Deep Research on High-Fit Entities

For entities scored High or Medium, do actual web research:
- Current operations, project types, geographic footprint
- Tech stack signals
- Recent news (major projects, incidents, leadership changes)

For Low-fit entities -- skip deep research but note them as conversation reference points.

### 5d: Person Fit Assessment

Separate from company fit. This answers: **Is this person relevant because of who they are, or just because of where they work?**

| Dimension | What to Assess |
|-----------|---------------|
| **Role relevance** | Decision-maker, influencer, day-to-day user, connector, or domain expert? |
| **Domain depth** | Do they understand the pain firsthand, or is it adjacent? |
| **Career signal** | Have they moved through roles that repeatedly touch the relevant domain? |
| **Comparative insight** | Having worked at multiple companies, can they compare approaches? |
| **Network value** | Who can they connect you to? Past companies, board connections, peers. |
| **Engagement type** | Product conversation, expert interview, advisory session, or relationship-building? |

**Person fit scoring:**
- **High**: Touches domain directly, has decision-making authority, works at a high-fit company
- **Medium**: Oversees domain (not hands-on), or at a medium-fit company, or expert without buying authority
- **Low**: No direct domain connection, but has network value or knowledge worth extracting

### 5e: Persona Category Assignment

Based on entity map and person assessment, assign a persona category:

| Category | Signals | Primary Interest |
|----------|---------|-----------------|
| `EXEC` | C-suite, VP-level, P&L owner | Portfolio visibility, governance, strategic risk |
| `PRACTITIONER` | Manager, coordinator, hands-on operator | Time savings, workflow efficiency, tool consolidation |
| `LEGAL` | General counsel, compliance officer | Regulatory risk, audit defensibility |
| `COMPLIANCE` | Compliance manager, risk officer | Gap detection, automated monitoring |
| `PROCUREMENT` | Procurement lead, vendor manager | Vendor lifecycle, onboarding efficiency |
| `FINANCE` | CFO, controller, finance director | Cost control, accuracy, reserve management |
| `HR` | HR director, workforce compliance | Workforce compliance, classification accuracy |
| `OPERATIONS` | COO, operations manager | Operational efficiency, process standardization |
| `ENGINEERING` | CTO, VP Engineering, tech lead | Technical architecture, integration, scalability |
| `PRODUCT` | CPO, product manager | Feature fit, roadmap alignment, user experience |
| `SALES` | CRO, VP Sales, sales leader | Pipeline impact, revenue growth, competitive edge |
| `ADVISOR` | Board member, mentor, investor, industry advisor | Strategic feedback, market validation, competitive landscape |

## Step 6: Hypothesis Synthesis

**Read `references/meeting-types.md` for type-specific hypothesis format.** The default below applies to `discovery` type. Other types use entirely different hypothesis structures (e.g., investor uses "Investor Angle", advisory uses "Decision/Dilemma Framing", customer-success uses "Account Status").

Synthesize everything from Steps 2-5 into a concise hypothesis block. This is the analytical core -- it connects:
- **Who they are** (person research)
- **What they struggle with** (research findings, industry analysis)
- **What competitors they use** (research findings)
- **How you can help** (your product/service positioning)

This is the "why this meeting matters" section.

**Generate:**

```
HYPOTHESIS
-- Primary opportunity: [Which entity is the main fit? Why?]
-- Secondary angles: [Other high-fit entities from their background]
-- Person fit: [High/Medium/Low -- are they a buyer, user, expert, or connector?]
-- Engagement type: [Customer Discovery / Expert Insight / Channel-Partner / Advisory / Network Expansion]
-- Key unknowns: [2-3 specific things you need to learn from this conversation]
-- Companies to reference: [Which companies from their background to mention]
-- Predicted objections: [2-3 likely pushback points based on research]
-- Prior context (follow-ups only): [If is_followup: summarize prior interactions. If first meeting: omit.]
```

**The key unknowns drive the questions.** If research already answers something, don't ask it. The unknowns are things research COULDN'T answer.

### 6b: Value Synthesis

Using the persona category and research, synthesize a "Why This Matters" block:

**Pain Points for This Role** -- 3-5 bullet points relevant to their persona, ranked by likelihood based on research.
If no validated data for this segment, flag as "pure discovery."

**What You Solve** -- 1-2 key value propositions described in this person's language using their entity names.

**The Pitch** -- Single paragraph, 2-3 sentences, second person, using their entity names and relevant figures.

## Step 7: Generate Questions or Answers (Type-Dependent)

**Read `references/meeting-types.md` for type-specific question/answer generation.** The paradigm flips by type:
- **Discovery/Follow-up/Advisory/Partnership/Customer Success:** Generate QUESTIONS (you're asking)
- **Investor/Hiring (being interviewed):** Generate ANSWERS (you're being asked) + a few questions for them
- **Internal:** Generate DISCUSSION QUESTIONS (you're facilitating)

**Multi-person question allocation (is_multi_person = true):**
- Total question budget: 10-12 for the meeting, NOT per person
- Tag each question with the intended respondent: `[For VP Eng]`, `[For Procurement]`, `[For all]`
- Sequence: start with common-ground questions, then role-specific ones
- Adapt depth to seniority: strategic for execs, operational for practitioners

**Default below applies to discovery/demo type.** Questions should be informed by ALL the research done above -- never ask something the research already answered.

Read `references/question-bank.md` as a prompt library for phrasing inspiration.

### Follow-Up Meeting Questions (is_followup = true)

Replace default structure with follow-up-specific blocks:

**Block 1 -- "Building on Prior Context" (2-3 questions):**
Follow up on commitments, action items, and events since the last interaction. Source from prior meeting notes.

**Block 2 -- "Deepening Discovery" (4-5 questions):**
Go deeper into topics introduced but not fully explored. Rules:
- **Never repeat already-answered questions** -- check prior notes for topics covered
- **Reference their own context** -- use prior interaction data to show continuity
- **Calibrate rather than re-discover** -- refine understanding, don't start over
- **Probe gaps** -- what was hinted at but not elaborated?

**Block 3 -- "Strategic & Expansion" (3-4 questions):**
Entity names and context from prior interactions. Referrals, advisory formalization, next steps.

### First Meeting Questions (is_followup = false)

**Block 1: Opening Reaction (2 questions):**
1. "What stood out to you?" -- Open-ended, let them lead.
2. "What's missing? What would you need to see?" -- Inversion, let them identify gaps.

**Block 2: Context & Validation (5-8 questions):**
Every question demonstrates you've done your homework.
- Never ask something research already answered
- Reference specific companies from their background
- Probe the key unknowns from the hypothesis

**Question sources (pick the best 5-8):**
- From their **current company**: process, workflow, pain, tech stack, scale
- From their **career arc**: comparative insight across companies
- From **entity-specific research**: reference specific findings

**Block 3: Strategic & Expansion (3-4 questions):**
- Company-specific references from their background
- Org structure probe
- Advisory frame
- Next step / proof-of-value offer

### Objection Preparation

**For follow-ups:** Check prior notes for objections they raised previously. Address specifically with evolved responses.

**For first meetings:** Based on persona category and research, list 2-3 most likely objections with response strategies.

Format:
```
If they say: "[Objection]"
Prepare: [Response strategy]
```

### Question Quality Checklist

Before finalizing, verify each question:
- [ ] Is it specific to THIS person and THESE companies? (Uses names, tools, details)
- [ ] Does it ask something research COULDN'T answer?
- [ ] Does it serve a purpose? (Validates fit, quantifies pain, reveals workflow, expands network)
- [ ] Would this person find it insightful? (Shows preparation)

## Step 8: Generate HTML Briefing

**Read `references/briefing-quality-checklist.md` before finalizing.** This is the quality calibration reference. Use it to:
1. Run universal quality gates against the briefing content
2. Run meeting-type-specific gates (using `meeting_type` from Step 1b)
3. Run multi-person gates if multiple attendees
4. Score each confidence dimension (research depth, outcome clarity, question quality, risk preparation, attendee coverage)
5. Calculate overall confidence score
6. Add Prep Gaps callout if any dimension scores below 7

### HTML Structure -- 9-Section Document

**Design philosophy:** This is a personal prep document. Optimize for **scanning speed** over visual polish. Think Notion page, not SaaS landing page.

**Overall page:**
- White background, single column
- Max-width container: `720px`, centered, with `padding: 48px 24px`
- Font: Inter (from Google Fonts), fallback to -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif
- Headings: `#121212`, body text: `#4B5563`
- Accent color: `#2563EB` (blue) for badges, links, and highlights
- Sections separated by `<hr>` with `border-top: 1px solid #E5E7EB` and `margin: 32px 0`
- No background alternation between sections

**Header block:**
- Meeting type badge (e.g., "Discovery", "Investor", "Advisory") in accent color
- Follow-up badge (if is_followup): Green success badge: "Nth Meeting"
- **Single attendee:** Person name (H1), title + company below
- **Multi-person:** Meeting title (H1, e.g., "Acme CPQ Review"), attendee list below with names + titles
- Desired outcome (1 sentence, bold) -- from Step 1
- Meeting date. For follow-ups: append last interaction date.
- Badge row: meeting type, persona category (primary), engagement type, person fit, company fit
- Print button (top right)

**Section 0.5 -- Attendee Dossiers (multi-person only):**
If `is_multi_person`, add a section before the Executive Summary:
- Mini-card per attendee: name, title, role in meeting (decision maker/influencer/evaluator), 2-3 bullet background, likely concerns
- Group dynamics map: power structure visualization (who defers to whom)
- If `attendees_our_side` > 1: "Who Covers What" table (our person -> their topic -> handoff trigger)

**Section 0.5 is omitted for single-attendee meetings.**

**Section 1 -- Executive Summary / Hypothesis & Angle:**
- Structured bullet list
- Primary opportunity, secondary angles, person fit, engagement type
- **Key unknowns** -- bold, highlighted. These drive the questions.
- Companies to reference

**Section 1.5 -- Prior Context Recap (follow-ups only):**
Blue info box with:
- What was covered in prior interactions
- Commitments made / action items
- Relationship status summary

**Section 2 -- Why This Matters (Value Synthesis):**
- Pain Points for This Role -- 3-5 bullet points ranked by evidence
- What You Solve -- 1-2 key value propositions described in their language
- The Pitch -- 2-3 sentence tailored value proposition

**Section 3 -- Conversation Hooks:**
- 3 numbered hooks with bold opener + context
- Style signals as a single caption line

**Section 4 -- Key Background:**
- 6-8 bullet points covering career arc, education, expertise
- Only what's relevant to THIS conversation

**Section 5 -- Entity Map (Orbit Map):**
- High/Medium fit entities: paragraph each with fit badge, context, and conversation relevance
- Low fit entities: single line with network value note
- No fit entities: grouped in one line

**Section 6 -- Questions (10-15):**
- For follow-ups: "Building on Prior Context" (2-3), "Deepening Discovery" (4-5), "Strategic & Expansion" (3-4)
- For first meetings: "Opening Reaction" (2), "Context & Validation" (5-8), "Strategic & Expansion" (3-4)
- Must-ask: `font-weight: 600`
- If-time-allows: `font-weight: 400`, muted color
- Follow-up prompts: indented, smaller font
- Purpose line: smallest font, muted

**Section 7 -- Objection Prep:**
- 2-3 items: bold objection text, then response paragraph
- Separated by light horizontal rules

**Section 8 -- Quick Reference (print-friendly):**
- Compact section with accent-color top border
- Key-value: name, role, fit, pitch, top 5 must-ask, key unknowns
- Confidence score from quality checklist
- Print CSS: show all sections, reduce fonts, max-width 100%, hide print button and sources

**Section 9 -- Sources (screen only, hidden on print):**
- List of all URLs used during research
- Small font, muted color, underlined links

### CSS Requirements

Include the following in the HTML `<head>`:
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

Key styles:
- `.badge` -- inline-block, padding 2px 10px, border-radius 999px, font-size 13px
- `.must-ask` -- font-weight 600
- `.if-time` -- font-weight 400, color #9CA3AF
- `.follow-up-prompt` -- margin-left 24px, font-size 14px, color #6B7280
- `.purpose` -- font-size 12px, color #9CA3AF
- `.info-box` -- background rgba(37,99,235,0.05), border-left 3px solid #2563EB, padding 16px, border-radius 8px
- `.objection` -- font-weight 600
- `.source-link` -- font-size 13px, color #6B7280

Print CSS (`@media print`):
- Hide print button and sources section
- Max-width 100%
- Reduce font sizes by ~15%
- All links as plain text (no underline, inherit color)

### Linking Requirements

Every mention of a company, article, or profile should be a clickable link when a URL is available. Print: all links as plain text.

### Save & Deliver

Save the HTML briefing. Suggested path pattern: `~/meeting-prep/YYYY-MM-DD-person-name.html`

If the user has a preferred output directory, use that instead.

**Always end with the deliverables block:**

```
---
YOUR FILES
---
  Briefing (HTML):  /absolute/path/to/briefing.html
                    Open in any browser for the full prep doc
---
```

## Step 9: Post-Meeting Reminder

After generating the briefing, remind the user:

> After your meeting, paste your notes back here and I can help you:
> - Extract key takeaways and action items
> - Draft follow-up emails
> - Prepare for the next meeting with richer context

---

## Memory & Learned Preferences

Check for auto-memory file at the skill's installation directory or a user-specified location. If it exists, load:
- Preferred briefing format (detailed vs concise)
- Key contacts and their companies (avoid re-asking)
- Meeting type preferences for recurring meetings
- Topics the user always wants covered
- Preferred output directory for briefings

After generating a briefing, note any new preferences the user expresses.

## Progress Updates

Report status at each milestone:
- "Researching [person name]... (1 of N attendees)"
- "Company intel gathered -- moving to recent news..."
- "Hypothesis drafted -- generating briefing document..."

For multi-attendee meetings, report after each person completes.

## Error Handling

- **No web access:** Produce a briefing using only user-provided information. Note: "Web research unavailable. Briefing is based on provided context only."
- **Partial data:** Include whatever is available; never crash on missing data
- **WebSearch returns nothing:** Try alternative search queries (different name formats, company variations). If still nothing, note the gap and continue.
- **LinkedIn profile not accessible:** Fall back to web search for the person's name + company. Note the limitation.

## Guidelines

- **Be honest about unknowns.** If info isn't findable, say so. Never fabricate.
- **Privacy first.** Only use publicly available information.
- **Scale to context.** A 15-min coffee chat needs a lighter brief than a 45-min deep dive.
- **Questions are the product.** The question list is the most valuable section.
- **Entities first, product second.** Map the person's orbit before thinking about fit.
- **Research eliminates questions.** If research answers it, don't ask it.
- **Anonymize past insights.** Say "teams in this segment report..." not specific names.
- **Flag novel segments.** If first conversation in a segment, note it explicitly.
- **Reference real companies.** Use actual company names from their background.

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-19 | Initial standalone release. Full 8-type meeting support (discovery, follow-up, investor, advisory, partnership, customer success, internal, hiring). Multi-person meetings with group dynamics. Web research via WebSearch/WebFetch. Entity orbit mapping. Hypothesis and question generation. Professional HTML briefings with confidence scoring. No external dependencies beyond Claude Code. |
