---
name: redreplier
description: >
  Find and triage leads from public conversations on Reddit, Hacker News, X (Twitter) and Bluesky
  through the RedReplier MCP tools. RedReplier matches the user's keywords and scores every mention
  0-100 for relevance. Use this skill WHENEVER the user asks about new mentions or leads, wants the
  best conversations to reply to, asks why a mention scored high or low, wants a reply drafted for a
  thread, wants to approve or reject mentions, add or change the websites and keywords RedReplier
  watches, or set mention email alerts. Trigger it even for short asks like "any new leads?", "what
  are people saying about us on Reddit", "draft a reply to that HN thread", "add the keyword
  'hubspot alternative'" or "stop tracking that keyword".
last-updated: 2026-09-27
---

# RedReplier

RedReplier watches Reddit, Hacker News, X and Bluesky for the user's keywords and scores every
matched mention 0-100 for relevance to their product. You work the account through the redreplier
MCP server's tools. The tool prefix depends on how the plugin was installed, for example
`mcp__plugin_redreplier_redreplier__list_mentions`. Each tool carries its own parameter
descriptions. This skill is the operating logic on top of them.

If more than 30 days have passed since `last-updated`, tell the user this skill may be out of date
and that `/plugin marketplace update` pulls the latest version.

## Sign-in

The plugin connects to `https://mcp.redreplier.com/mcp` and signs in with OAuth. There is no API
key to set up.

- If the redreplier tools are missing, or a call returns 401 or "Authentication required", tell the
  user to run `/mcp`, pick `redreplier` and sign in with the email they use on RedReplier. Stop and
  retry the original request once they say it is done.
- Never ask for an API key in chat. Never look for keys in environment variables, config files or
  anywhere else on the machine. If the user pastes a key anyway, tell them to revoke it at
  RedReplier and use the `/mcp` sign-in instead.
- A sign-in reaches every workspace the user belongs to, each with its own websites, keywords
  and mentions. `list_workspaces` returns them with `id`, `name`, `organization`, `role`,
  `isDefault` and `current`. Every other tool takes an optional `workspaceId`; without it the
  tool works in the default workspace. Call `list_workspaces` when the user names a workspace,
  brand or client, or when websites they expect are missing, then pass the same `workspaceId`
  on every call about that workspace. IDs from one workspace do not exist in another.

## Tools by job

| Job | Tools |
|-----|-------|
| Which workspace | `list_workspaces` |
| Find leads | `list_mentions`, `count_mentions`, `explain_mention` |
| Triage | `update_mention_status` |
| Websites | `list_websites`, `get_website`, `create_website`, `update_website`, `delete_website`, `analyze_website` |
| Keywords | `add_keywords`, `edit_keyword`, `disable_keyword`, `enable_keyword`, `delete_keyword`, `keyword_change_usage` |
| Alerts | `get_alert_settings`, `update_alert_settings` |

Start with `list_websites` when you need website or keyword IDs.

## How RedReplier works

1. **Websites.** Each monitored website has a description. The AI scores every new mention against
   it. A site with no description gets unscored mentions ("Scoring skipped: website description
   missing").
2. **Keywords.** Each website has keywords that decide what gets matched. Statuses:
   - `PENDING`: proposed, not live, matches nothing.
   - `ACTIVE`: live and monitored.
   - `DISABLED`: paused. Keeps its stored mentions.
   - `SUSPENDED`: the grader judged it too noisy. Fix it with `edit_keyword`.
3. **Plan limits.** The plan sets how many keywords can be active and how many websites the account
   can have. Keywords over the limit stay `PENDING`. No tool charges, upgrades or changes the
   subscription. Only the user can change the plan, in the RedReplier app.
4. **Mentions.** Matched posts and comments, each with a score, source, matched keyword, content and
   `url`. On request RedReplier adds a relevance reason, tags and a drafted reply. You triage them
   as `APPROVED` (real lead), `REJECTED` (noise) or `NEW` (inbox).
5. **Alerts.** Optional email digests of new mentions, sent on a cadence the plan allows.

## Finding leads

1. Call `list_mentions` with `statuses: ["NEW"]` and `sort: "RELEVANCE"`. Use `sort: "RECENT"` for
   "what's new". For "since yesterday" questions pass `from` as an ISO 8601 datetime. `from` and
   `to` filter on when RedReplier found the mention, not when it was posted.
2. Read each mention's `title`, `contentText` and `relevanceScore` against the website description.
   The score is a first pass, not the verdict.
3. For the few worth acting on, call `explain_mention`. It returns `relevanceReason`, `tags` and a
   drafted reply in `aiReplySuggestion`, generating them on first call. It is slow, so never run it
   across a whole list. It needs the website to have a description, and returns `null` for an
   unknown ID.
4. Present the leads (see Output style) and draft replies on request.

Useful filters on `list_mentions` and `count_mentions`:

- `websiteId`, `keywords` (exact keyword values, case-insensitive) and `sources`: `REDDIT_POST`,
  `REDDIT_COMMENT`, `TWITTER` (X), `BLUESKY`, `HACKERNEWS`.
- `scoreBuckets`: `VERY_LOW` (under 10), `LOW` (10-29), `MEDIUM` (30-49), `HIGH` (50-74),
  `VERY_HIGH` (75+).
- `minScore` (0-100) keeps mentions scoring at least that much and drops unscored ones.
- `list_mentions` pages with `limit` (1-500, default 50) and `offset`. It returns
  `{ mentions, total, limit, offset }`. Keep paging while `offset < total`.
- `count_mentions` returns `{ total }` only. Skip it when you are fetching rows anyway.

Two defaults hide rows. REJECTED mentions are left out unless `statuses` names them. Mentions under
the website's minimum score (30 unless changed) are hidden unless `includeLowRelevance` is true.
`scoreBuckets` with `LOW` or `VERY_LOW` and `minScore` below 30 do not lift that cutoff on their
own. If the user asks where a mention went, check both defaults.

`subreddit` is only set for Reddit sources.

## Triage

`update_mention_status` takes `mentionId` and `status` (`APPROVED`, `REJECTED` or `NEW`). Any status
can move to any other, so triage is fully reversible. Triage what the user asks you to and never
approve a mention you have not read. Call `explain_mention` first when a score looks wrong for a
borderline lead. When the user keeps rejecting the same kind of mention, suggest tightening the
keyword or the website description.

## Drafting replies

Read `references/replies.md` before drafting. RedReplier never posts, and neither do you. The user
posts the reply on the network in their own name.

## Websites

- `list_websites` is not a plain read. It also activates pending keywords that fit the plan and
  queues searches. It never charges. Use `get_website` (`websiteId`) to re-read one site you know.
- `create_website` takes `url` and optional `name`, `keywords` and `description`. A domain already
  on the account returns 400. Re-adding a deleted domain revives the old record. Without a
  description the server scrapes the URL to write one, spending one AI generation from the monthly
  quota. If the response shows `description: null`, draft one with `analyze_website` and save it
  with `update_website`. Initial keywords are stored `PENDING` until `list_websites` or
  `add_keywords` promotes the ones that fit.
- `analyze_website` (`url`) returns `{ description }` without changing anything. It spends one AI
  generation from the monthly quota. Show the draft and save it after a "yes".
- `update_website` takes `websiteId` and optional `name` and `description`. Omitted fields keep
  their value, and an empty description clears it. Mentions already scored are not rescored. When
  scores look wrong across the board, a vague description is the usual cause.
- `delete_website` (`websiteId`) stops all monitoring for the site. There is no restore tool.
  Confirm with the user first and name the domain, not only the ID.

## Keywords

- `add_keywords` takes `websiteId` and `keywords` (max 255 characters each). Values are trimmed,
  lowercased and deduplicated. The ones that fit the plan go `ACTIVE`, the rest stay `PENDING`. It
  returns the whole website, so check the new statuses there and tell the user which keywords are
  still `PENDING` because the plan is full.
- `edit_keyword` takes `keywordId` and `value`. It keeps the ID, re-grades the new text and is the
  fix for a `SUSPENDED` keyword. Prefer it to adding a near-duplicate.
- `disable_keyword` (`keywordId`) pauses a keyword and keeps its mentions. `enable_keyword`
  (`keywordId`) resumes it: `ACTIVE` if the plan has room, otherwise `PENDING`. Neither charges.
- `delete_keyword` (`keywordId`) erases the keyword in any status and every mention it produced.
  There is no undo. Prefer disabling, and confirm by keyword name before deleting.
- `keyword_change_usage` reports the monthly edit allowance as `{ limit, used, remaining,
  unlimited }`. Every current plan reports unlimited (`limit: -1`), so there is no need to check it
  before `edit_keyword`.

Keyword advice lives in `references/keywords.md`.

## Alerts

Call `get_alert_settings` first. It returns `enabled`, `cadenceMinutes`, `minIntervalMinutes` (the
fastest cadence the plan allows) and `availableCadences`. Pick a value from `availableCadences`.
Valid cadences are 15, 30, 60, 120, 180, 240, 720 and 1440 minutes. A value faster than the plan
allows is raised to the plan's floor.

`update_alert_settings` takes `enabled` (required) and `cadenceMinutes`. It replaces both settings
on every call. Omitting `cadenceMinutes` resets it to the fastest cadence allowed, so pass the
current value when only switching alerts on or off. Call `get_alert_settings` afterwards to confirm
what applied.

## Errors

Failed calls return text starting with `Error: API error:` followed by the server message.

- 401, or the tools are missing: the sign-in is missing or expired. Send the user to `/mcp` as
  described in Sign-in.
- 401 with `token_issuer_lost_access`: the member behind the connection left the workspace or was
  deactivated. The user needs to sign in again through `/mcp` with an account that still has access.
- 403 with `permission_denied`: the user's workspace role cannot use RedReplier. Admin and Editor
  can. A Contributor or Viewer needs a workspace admin to change their role.
- 400: usually a plan limit (no website slots left, AI generation quota used up), a duplicate
  domain or keyword, or an invalid ID or URL. Report the message. Do not retry the same call.
- 403 with `workspace_access_denied`: the `workspaceId` is not one this sign-in reaches. Call
  `list_workspaces` and pick an id from it.
- 404: the ID is unknown in this workspace. Re-read IDs with `list_websites`, and check you passed
  the same `workspaceId` the ID came from.
- 429: rate limited. Wait before trying again and do not loop.

## Output style

- Leads: one short block per lead, best first. Network (and subreddit), title, score, one line on
  why it is a lead, and the `url`.
- Replies: the draft on its own, ready to paste, then one line on what it answers.
- Counts and summaries: the number first, then the breakdown by network or keyword.
- Keyword changes: each keyword with its resulting status, and a note on any left `PENDING`.
