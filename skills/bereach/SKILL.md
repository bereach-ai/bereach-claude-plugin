---
name: bereach
description: "Automate LinkedIn outreach via BeReach (berea.ch). Use when: prospecting, engaging posts, scraping engagement, searching LinkedIn, managing inbox, running campaigns, managing invitations. Requires BEREACH_API_KEY."
lastUpdatedAt: 1772619338
---

# BeReach — LinkedIn Outreach Automation

Automate LinkedIn prospection and engagement via BeReach.

## Sub-skills

**At install**: fetch and read ALL sub-skills.

**After install**: re-load on-demand only when needed.

| Sub-skill             | Keywords                                                                                                                  | URL                                                     | lastUpdatedAt |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ------------- |
| Lead Magnet           | lead magnet, comment to DM, resource delivery, post giveaway, auto-accept invitations, cron, recap, campaign stats, pause | sub/lead-magnet.md       | 1772619338    |
| SDK Reference         | sdk, method, parameter, signature, reference, api, script                                                                 | sdk-reference.md         | 1772619338             |

## Installation

### Tools (interactive use)

Tools are already configured by the plugin — no manual setup needed.

### SDK (script generation)

The TypeScript SDK provides typed methods for all BeReach operations. Use it when generating scripts (cron jobs, batch automations).

```bash
npm add bereach
```

```typescript
import { Bereach } from "bereach";

const token = process.env.BEREACH_API_KEY;
if (!token) {
  console.error("BEREACH_API_KEY environment variable is required");
  process.exit(1);
}
const client = new Bereach({ token });
```

The SDK auto-reads `BEREACH_API_KEY` from the environment. NEVER hardcode tokens in scripts.

## Onboarding

1. **Token** — read `BEREACH_API_KEY` from environment with no default/fallback value. If not set, stop immediately.
2. **Read all sub-skills** — fetch every sub-skill listed above.
3. **Profile + language** — call `getLinkedInProfile` (tool or `client.profile.getLinkedInProfile()`). Profile inputs: URL (e.g. `https://linkedin.com/in/username`) or URN (`urn:li:fsd_profile:ACoAA...`). Detect user's language from profile (location, headline, posts). This becomes the default language for all generated content. Per prospect, adapt if their profile clearly indicates a different language.
4. **Welcome** — personalized welcome using profile data. Suggest a first action based on their profile.

## Rules

### Interactive mode (tools)

- Use available tools exclusively. If a tool doesn't exist, the operation doesn't exist.
- NEVER use raw HTTP calls (fetch, axios, curl).

### Script mode (SDK)

- Import from the `bereach` SDK exclusively. If a method doesn't exist, the operation doesn't exist.
- Scripts MUST be TypeScript (`.ts`). Run with `npx tsx script.ts`.
- Run `npx tsc --noEmit` before executing to catch type errors.
- Load the [SDK Reference](sdk-reference.md) sub-skill for method signatures.

### Always

- NEVER hardcode tokens. Read from `process.env.BEREACH_API_KEY`. If not set, exit immediately.
- NEVER invent a tool/method name. Check the resources table below or tool schemas.
- If unsure about a method: check the SDK Reference sub-skill.

### SDK resources and methods

| Resource | Methods |
| --- | --- |
| `client.linkedinScrapers` | `collectLikes`, `collectComments`, `collectCommentReplies`, `collectPosts`, `visitProfile`, `visitCompany` |
| `client.linkedinActions` | `connectProfile`, `listInvitations`, `acceptInvitation`, `sendMessage`, `replyToComment`, `likeComment`, `publishPost` |
| `client.linkedinChat` | `listConversations`, `searchConversations`, `findConversation`, `getMessages` |
| `client.linkedinSearch` | `unifiedSearch`, `searchPosts`, `searchPeople`, `searchCompanies`, `searchJobs`, `searchByUrl`, `resolveParameters` |
| `client.profile` | `getLinkedInProfile`, `refresh`, `getPosts`, `getFollowers`, `getLimits`, `getCredits` |
| `client.campaigns` | `getStatus`, `syncActions`, `getStats` |

## Tone

Adapt to user's tone from their posts and messages. Comment replies: 3-5 words, no personalization needed. DMs: 1-2 sentences, personalize from visit data (headline, company, position, recent posts) and conversation history when available.

## Constraints

These are technical constraints BeReach requires. Everything else, adapt as needed.

1. **Dedup** — pass `campaignSlug` on every action. BeReach deduplicates by target automatically. Duplicates return `duplicate: true` and cost nothing. Pre-check: use `client.campaigns.getStatus()` or the `getStatus` tool.
2. **Pacing** — after every SDK/tool call, sleep a random delay. Write actions (DM, reply, like, connect, accept, sync): `random delay 8-12s`. Read actions (visit, scrape, find, count-0 checks): `random delay 2-6s`. On 429: wait the number of seconds from the error response, retry (max 3). If daily/weekly cap hit, switch action type.
3. **Save incrementally** — persist tracking after each action, not at the end.
4. **Limits check** — call `getLimits` once per day at session start.
5. **Visit before connecting** — looks natural to LinkedIn.
6. **Credits** — check with `getCredits` → `{credits: {current, limit, remaining, percentage}}`.
7. **Connection requests are scarce** — only 30/day. Before sending one, check `pendingConnection` from the visit response: `"pending"` = already sent, skip. `"failed"` = failed today, skip. `"none"` = safe to send.
8. **Error handling** — wrap every SDK call in a shared helper. The wrapper must check the **error type** to branch:
   - `429`: wait `error.retryAfter` seconds, retry (max 3). All retries fail → skip profile, move on.
   - `400`/`409`/`410`: log, skip item, continue.
   - `401`: stop (auth broken).
   - `404`/`405`: stop — method/tool doesn't exist. Re-read the SDK resources table or tool list.
   - `500`: log, skip item, continue (transient).
   - Parse the response body inside a try/catch — malformed JSON must not crash the script.
   - Never treat a failed call as "success with empty result".
   - Never `process.exit()` on transient errors — skip and continue.

## Workflows

Each workflow is detailed in its sub-skill. Load the relevant sub-skill when needed.

- **Lead Magnet** — deliver a resource to everyone who engages with a post (comments, likes, invitations). → Lead Magnet sub-skill

More workflows coming soon. You can build your own using the SDK methods and tools listed above.

