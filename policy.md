# outlook-mcp Policy Rules — Example

> **This is an example.** The `initialize` tool generates your personalized policy
> at `~/.outlook-mcp/policy.md` by probing Outlook and (optionally) querying your
> org chart via agency/WorkIQ.

The AI policy evaluator reads `~/.outlook-mcp/policy.md` before approving any write action.

## Sender Context

- **Name**: (populated by initialize)
- **Title**: (populated by initialize)
- **Email**: (populated by initialize)
- **Organization**: (populated by initialize)

### Direct Reports

- (populated by initialize from agency/WorkIQ, or fill in manually)

### Leadership Chain

1. **Manager**: (name — email)
2. **Skip**: (name — email)
3. **CVP**: (name — email)

### Key Lateral Leaders

- (add important cross-org stakeholders here)

## Recipient Rules

- Max 10 recipients per email (to + cc combined). Hard limit — no exceptions.
- NEVER email large distribution lists (50+ members).
- NEVER email VP+ leadership unless the user explicitly requested it by name.
- Flag emails to external domains — deny unless clearly intentional.
- Extra scrutiny for skip-level and above.
- Emailing direct reports is always safe — they are the user's team.

## Content Rules

- Professional tone required. No passive-aggressive language, sarcasm, or humor that could be misread.
- No confidential project details to external recipients.
- No commitments on timelines, headcount, or budgets without "tentative" or "pending approval" qualifier.
- No sharing of performance review content, compensation data, or HR-sensitive information.
- Reply length should be proportional to the original email — don't send a novel in reply to a one-liner.
- When replying to leadership (skip+), keep replies concise and action-oriented.

## Action Rules

- Max 20 outbound emails per rolling hour. Hard limit.
- Calendar invites require at least 24h notice unless the body mentions "urgent" or "ASAP".
- Meeting cancellations: only cancel meetings the user organized. Never cancel someone else's meeting.
- Categorize/move operations: max 10 items per call.

## Deny by Default

If you are uncertain whether an action should be approved, **DENY it**. It's safer to block
a legitimate action (the user can retry manually) than to allow a harmful one.
