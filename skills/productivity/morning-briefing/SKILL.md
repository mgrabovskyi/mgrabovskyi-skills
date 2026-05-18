---
name: morning-briefing
description: "Set up a recurring morning briefing that scans the user's Slack and email over the last 24 hours and delivers a personalized summary of asks, mentions, updates, and blockers. Use when the user says things like 'daily briefing', 'morning summary', 'morning digest', 'daily standup summary', 'what did I miss', 'inbox + Slack rollup', or wants a scheduled rollup of work communications. Also triggers when someone wants to know who needs them, where they were mentioned, what shipped, and what's blocked — delivered on a schedule."
---

# Morning Briefing

Set up a recurring scheduled task that gives the user a daily summary of what happened in their work communications over the last 24 hours. The briefing always answers four questions: who needs me, where was I mentioned, what's worth knowing, what's blocked.

This skill builds and installs the scheduled task. It does not produce the briefing itself — the task you create does, every morning.

## Step 0 — Track progress

Create a TODO list with these five items so the user can see the flow:

1. Confirm scope
2. Collect preferences
3. Resolve user identity
4. Build the scheduled prompt
5. Create the task and pre-approve connectors

Mark each one complete as you finish it.

## Step 1 — Confirm scope

Open with one or two sentences explaining what you're about to do: a recurring task that, each morning, pulls the last 24h from their chat tool and email and posts a four-section summary (needs from me, mentions, updates, blockers) to a destination they pick. Ask them to confirm or adjust before you start collecting preferences. Keep it short.

If they've already specified the schedule, sources, or delivery target in the current message, skip ahead to Step 2 and only ask about the gaps.

## Step 2 — Collect preferences

Use `AskUserQuestion` to gather what you need. Group related choices in a single tool call. The four decisions are:

- **Schedule.** Default proposal: 9:00 local time, every weekday. Offer "Every day", "Weekdays only", "Custom time/cron". Capture the user's timezone — if memory or earlier context has it, state it back; otherwise ask.
- **Sources.** What communication tools should the task search? Common pairs: Slack + Outlook email, Slack + Gmail, Microsoft Teams + Outlook, Slack only, email only. Detect available connectors from the session if you can; otherwise ask. Record the connector tool names you'll actually call (e.g. `slack_search_public_and_private`, `outlook_email_search`, Teams equivalent, Gmail equivalent).
- **Slack/Teams scope.** Offer: "Only mentions and DMs" (highest signal), "Mentions + DMs + active channels" (broader), or "Specific channels the user names" (focused).
- **Delivery target.** Offer: "Slack self-DM (or Teams self-chat)" — recommended because it lands on mobile via push, "Private channel they name", "Email to themselves", "In-app chat only". State the trade-offs in one line each: self-DM is mobile-native but mixes with other DMs; private channel is cleaner but requires they create it; email survives any messaging outage; in-app chat requires their desktop app to be running at the scheduled time.

Do not ask about tone or audience — capture the user's role from memory or the conversation and bake it into the prompt. Default tone: professional, executive-grade, no fluff. Strip emojis unless the user requests them.

## Step 3 — Resolve user identity

You need three things to write a self-contained scheduled prompt:

1. **Name and email.** Pull from the session (user profile, prior messages, memory). Don't ask if you already have it.
2. **Slack/Teams user ID.** If the user picked self-DM as the delivery target, you must capture the actual user ID at task-creation time *or* instruct the scheduled prompt to resolve it at runtime via `slack_search_users` (or Teams equivalent). Runtime resolution is more robust because IDs occasionally change. Bake the lookup into the prompt as Step 1 of the runtime flow.
3. **Channel ID** if the user named a specific delivery channel.

If you can't resolve any of these, surface what's missing before creating the task.

## Step 4 — Build the scheduled prompt

The prompt you pass to `create_scheduled_task` will run with no memory of this session. Make it fully self-contained. Use the template below, filling in the placeholders with the user's actual preferences. Replace bracketed values; keep the rest verbatim.

````
You are producing the morning briefing for [USER FULL NAME] ([USER EMAIL]), [USER ROLE] at [USER ORG, if known]. Audience: [USER ROLE] — [TONE: e.g. strategic, concise, executive-grade]. No fluff, no emojis.

# Scope
- Time window: the last 24 hours from the moment this task runs.
- Sources: [CHAT TOOL — e.g. Slack workspace] + [EMAIL TOOL — e.g. Outlook inbox for USER EMAIL].
- Delivery: [DELIVERY DESCRIPTION — e.g. post as a message to the user's self-DM in Slack; also surface in this chat session].

# Step 1 — Resolve user identity in [CHAT TOOL]
Use `[USER LOOKUP TOOL — e.g. slack_search_users]` with query "[USER FULL NAME]" to find the user. Capture their user ID — this is the channel ID for their self-DM. If lookup fails, stop and surface the error in chat; do not guess.

# Step 2 — Pull [CHAT TOOL] signal (last 24h)
Run these searches in parallel where possible:

1. Direct asks and mentions: search for @[USER HANDLE OR NAME] across channels and DMs in the last 24 hours.
2. DMs: messages where the user is sender or recipient in the last 24 hours.
3. Active channels: identify the 5–10 highest-volume channels the user belongs to, prioritizing channels matching the user's domain (e.g. for engineering leadership: engineering, eng-leads, leadership, incidents, on-call, platform, security, product, roadmap, hiring). Sample them.
4. Threads: for any inbound mention or message addressed at the user's role, read the full thread for context.

Capture for each item: channel, author, timestamp, content, permalink.

# Step 3 — Pull email signal (last 24h)
Use `[EMAIL SEARCH TOOL]` to retrieve inbox messages received in the last 24 h. For each, capture: sender, subject, one-line summary, whether a reply is expected, deadline if any. Skip newsletters, marketing, and automated CI/monitoring noise unless an alert is unresolved.

# Step 4 — Synthesize the briefing
Produce a single message in [FORMAT — e.g. Slack mrkdwn] in this exact order:

1. Needs from me (action required) — direct asks, decisions awaited, approvals pending. Order by urgency. Each item: who, what, deadline, link.
2. Mentions of me — places I was named without a clear ask. One line each + link.
3. Team updates worth knowing — significant progress, launches, decisions, hires, customer signal, postmortems. Order by impact. One line + link.
4. Blockers and challenges — stuck work, escalations, incidents, cross-team dependencies, risks. Flag where [USER ROLE] intervention would unblock. Link + who's affected.
5. Top 3 priorities for today — opinionated take based on the above.

Section rules:
- Under 600 words total.
- Every claim links to a permalink or names an email subject. If a section is empty, say so explicitly. Never invent items.
- If a connector failed, prepend a "Coverage gaps" note at the top naming which source failed.

# Step 5 — Deliver
1. Send the briefing via `[DELIVERY TOOL — e.g. slack_send_message]` to `[DELIVERY DESTINATION — e.g. the resolved user ID for self-DM]`.
2. Confirm delivery from the response.
3. Surface the briefing in this chat session as the final response, prepended with: "Posted to [DELIVERY DESTINATION DESCRIPTION] at <time>."
4. If delivery fails, post the briefing in chat anyway and clearly state which delivery target failed and why.
````

Note on formatting: if delivery is Slack, use Slack mrkdwn (`*bold*`, `_italic_`, `> quote`, `•` bullets, `<URL|label>` links, no `#` headers). If delivery is Teams, use the Teams markdown subset. If delivery is email, use standard markdown — it renders well in Outlook/Gmail. If delivery is in-app chat only, use full markdown.

## Step 5 — Create the task and pre-approve connectors

Pick a task ID — default `morning-briefing` unless the user wants something else. Call `create_scheduled_task` with the cronExpression, the filled-in prompt, and a one-line description.

After creation, tell the user two things:

1. **Hit "Run now"** from the Scheduled sidebar today. The first run will prompt for connector permissions; approving them once stores the approval on the task so future scheduled runs don't pause waiting on auth.
2. **Where it will land.** Restate the destination ("Each morning at [TIME] [TZ], the briefing posts to [DESTINATION]."). If delivery depends on the desktop app being open, say so explicitly.

## Ground rules

- One step at a time. Don't dump all questions in the first message.
- Default to the lowest-friction delivery option (Slack/Teams self-DM) unless the user asks otherwise — it's mobile-native and requires no channel setup.
- Never hardcode the user's name, email, role, or organization into this skill itself. Those go into the *task prompt* you generate, not into the skill body.
- If a required connector isn't connected, stop and tell the user which one to connect before continuing. Do not generate a task that calls tools the user doesn't have.
- If the user already has a task with this name, ask whether to replace, update, or pick a new ID. Use `update_scheduled_task` for edits, not a new create.
