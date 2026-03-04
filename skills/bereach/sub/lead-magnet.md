---
name: bereach-lead-magnet
description: "Lead magnet workflow — deliver a resource to everyone who engages with a LinkedIn post."
lastUpdatedAt: 1772619338
---

# BeReach Lead Magnet Skill

> Sub-skill of [BeReach](bereach-skill.md). Deliver a resource to everyone who engages with a LinkedIn post.

## Prerequisites

Requires [BeReach SKILL.md](bereach-skill.md) (constraints, SDK setup, anti-hallucination rules) and [SDK Reference](sdk-reference.md) for method signatures and parameters. Do NOT generate code without both loaded.

## Script generation rules

All lead magnet scripts MUST be TypeScript using the `bereach` SDK (see main skill for SDK setup and rules).

## Tone

All script output and chatbot messages are **for the end user**. Write like you're talking to a non-technical person. Never mention internal concepts like "Track A", "Layer 2", "DM guard", "server-side dedup", "retryAfter", "URN", "O(1) lookup", etc. The campaign slug is fine to show — it helps the user know which campaign is running. Just say what happened in plain language: "Sent the resource to 12 people", "Skipped 3 (already received it)", "Outside active hours, will resume at 7:00".

## Objective

**Every person who engages with the post must receive the resource via DM as soon as they are distance 1.** Comment, like, invitation — the channel doesn't matter. The person gets the resource the moment a DM is possible.

## Configuration

Only ask for these on activation. If `POST_URL` is omitted, call `getPosts` and let the user pick.

```
POST_URL=<LinkedIn post URL>
CAMPAIGN_SLUG=<unique name, e.g. "lm-feb-2026">
DM_TEMPLATE=<DM with {firstName} and {link} placeholders, e.g. "{firstName}, voici la ressource: {link}">
RESOURCE_LINK=<URL to share, or empty if DM is plain text>
```

`DM_TEMPLATE` supports `{firstName}` (commenter's first name) and `{link}` (the `RESOURCE_LINK` value). For richer personalization (headline, company), use visit data from the profile response when available.

Ask the user if their DM includes a link to share. If yes, store it in `RESOURCE_LINK` — this enables the DM guard (inbox dedup). If no link, set `RESOURCE_LINK` to empty — the DM guard is disabled and only server-side `campaignSlug` dedup applies.

## Tracking

BeReach is the source of truth for action tracking (`campaignSlug` dedup).

For the incremental comment check (`previousTotal`), use a local state file **per campaign** as a fallback — this survives process restarts and avoids re-processing if the cache resets. Only store the comment total counter locally; all action tracking stays on BeReach.

- **Check status**: `getStatus` with `campaignSlug` and `profiles` → returns per-profile booleans (`message`, `reply`, `like`, `visit`, `connect`)
- **Manual sync**: `syncActions` with `campaignSlug` and `profiles: [{ profile, actions }]` → mark actions as completed without performing them (e.g. manual DMs done outside the bot)
- **Automatic dedup**: every action with `campaignSlug` is tracked by BeReach. Duplicates return `duplicate: true` and cost nothing.

## Pre-flight

All 3 scripts must perform pre-flight at the start of each run:

1. `getLinkedInProfile` → cache own profileUrl/URN
2. First run of day: `getLimits` → check remaining quotas

## Architecture

Split into 3 independent scripts. Each script:

1. Acquires a file lock **scoped to the campaign slug + script** (e.g. using a lock file with `fs.open` exclusive flag). If locked, exit immediately — a previous run is still active.
2. Runs its logic.
3. Prints recap and releases lock on exit.

The 3 scripts share `CAMPAIGN_SLUG` — BeReach dedup ensures no action is performed twice across scripts.

### Multiple campaigns

A user can run lead magnets on several posts simultaneously. Each campaign has its own set of 3 scripts and 3 crons. Lock files and state files must include the campaign slug to avoid collisions between campaigns.

All campaigns share the same LinkedIn rate limits. If multiple campaigns run at the same time, they compete for the same quota. BeReach handles conflicts automatically.

### Script 1 — Comments

Scrape commenters and engage them.

1. **Check first** (mandatory): `collectComments` with `count: 0` → store `total`. On subsequent rounds, compare with previous `total`. If unchanged, skip this round. This saves credits and rate limit slots.
2. **Fetch**: `collectComments` with `campaignSlug` → response: `profiles[]` (NOT "comments"). Paginate: while `hasMore`, increment `start` by `count`.
3. Each profile has: `profileUrl`, `commentUrn`, `name`, `profileUrn`, `actionsCompleted: { message, reply, like, visit, connect }`, `knownDistance`, `hasReplyFromPostAuthor`
4. For each profile (skip own URN, skip `actionsCompleted.message === true`):
   a. If `knownDistance` is set, use it (skip visit, saves 1 credit). Otherwise: `visitProfile` with `campaignSlug` → `memberDistance`, `pendingConnection`
   b. Distance 1:
   - **DM dedup** (see "DM dedup" section): check both layers. If either says "already sent", skip. Otherwise: `sendMessage` with `campaignSlug`.
   - Skip reply if `actionsCompleted.reply === true` or `hasReplyFromPostAuthor === true`. Otherwise: `replyToComment` with `commentUrn` and `campaignSlug`
   - Skip like if `actionsCompleted.like === true`. Otherwise: `likeComment` with `commentUrn` and `campaignSlug`
   c. Distance 2+:
   - Skip reply if `actionsCompleted.reply === true` or `hasReplyFromPostAuthor === true`. Otherwise: `replyToComment` with `commentUrn` and `campaignSlug` (CTA to connect)
   - Skip like if `actionsCompleted.like === true`. Otherwise: `likeComment` with `commentUrn` and `campaignSlug`
   - Do NOT call `connectProfile`. Script 2 delivers the resource when they connect.

### Script 2 — Invitations

Accept pending invitations and deliver the resource.

- `listInvitations` → response: `invitations[]`. Each has: `invitationId`, `sharedSecret`, `fromMember: { name, profileUrl }`. Paginate: while `total > start + count`.
- For each invitation:
  1. `acceptInvitation` with `invitationId` and `sharedSecret` — only these 2 required params
  2. `visitProfile` with `fromMember.profileUrl` and `campaignSlug`
  3. If `memberDistance === 1`: **DM dedup** (see "DM dedup" section) → DM if not already sent.

### Script 3 — New connections

Check if distance 2+ commenters have now connected.

- Re-fetch commenters: `collectComments` with `campaignSlug` — paginate all
- Filter: `actionsCompleted.message === false` and `knownDistance > 1` (skip profiles where `knownDistance` is null — they haven't been visited yet and belong to Script 1)
- For each: `visitProfile` with `campaignSlug` → check `memberDistance` and `pendingConnection`
  - `memberDistance === 1`: **DM dedup** (see "DM dedup" section) → DM the resource if not already sent.
  - `pendingConnection === "pending"`: not connected yet, skip

## Dedup

### Reply guard

`hasReplyFromPostAuthor` from the comment scrape (0 credits). If `true`, skip reply.

### DM dedup (2 layers)

Check in order. If either says "already sent", skip the DM.

**Layer 1 — `actionsCompleted.message`** (free, within campaign). Already checked in the script loop — `true` → skip.

**Layer 2 — `/find` inbox lookup** (0 credits, cross-campaign). Only when `RESOURCE_LINK` is set. Call `findConversation` with `includeMessages: true` → `{ found, messages }`. `messages` is at the **top level**, not nested. If any message text contains `RESOURCE_LINK`: skip DM, call `syncActions` to mark done. If found but no link in messages: pass `messages` as context for DM tone adaptation. The function must return both skip decision and messages.

**Fail-safe**: if `findConversation` errors or all retries fail, **skip this profile** — do NOT send the DM. A failed lookup treated as "not found" causes duplicates. The profile will be retried next cron run.

## Language

For lead magnet, the default language is the **post language** — everyone commenting on a French post speaks French. Detect it from the post content, or let the user specify it. This overrides the general user-language detection from `getLinkedInProfile` (which still applies for other workflows outside lead magnet).

## Comment replies

- Distance 1: acknowledge, tell them you're sending it / it's in their DMs
- Distance 2+: tell them to connect with you to receive it. Never mention DMs.
- Language = post language (see above). The `generateReply()` function should accept a `language` parameter and maintain separate pools per language (at minimum FR and EN).
- Short, casual, 3-5 words.
- Use random selection over a pool of varied templates (at least 5 per distance per language). Vary phrasing and tone to avoid LinkedIn spam detection.

## DM content

- Use `DM_TEMPLATE` as the **base pattern** — same meaning, same `{link}`, but **always randomize phrasing**. Generate at least 10 variations of the template at boot (different wording, different greeting, different emoji, different sentence structure). Pick a random variation for each DM. Never send the same literal text twice in a row.
- Language = post language.
- **When conversation history exists** (DM guard fetched messages, no `RESOURCE_LINK` found): adapt the tone — more casual, shorter. The conversation messages are context.

## Time window

Scripts only run between **7:00 and 23:00** in the user's timezone (default). At the start of each run, check the current local time. If outside the window, exit immediately with a short log ("Outside active hours, skipping"). The user can override the window if they ask.

Detect timezone from `getLinkedInProfile` → `location` (e.g. "Paris, France" → Europe/Paris). If ambiguous, ask the user once and store it in the campaign state file.

## Pacing

Follow pacing rules from the main skill (Constraints #2). No exceptions — sleep after every SDK call.


### Pause / Resume

The user can pause or resume any script independently. When multiple campaigns are active, each campaign's scripts are paused/resumed separately.

### Recap (print after each run)

At the end of every run, call these two methods and print the recap:

1. `getStats` with `campaignSlug` → `{ stats: { message, reply, like, visit, connect }, totalProfiles, creditsUsed }`
2. `getCredits` → `{ credits: { current, limit, remaining, percentage } }`

Human-friendly, no jargon. Format:

    {slug} — Recap
    {totalProfiles} people reached
    {stats.message} DMs sent · {stats.reply} replies · {stats.like} likes
    Credits: {credits.current} used · {credits.remaining}/{credits.limit} remaining

    You can ask me to disable features to save credits.

### Toggleable features

All enabled by default. The user can ask to disable any of these to save credits:

- **Visits** for profiles with knownDistance already set (saves 1 credit/profile)
- **Likes** on comments (saves 1 credit/comment)
- **Connection re-checks** (saves visit + DM credits)
- **Comment replies** (saves 1 credit/comment)
- **DM guard** — conversation lookup before DMing (0 credits, auto-disabled when no `RESOURCE_LINK`)
- **Reply guard** — hasReplyFromPostAuthor check (free, but can be skipped if user wants to re-reply)
