# TILI

TILI runs your Meta advertising from a chat. It sets up your Brand OS (voice, visuals, audience,
offers), then walks each campaign through strategy, an assets plan, creatives, and a media plan.
Every phase waits for your approval on a card. Nothing is published, paused, or ended on Meta
until you say yes.

## How to use it

1. Install the plugin and sign in to TILI when your assistant asks (OAuth). You need a TILI
   account at [tili.global](https://tili.global).
2. Connect your Meta ad account in the TILI app if you have not already.
3. Ask, for example: "Set up my Brand OS from my website", "Start a Meta campaign for our summer
   sale", or "How is my campaign doing?"

The plugin bundles skills (the workflow your assistant follows) and one MCP server:
`https://backend.tili.global/mcp`.

## What data is sent where

- **To TILI (`https://backend.tili.global/mcp`):** your chat assistant calls TILI tools over OAuth. TILI receives
  your account email, the Brand OS and campaign drafts you work on, your approvals, and images you
  upload. TILI stores them in your workspace; uploaded images are kept in TILI object storage.
- **To Meta:** TILI uses the Meta token you connected to read your ad account, pages, posts,
  audiences, pixel stats, and reporting. Only after you approve does TILI send the campaign
  structure, budget, targeting, and creatives to Meta, and pause, resume, or end delivery.
- **Brand pages:** when you give a website link, TILI fetches that one page to read text and
  image links for your Brand OS. It does not crawl other sites.
- **Connected databases and files:** only if you connect one in TILI, the database or file tools
  return a small sample of rows to your assistant.
- **Back to your assistant:** tool results include workspace and campaign ids, Brand OS and
  campaign content, image links, and any data samples above. TILI never returns access tokens.

Privacy: [tili.global/privacy](https://tili.global/privacy). Terms:
[tili.global/terms](https://tili.global/terms). Support:
[tili.global/contact](https://tili.global/contact).
