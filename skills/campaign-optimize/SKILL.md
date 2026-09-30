---
name: campaign-optimize
description: >-
  Post-publish campaign optimization via MCP: read daily reports, propose lifecycle changes,
  optionally save strong creatives as Canva templates. Always load when the campaign is PUBLISHED
  or the advertiser asks for performance / pause / resume / end. No silent spend changes.
license: proprietary
---

# Campaign Optimize

Read performance and propose lifecycle actions the advertiser must confirm.

## Workflow

1. Confirm which campaign (ask if ambiguous). Orient with status tools when context helps.
2. Read performance with available report tools — choose from the MCP schema.
3. State findings with tool-backed numbers; propose pause / resume / end only with advertiser intent.
   Before `pause_campaign`, `resume_campaign`, `end_campaign`, or any Meta write tool, ask the
   advertiser; pass `userConfirmed: true` only after they say yes in this conversation.
4. Optional: suggest saving a winning archived design as a Canva brand template (agent Canva MCP;
   edit/archive only — never Canva invent) with advertiser intent.
5. Stop — do not start a new build. Hand builds back to `campaign-workflow`.

## Rules

- DO NOT change budgets or bids silently
- DO NOT publish, create, or rebuild campaigns from this skill
- DO NOT bypass human approval for spend or delivery changes
- DO NOT invent metrics not returned by tools
- DO NOT hand-edit Meta Graph JSON — rebuilds go through `campaign-workflow` artifacts
- ONLY report, recommend, and run lifecycle tools when the advertiser intends them

## Examples

Input: “How is campaign 12 doing?”  
Output: report tools → plain-language findings → optional pause/resume proposal

Input: “Pause the underperforming ad set”  
Output: ask to confirm → advertiser says yes → lifecycle tool (`userConfirmed: true`) → confirm
result

## Edge cases

- Ambiguous campaign → ask which id before acting
- Rebuild request → hand off to `campaign-workflow` (strategy → assets → creatives → media)
