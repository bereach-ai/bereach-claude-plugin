---
name: bereach-sdk-reference
description: "Complete SDK method reference — parameters, types, and descriptions for all BeReach operations."
lastUpdatedAt: 1772619338
---

# BeReach SDK Reference

> Auto-generated from OpenAPI spec. This is the single source of truth for SDK method signatures.
> Use `client.<resource>.<method>(params)` — see [BeReach Skill](bereach-skill.md) for behavioral rules.

## linkedinScrapers

### collectLikes

Scrape LinkedIn post likes. Returns paginated list of profiles who liked a post.

`client.linkedinScrapers.collectLikes(params)`

- **postUrl** (string, required) — LinkedIn post URL to inspect for reactions.
- **start** (integer, min 0) — Pagination offset (multiples of 200).
- **count** (integer, 0-200) — Number of likes to fetch per page (0-200, default 200). Use count=0 for a free total-only check.

### collectComments

Scrape LinkedIn post comments. Returns paginated list of commenters with comment text and URNs.

`client.linkedinScrapers.collectComments(params)`

- **postUrl** (string, required) — LinkedIn post URL to inspect for comments.
- **start** (integer, min 0) — Pagination offset (multiples of 100).
- **count** (integer, 0-100) — Number of comments to fetch per page (0-100, default 100). Use count=0 for a free total-only check.
- **campaignSlug** (string) — When provided, each profile includes actionsCompleted and knownDistance for campaign-aware scraping.

### collectCommentReplies

Scrape replies to a LinkedIn comment. Returns paginated replies for a specific comment URN.

`client.linkedinScrapers.collectCommentReplies(params)`

- **commentUrn** (string, required) — Comment URN returned by the comments endpoint (e.g., 'urn:li:comment:(activity:123456,789)').
- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 0-100) — Number of replies to fetch per page (0-100, default 100). Use count=0 for a free total-only check.

### collectPosts

Scrape LinkedIn profile posts. Returns paginated list of posts from a profile.

`client.linkedinScrapers.collectPosts(params)`

- **profileUrl** (string, required) — LinkedIn profile URL to fetch posts from.
- **count** (integer, 0-100) — Number of posts to fetch (0-100, default 20). Use count=0 for a free total-only check.
- **start** (integer, min 0) — Pagination offset (default 0).
- **paginationToken** (string) — Pagination token from a previous response to fetch the next page.

### visitProfile

Visit LinkedIn profile and extract contact data. Distance-1 profiles cached 24h. No dedup — always executes.

`client.linkedinScrapers.visitProfile(params)`

- **profile** (string, required) — LinkedIn profile URL or profile URN.
- **campaignSlug** (string) — Optional campaign identifier for tracking only. No dedup — visit always executes.
- **includePosts** (boolean) — When true, fetches the last 5 posts from the profile. Defaults to false.

### visitCompany

Visit LinkedIn company page and extract profile data including description, industry, employee count, headquarters, and more.

`client.linkedinScrapers.visitCompany(params)`

- **companyUrl** (string, required) — LinkedIn company URL (e.g., 'https://www.linkedin.com/company/openai') or universal name (e.g., 'openai').
- **includeWorkplacePolicy** (boolean) — Include workplace policy data such as hybrid/remote status and benefits. Costs 1 extra credit.

## linkedinSearch

### unifiedSearch

Unified LinkedIn Search — search posts, people, companies, or jobs with a single endpoint. Supports all filter types via category selection.

`client.linkedinSearch.unifiedSearch(params)`

- **category** (string) — Type of search to perform (required unless url is provided). Values: posts, people, companies, jobs.
- **url** (string) — LinkedIn search URL — category and filters are extracted automatically.
- **keywords** (string) — Search keywords. Supports LinkedIn Boolean syntax.
- **sortBy** (string) — Sort order (posts & jobs). Values: relevance, date.
- **datePosted** (string) — Time filter (posts & jobs). Values: past-24h, past-week, past-month.
- **contentType** (string) — Media type filter (posts only). Values: images, videos, documents.
- **authorIndustry** (string[]) — Author industry IDs (posts only, resolve via /search/parameters).
- **authorCompany** (string[]) — Author company IDs (posts only, resolve via /search/parameters).
- **connectionDegree** (string[]) — Connection degree: F=1st, S=2nd, O=3rd+ (people only). Values: F, S, O.
- **firstName** (string) — First name filter (people only).
- **lastName** (string) — Last name filter (people only).
- **title** (string) — Job title filter, supports OR syntax (people only).
- **connectionOf** (string) — Profile URN to find connections of (people only).
- **profileLanguage** (string[]) — Profile language codes e.g. ['en','fr'] (people only).
- **school** (string[]) — School IDs (people only, resolve via /search/parameters).
- **location** (string[]) — Geo IDs (people, companies, jobs — resolve via /search/parameters).
- **industry** (string[]) — Industry IDs (people, companies — resolve via /search/parameters).
- **currentCompany** (string[]) — Current company IDs (people only, resolve via /search/parameters).
- **pastCompany** (string[]) — Past company IDs (people only, resolve via /search/parameters).
- **companySize** (string[]) — Company size: A=1-10, B=11-50, C=51-200, D=201-500, E=501-1K, F=1K-5K, G=5K-10K, H=10K+, I=self (companies only). Values: A, B, C, D, E, F, G, H, I.
- **jobType** (string[]) — Job type: F=Full-time, P=Part-time, C=Contract, T=Temporary, I=Internship, V=Volunteer, O=Other (jobs only). Values: F, P, C, T, I, V, O.
- **experienceLevel** (string[]) — Experience level: 1=Internship, 2=Entry, 3=Associate, 4=Mid-Senior, 5=Director, 6=Executive (jobs only). Values: 1, 2, 3, 4, 5, 6.
- **workplaceType** (string[]) — Workplace type: 1=On-site, 2=Remote, 3=Hybrid (jobs only). Values: 1, 2, 3.
- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-50) — Results per page (default 10, max 50).

### searchPosts

Search LinkedIn posts by keywords with optional filters for date, content type, author industry, and author company.

`client.linkedinSearch.searchPosts(params)`

- **keywords** (string, required) — Search keywords (required). Supports LinkedIn Boolean syntax.
- **url** (string) — Optional LinkedIn search URL. Explicit params override URL-derived values.
- **sortBy** (string) — Sort order. Values: relevance, date.
- **datePosted** (string) — Filter by publication date. Values: past-24h, past-week, past-month.
- **contentType** (string) — Filter by media type. Values: images, videos, documents.
- **authorIndustry** (string[]) — Author industry IDs (resolve via bereach_resolve_parameters).
- **authorCompany** (string[]) — Author company IDs (resolve via bereach_resolve_parameters).
- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-50) — Results per page (default 10, max 50).

### searchPeople

Search LinkedIn people by keywords, connection degree, name, title, location, industry, company, and school.

`client.linkedinSearch.searchPeople(params)`

- **keywords** (string) — Search keywords. Matches against name, headline, company, skills, and bio.
- **url** (string) — Optional LinkedIn search URL.
- **connectionDegree** (string[]) — Connection degree: F=1st, S=2nd, O=3rd+. Values: F, S, O.
- **firstName** (string) — Filter by first name (case-insensitive).
- **lastName** (string) — Filter by last name (case-insensitive).
- **title** (string) — Filter by job title. Supports OR syntax with '|' separator.
- **connectionOf** (string) — Profile URN to find connections of.
- **profileLanguage** (string[]) — Profile language codes (ISO 639-1).
- **school** (string[]) — School IDs (resolve via bereach_resolve_parameters).
- **location** (string[]) — Geo IDs (resolve via bereach_resolve_parameters).
- **industry** (string[]) — Industry IDs (resolve via bereach_resolve_parameters).
- **currentCompany** (string[]) — Current company IDs (resolve via bereach_resolve_parameters).
- **pastCompany** (string[]) — Past company IDs (resolve via bereach_resolve_parameters).
- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-50) — Results per page (default 10, max 50).

### searchCompanies

Search LinkedIn companies by keywords, location, industry, and company size.

`client.linkedinSearch.searchCompanies(params)`

- **keywords** (string) — Search keywords. Matches against company name, description, and specialties.
- **url** (string) — Optional LinkedIn search URL.
- **location** (string[]) — Geo IDs (resolve via bereach_resolve_parameters).
- **industry** (string[]) — Industry IDs (resolve via bereach_resolve_parameters).
- **companySize** (string[]) — Employee count: A=1-10, B=11-50, C=51-200, D=201-500, E=501-1K, F=1K-5K, G=5K-10K, H=10K+, I=self. Values: A, B, C, D, E, F, G, H, I.
- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-50) — Results per page (default 10, max 50).

### searchJobs

Search LinkedIn jobs by keywords, location, job type, experience level, and workplace type.

`client.linkedinSearch.searchJobs(params)`

- **keywords** (string) — Search keywords. Matches against job title, company name, and description.
- **url** (string) — Optional LinkedIn search URL.
- **location** (string[]) — Geo IDs (resolve via bereach_resolve_parameters).
- **datePosted** (string) — Filter by posting date. Values: past-24h, past-week, past-month.
- **sortBy** (string) — Sort order. Values: relevance, date.
- **jobType** (string[]) — Employment type: F=Full-time, P=Part-time, C=Contract, T=Temporary, I=Internship, V=Volunteer, O=Other. Values: F, P, C, T, I, V, O.
- **experienceLevel** (string[]) — Seniority: 1=Internship, 2=Entry, 3=Associate, 4=Mid-Senior, 5=Director, 6=Executive. Values: 1, 2, 3, 4, 5, 6.
- **workplaceType** (string[]) — Workplace: 1=On-site, 2=Remote, 3=Hybrid. Values: 1, 2, 3.
- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-50) — Results per page (default 10, max 50).

### searchByUrl

Search LinkedIn by URL. Automatically extracts category, keywords, and filters from a LinkedIn search URL.

`client.linkedinSearch.searchByUrl(params)`

- **url** (string, required) — A LinkedIn search URL. Category and filters are extracted automatically.
- **start** (integer, min 0) — Override pagination offset.
- **count** (integer, 1-50) — Override results per page (default 10, max 50).

### resolveParameters

Resolve text to LinkedIn search parameter IDs (typeahead). Prerequisite for using location, industry, company, or school filters.

`client.linkedinSearch.resolveParameters(params)`

- **type** (string, required) — Parameter type to resolve. Values: GEO, COMPANY, INDUSTRY, SCHOOL, CONNECTIONS, PEOPLE.
- **keywords** (string, required) — Text to resolve into LinkedIn IDs.
- **limit** (integer, 1-50) — Max results (default 10, max 50).

## linkedinActions

### connectProfile

Send LinkedIn connection request. Deduplicates by profile within a campaign.

`client.linkedinActions.connectProfile(params)`

- **profile** (string, required) — LinkedIn profile URL or profile URN.
- **campaignSlug** (string) — Campaign identifier for deduplication.

### listInvitations

List received LinkedIn connection invitations. Returns pending invitations with IDs needed to accept them.

`client.linkedinActions.listInvitations(params)`

- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-100) — Number of invitations to return (default 10, max 100).

### acceptInvitation

Accept a LinkedIn connection invitation. Requires invitationId and sharedSecret from the list invitations endpoint.

`client.linkedinActions.acceptInvitation(params)`

- **invitationId** (string, required) — Invitation ID (from the list invitations endpoint).
- **sharedSecret** (string, required) — Shared secret / validation token (from the list invitations endpoint).
- **senderProfileId** (string) — Sender's profile ID. Recommended for reliability.
- **firstName** (string) — Sender's first name. Recommended for reliability.
- **lastName** (string) — Sender's last name. Recommended for reliability.

### sendMessage

Send LinkedIn message. Rate limited to 80 messages per day.

`client.linkedinActions.sendMessage(params)`

- **profile** (string, required) — LinkedIn profile URL or profile URN.
- **message** (string, required) — Message content to send.
- **campaignSlug** (string) — Campaign identifier for deduplication.

### replyToComment

Reply to a LinkedIn comment. Use the commentUrn from the comments endpoint directly.

`client.linkedinActions.replyToComment(params)`

- **commentUrn** (string, required) — LinkedIn comment URN (e.g., 'urn:li:comment:(activity:123,456)').
- **message** (string, required) — Reply message text.
- **campaignSlug** (string) — Campaign identifier for deduplication.

### likeComment

Like (react to) a LinkedIn comment. Use the commentUrn from the comments endpoint directly.

`client.linkedinActions.likeComment(params)`

- **commentUrn** (string, required) — LinkedIn comment URN.
- **reactionType** (string) — Reaction type (default: LIKE). Values: LIKE, LOVE, CELEBRATE, SUPPORT, FUNNY, INSIGHTFUL.
- **campaignSlug** (string) — Campaign identifier for deduplication.

### publishPost

Publish or schedule a LinkedIn post. Supports text, images, mentions, and visibility control.

`client.linkedinActions.publishPost(params)`

- **text** (string, required) — Post commentary text.
- **mode** (string, required) — Publish mode: 'instant' publishes immediately, 'scheduled' schedules for later. Values: scheduled, instant.
- **scheduledAt** (integer) — Timestamp in milliseconds for scheduled posts (required when mode='scheduled').
- **imageUrl** (string) — URL of an image to attach to the post.
- **visibility** (string) — Post visibility (default: ANYONE). Values: ANYONE, CONNECTIONS.
- **mentions** (object[]) — Profile mentions with text positions.
- **campaignSlug** (string) — Campaign identifier for deduplication.

## linkedinChat

### listConversations

List LinkedIn inbox conversations with participants, last message, and read status.

`client.linkedinChat.listConversations(params)`

- **nextCursor** (string) — Pagination cursor from a previous response.

### searchConversations

Search LinkedIn inbox conversations by keyword. 0 credits.

`client.linkedinChat.searchConversations(params)`

- **keywords** (string, required) — Search keywords.
- **nextCursor** (string) — Pagination cursor from a previous response.

### findConversation

Find a conversation with a specific person. Direct O(1) lookup via LinkedIn's compose API. 0 credits.

`client.linkedinChat.findConversation(params)`

- **profile** (string, required) — Profile URL or URN for direct conversation lookup.
- **includeMessages** (boolean) — If true, also return the conversation's recent messages (0 extra credits). Default: false.

### getMessages

Read messages from a LinkedIn conversation. 0 credits.

`client.linkedinChat.getMessages(params)`

- **conversationUrn** (string, required) — Full conversation URN as returned by list/search conversations.
- **deliveredAt** (integer) — Timestamp (ms) of the oldest message from previous page — pass this to load older messages.

## profile

### getLinkedInProfile

Get authenticated user's LinkedIn profile from the database. No LinkedIn API call, 0 credits.

`client.profile.getLinkedInProfile(params)`

No parameters.

### refresh

Refresh authenticated user's LinkedIn profile by fetching latest data from LinkedIn.

`client.profile.refresh(params)`

No parameters.

### getPosts

Get authenticated user's LinkedIn posts.

`client.profile.getPosts(params)`

- **count** (integer, 1-100) — Number of posts to fetch (default 20, max 100).
- **start** (integer, min 0) — Pagination offset (default 0).
- **paginationToken** (string) — Pagination token from a previous response.

### getFollowers

Get authenticated user's LinkedIn followers.

`client.profile.getFollowers(params)`

- **start** (integer, min 0) — Pagination offset (default 0).
- **count** (integer, 1-50) — Number of followers to fetch per page (default 10, max 50).

### getLimits

Get current LinkedIn rate limit status for all action types. 0 credits.

`client.profile.getLimits(params)`

No parameters.

### getCredits

Get current BeReach credit balance including used, remaining, and percentage. 0 credits.

`client.profile.getCredits(params)`

No parameters.

## campaigns

### getStatus

Query per-profile action status within a campaign. Returns which actions have been completed for each profile. 0 credits.

`client.campaigns.getStatus(params)`

- **campaignSlug** (string, required) — Campaign identifier.
- **profiles** (string[], required) — LinkedIn profile URLs or URNs to check status for.

### syncActions

Mark actions as completed without performing them on LinkedIn. Use when actions were performed outside the API. 0 credits.

`client.campaigns.syncActions(params)`

- **campaignSlug** (string, required) — Campaign identifier.
- **profiles** (object[], required) — Profiles and actions to mark as completed. Values: message, reply, like, visit, connect.

### getStats

Get aggregate campaign statistics: per-action counts, unique profiles, and total credits used. 0 credits.

`client.campaigns.getStats(params)`

- **campaignSlug** (string, required) — Campaign identifier.
