# Meeting Intelligence — Transcript to Action Items Pipeline

## The Problem
Every professional meeting generates 30-60 minutes of post-meeting admin: writing summaries, extracting action items, assigning owners, creating tasks in PM tools, and emailing attendees. A 5-person team with 10 meetings per week each wastes 25-50 hours on this admin alone.

## The Solution
A form-triggered workflow that takes a meeting transcript and automatically:
- Generates an executive summary and lists key decisions
- Extracts structured action items (task, owner, deadline, context)
- Creates a Notion database page for each action item
- Emails the requester a formatted summary with all data

## Architecture
Form Trigger → AI Extraction → Edit Fields → Split Out → Notion (per task) → Aggregate → Gmail

## Why These Nodes
- Form Trigger: Public shareable URL, no external tools needed to test
- OpenAI (JSON mode): Structured action_items array enables per-item iteration
- Split Out / Aggregate: Fan-out/fan-in pattern — map-reduce logic using native nodes
- Notion: Creates real database pages with proper typed properties (Date, Status, Text)
- Gmail: Delivers branded HTML summary to the requester

## Technical Highlights
- Defensive deadline handling: empty AI deadlines default to meeting-date + 7 days
- Custom form styling via CSS variable overrides
- Data normalization layer (Edit Fields) flattens nested AI output for clean downstream references

## Business Impact
- Replaces 30-60 min of post-meeting admin per meeting
- For a 5-person team with 10 meetings/week each: recovers 25-50 hours/week
- At $50/hr loaded cost: $65K-$130K/year in recovered productivity

## What I'd Add in Production
- Gmail trigger to auto-process meeting recording emails (Zoom, Fathom, Otter)
- Sentiment analysis layer for risk escalation
- Slack integration alongside email
- Duplicate detection for repeat meetings
- Error handling on AI node with fallback to raw transcript delivery

## Demo Video
[Loom link here]

## Files
- `workflow.json` — Import into n8n to replicate
