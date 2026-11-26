---
name: publishbuddy
description: "Draft, check and schedule social media posts with the PublishBuddy MCP tools, and report how posts are doing. Use when the user wants to write, plan, schedule, queue, publish, fix or delete posts for their social accounts (Facebook, Instagram, LinkedIn, X, TikTok, YouTube, Threads, Bluesky, Pinterest, Telegram), add pictures or video to a post, see what is scheduled or failed, or read their social analytics. Posts stay drafts until the user says to schedule or publish them."
compatibility: "Needs the PublishBuddy MCP server (https://api.publishbuddy.com/mcp) connected and a PublishBuddy account. Sending files from disk needs a shell with curl."
---

# PublishBuddy

PublishBuddy is a social media management tool. Through its MCP server you can read a workspace's social accounts and its brand, look at its analytics, save posts as drafts, and schedule or publish them once the user says so.

## The rule that comes first

**You draft and the user approves. Nothing goes out without their OK.**

- Every post you create is a draft. It stays a draft until the user tells you to schedule or publish it.
- Before you schedule or publish, show the user the final text, the account, the media and the time. Wait for a clear yes. A yes covers the posts you showed and nothing else.
- If the user asks you to write and schedule in one go, still show the drafts before you schedule them.
- `schedule_post` and `publish_now` take `confirm: true`. Send it only after that yes. Without it, most workspaces keep the post as a draft and the answer says `kept_as_draft`. That's the expected result. Tell the user the draft is ready to review. Never add `confirm: true` on your own to get past it.
- `publish_now` and `delete_post` take two calls. The first changes nothing and returns a preview and a `confirmation_token` that lasts five minutes. Show the preview, get a yes, then call again with the same arguments plus the token.
- `review_post` with `decision: "approve"` lets a post publish on its own at its time. Ask first.
- Publishing can't be undone from PublishBuddy. Deleting can't be undone either.

## Before you start

These steps use the PublishBuddy MCP tools (`list_profiles`, `create_post`, `schedule_post` and the rest). Your app may show them with a prefix, such as `mcp__publishbuddy__create_post`.

If you have no such tools, the server isn't connected. Ask the user to add the MCP server `https://api.publishbuddy.com/mcp` in their agent's settings and sign in with their PublishBuddy account when the browser opens. The steps for each app are at https://docs.publishbuddy.com/mcp-server/connect. Sign-in happens in the browser. Never ask the user to paste a password or an API key into the chat.

Never ask the user for an ID. Find accounts with `list_profiles`, posts with `list_posts`, media with `list_media` and workspaces with `list_workspaces`.

## The order of work

1. **See where things stand.** Call `get_workspace_overview`: connected accounts, what's scheduled, failed or waiting for approval, the AI credit balance and the daily post cap. Call `list_profiles` for the `profile_id` of each account and its connection status. If no account is connected, tell the user to connect one in the PublishBuddy app (app.publishbuddy.com) and stop there.
2. **Read the brand.** `get_brand_kit` (pass `profile_id`) returns the brand's voice, audience, products, keywords and emoji preference. `get_user_preferences` returns the tone and hashtag habits the user saved. Write in that voice. When the user tells you a preference to remember, save it with `save_user_preferences`.
3. **Read the numbers.** `get_analytics` is free. `report: "top_posts"` shows what worked and `best_times` shows when the audience responds. `hashtags` ranks the tags. Use them to pick the angle and the time.
4. **Draft.** Write one version per network when the networks differ in length or style. Keep facts to what the user or the brand kit gave you. Don't invent offers, prices, dates, quotes or reviews. Ask.
5. **Check.** `validate_post` takes the same arguments as `create_post` and saves nothing. It returns `{ valid, errors }`, and each error carries a `fix` and sometimes `candidates` to choose from. Fix every error before you save.
6. **Save the draft.** `create_post` with `profile_id`, or `profile_ids` for one draft per account (they share a `group_id`). If you have a good time from `best_times`, pass `suggested_publish_at` and a short `suggested_reason`. That only suggests a time the user can accept in the app. It schedules nothing.
7. **Show and ask.** Give the user each draft: text, account, media, and the time you propose. Ask what to change and whether to schedule.
8. **Schedule on their yes.** `schedule_post` with `confirm: true` and one `mode`:
   - `at`: a fixed time in `scheduled_at`. Also moves a post that's already scheduled.
   - `queue_top` or `queue_bottom`: the account's queue, which publishes in its next free timeslot. `get_profile_options` with `option: "queue_times"` shows the slots.
   - `unschedule`: back to draft.

   Use `publish_now` only when the user asked to publish right away. In a team that reviews posts, `review_post` with `decision: "submit"` sends a draft for approval instead. `set_queue_times` changes an account's queue slots. It moves posts that already wait in the queue, so ask before you use it.
9. **Report back.** Say what was scheduled, on which account, and when, with the timezone. Say what's still a draft.

To change a draft or a waiting post, use `update_post`. Only what you send changes. Published posts can't be edited here.

## Times and workspaces

- Send every time with an offset, such as `2026-12-01T09:00:00+00:00`. A time without one is read in the workspace owner's timezone.
- Date ranges in `list_posts` are calendar days (`YYYY-MM-DD`) in the user's timezone.
- Each call works in one workspace: the one picked when the app was connected. An app connected to all workspaces takes `workspace_id` on every tool. When the user names a client or brand, find its id with `list_workspaces` and pass it on every call for that client. If it isn't clear which workspace they mean, ask. Each answer names the workspace it acted in, so check it.

## Per-network settings

Each network has its own text limit and its own options. They go in `platform_content`, an object keyed by network (`facebook`, `instagram`, `linkedin`, `pinterest`, `threads`, `tiktok`, `youtube`, `bluesky`). Each post reads only the block for its own account's network, so one object can serve several accounts. X and Telegram take no options.

- **Different text per network:** `platform_content.<network>.content` replaces the caption on that network only. Use it when one caption can't fit everywhere. X allows 280 characters, Bluesky 300 and Threads 500.
- **Pinterest** needs `settings.boardId` on every post. `get_profile_options` with `option: "boards"` lists the boards. `settings.title` and `settings.targetUrl` are optional.
- **TikTok** needs `settings.privacyLevel` on every post. TikTok allows no default, so ask the user. `get_profile_options` with `option: "privacy_levels"` lists what that account may use.
- **Instagram and Facebook:** `contentType` is `post` (the default), `reels` or `story`. Instagram also takes `trial reel`. A reel or story needs media.
- **YouTube:** `settings.title` (up to 100 characters; without it the caption becomes the title), `settings.privacyStatus` and `settings.category`.
- **Threads:** `settings.pollOptions`, `settings.topicTag`, `settings.replyControl` and `settings.altTexts`.

The tool schema lists every field. For the limits of one account, call `get_profile_options` with `option: "limits"`. If your app can read MCP resources, `publishbuddy://networks` holds the rules for every network.

A required option that's missing comes back as an error with `candidates`: the values to offer the user. Don't guess a board or a privacy level.

Other fields worth knowing on `create_post`:

- `comments`: follow-up comments, such as a first comment that holds the hashtags.
- `labels`: tags for sorting posts in the app.
- `ai_generated`: set it to true when an attached file shows a realistic person or event made by AI. TikTok, YouTube and Instagram then show their AI label.
- `idempotency_key`: a unique string. If a call times out and you retry with the same key and arguments, you get the first result back and no second post.
- `allow_duplicate`: the same caption and media to the same account within 24 hours is refused as `DUPLICATE_POST`. Reuse the post in `candidates`, or pass this only if the user wants a second copy.

## Media

A post takes media by id, in `media_uuids`. There are three ways to get an id:

- **Already in the library:** `list_media` finds files by name or label.
- **A public link:** `upload_media` with `url` saves an image or video (JPEG, PNG, GIF, WebP, MP4 or QuickTime, up to 50 MB) and returns its id.
- **A file on the user's computer:** see below.

### Sending a local file

A tool call can't carry a file, so the server hands out a short-lived upload address and you send the file to it from a shell.

1. Call `request_media_upload`. Optional: `label`, and `folder_id` from `list_media`.
2. The answer holds `upload_url`, the limits (`max_files`, `max_file_bytes`), `expires_at` and a ready `curl` line.
3. Run it with the real path, one `-F` per file:

   ```bash
   curl -sS -F "files[]=@/path/to/photo.jpg" -F "files[]=@/path/to/clip.mp4" "<upload_url>"
   ```

   In Windows PowerShell, write `curl.exe`. To store a file under another name, add `;filename=<name>` after the path.
4. The response is JSON, `{ "data": [ ... ] }`. Each item's `id` goes in `media_uuids`.

The address is the credential. Use it straight away. Don't paste it into the chat or save it anywhere. It works for several uploads until it expires. After that, ask for a new one. Send only the files the user named.

If you have no shell, you can't send local files. Ask the user for a public link, or to add the file in the PublishBuddy app's Media Library.

## What costs AI credits

Only `generate_media` spends AI credits. Posts, scheduling, uploads and analytics are free.

- `generate_media` makes an image (`kind: "image"`), a short video (`video`), a video from an existing image (`animate`) or caption variations (`text`) with PublishBuddy's Studio models. The price depends on the model.
- It takes two calls. The first makes nothing and returns the quote: `credits_required`, `credits_remaining` and sometimes a `cheaper_option`, plus a `confirmation_token`. Tell the user the cost and wait for a yes. Then call again with the same arguments plus the token.
- Images and videos run in the background. You get a `generation_id`. Read the result with `get_media_generation` until `status` is `succeeded`. Its `media.id` goes in `media_uuids`. A failed generation costs nothing.
- Leave `model` out unless the user asks for one. Pass `profile_id` so the brand's colors and voice apply.
- You can write captions yourself for free. Use `kind: "text"` only when the user asks for it.
- Skip `kind: "carousel"` and the `finalize_carousel` tool for now. Carousels made in Studio aren't switched on yet. For a multi-image post, attach several images to one post.
- `get_workspace_overview` shows the credit balance. `publishbuddy://media-models` lists the models and prices.

If `generate_media` isn't in your tool list, this connection doesn't offer it. Say so, and work with the user's own media.

## Looking things up

- What's scheduled, what failed, what waits for approval: `list_posts` with `statuses` (`draft`, `needs_approval`, `waiting`, `publishing`, `published`, `failed`) and a date range.
- One post in detail, including why it failed: `get_post`.
- Analytics: `get_analytics` with `report` set to `engagement`, `profile_metrics`, `top_posts`, `best_times`, `hashtags`, `post_drivers` or `creative_brief`.

Report only the numbers the tools return. An empty breakdown means the plan or the network doesn't include it. It doesn't mean zero.

**A failed post:** read the error with `get_post` and tell the user why it failed. Fix what the error names: shorter text, say, or a reconnected account. `publish_now` re-runs a failed post, with the same two calls and the same yes from the user.

**Deleting:** `delete_post` works on drafts and posts waiting to publish, one at a time, in two calls. To pull a scheduled post back and keep it, use `schedule_post` with `mode: "unschedule"`.

## Permissions

A new connection can read, draft, schedule, upload media and read analytics. Publishing right away and approving (`posts:publish`), deleting (`posts:delete`), spending AI credits (`ai:spend`) and saving preferences (`settings:write`) are extra permissions the user grants.

If a call fails with `API_TOKEN_MISSING_SCOPE`, the message names the missing permission. Don't look for a way around it. Tell the user what it's for. They can grant it when your app asks them to sign in to PublishBuddy again, or the workspace owner can add it to the connected app.

## When a tool returns an error

Every error carries `error_code`, `category`, `message`, `fix`, `retryable` and sometimes `retry_after_seconds` and `candidates`. Read `fix` first. It says what to do.

| `category` | What it means | What to do |
| --- | --- | --- |
| `fix_content` | The post or the request is wrong | Change what `errors` or `message` names, then try again |
| `retryable_later` | A rate limit or a temporary fault | Wait `retry_after_seconds` if it's given, then try again once |
| `reconnect_account` | The social account's connection has expired | Ask the user to reconnect the account in the PublishBuddy app |
| `permission` | You or the user may not do this | Tell the user. Don't retry |
| `plan` | A plan, credit or daily limit | Tell the user. Retry only when `retryable` is true, after `retry_after_seconds` |
| `not_found` | The id doesn't exist in this workspace | Look it up again with a list tool |
| `conflict` | The post or the request isn't in a state that allows this | Read the post again with `get_post` and act on its current status |

The codes you'll meet most:

- `VALIDATION_FAILED`: a field is wrong. `errors` lists each one.
- `PROVIDER_VALIDATION_FAILED`: the network refused the content. The message says why.
- `DUPLICATE_POST`: the same post went to this account in the last 24 hours. See `allow_duplicate` above.
- `CONFIRMATION_INVALID`: the token expired, was used, or the arguments changed. Call again without a token for a new preview, and show it to the user again.
- `CONFLICT`: for example, scheduling a post that's already published, or queueing one that's already waiting.
- `DAILY_POST_LIMIT_REACHED` and `AGENT_CAP_REACHED`: a daily limit has been reached, for the workspace or for this agent. Wait `retry_after_seconds` or tell the user. Don't split the work to get around a cap.
- `AGENTS_PAUSED`: the workspace owner paused agents. Only they can resume them.
- `INSUFFICIENT_AI_CREDITS` and `AI_SPEND_CEILING_REACHED`: not enough credits, or the workspace's spending limit is reached. Tell the user what the action costs and what's left.
- `PROVIDER_AUTHENTICATION_FAILED`, `PROVIDER_AUTHORIZATION_FAILED` and `PROVIDER_REAUTHENTICATION_REQUIRED`: the user reconnects that account in the app.
- `PROVIDER_RATE_LIMITED` and `PROVIDER_UNAVAILABLE`: the network is busy or down. Try again in a few minutes.
- `RATE_LIMITED`: too many requests. Wait `retry_after_seconds`.
- `ACCOUNT_PAUSED` and `PLAN_UPGRADE_REQUIRED`: a plan limit that only the workspace owner can change. Pass on the `fix` from the error as it is.
- `FORBIDDEN`: the user's role doesn't allow this on that account. Some roles can only save drafts and send them for approval.

If your app can read MCP resources, `publishbuddy://errors` lists every code. When an error keeps coming back, give the user the `message` and the `request_id`. Support is at hello@publishbuddy.com.
