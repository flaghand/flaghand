# Flaghand

Your agents draft and schedule future posts. You approve immediate posting.

Flaghand is a publishing desk for connected social accounts, built for people and their AI agents. With this plugin, Claude drafts posts and threads, uploads media with alt text, checks each draft against the platform's rules and schedules ordinary future posts without a per-post passkey. Claude reports each scheduled time and any owner confirmation link. Only you confirm posting now, by-hand posts, unpicked options, TikTok or YouTube choices, each Pinterest Pin and a new workspace's first plan, on app.flaghand.com with your passkey.

## What's in it

- **Flaghand MCP server** at `https://app.flaghand.com/mcp`, the same remote server as the Flaghand connector. Sign in with your passkey, pick a workspace and click Allow. Claude's access is limited to reading, drafting, uploading media, sharing private preview links, managing future schedules and requesting approval. Revoke it any time in Settings › Agents.
- **The `flaghand` skill**: how to draft, check and schedule in your own voice, and how an agent with a shell finishes setup over Flaghand's HTTP API.
- **The `/flaghand:shipped [YYYY-MM-DD]` command**: in a git checkout, it reads what the repository merged since that date (default: the last 7 days) with `git log` and, if the GitHub CLI is signed in, `gh pr list`. It then drafts and checks one X post or thread and one LinkedIn post, and reports their scheduled times or any owner confirmation link. It never checks out, commits, pushes or comments.

## Install

Add it from Anthropic's directory, or in Claude Code run `/plugin marketplace add https://app.flaghand.com/agent-plugin/marketplace.json`, then `/plugin install flaghand@flaghand`.

## Before you connect

You need a Flaghand account with an active workspace plan where the tools require one, and your own X, LinkedIn or TikTok connection for those platforms. Without an account, start at https://flaghand.com/llms.txt. `GET https://app.flaghand.com/api/v1/plans` says whether sign-up is open and what the plans cost.

## What it sends and runs

- Your content goes only to Flaghand's hosted service at app.flaghand.com: the drafts, images, video, alt text and notes you ask Claude to create, including the summaries `/flaghand:shipped` writes from your repository. Flaghand publishes checked future schedules and owner-approved posts to the accounts you connected. Agents never approve, post immediately or send DMs.
- `/flaghand:shipped` also asks GitHub, through your own signed-in `gh`, which pull requests merged in that repository. It sends nothing else to GitHub.
- The plugin has no hooks and no local server, and it downloads and runs no software of its own. Besides your `git` and `gh` for `/flaghand:shipped`, Claude reaches Flaghand only through the MCP server above or, during setup from a shell, Flaghand's HTTP API at app.flaghand.com.
- The plugin keeps no data of its own. During setup from a shell, the skill has Claude save Flaghand's sign-up and claim replies in private files outside your repository and the Flaghand token in your own secret storage, never in a draft, note or log. What Flaghand keeps, and for how long: [Privacy](https://flaghand.com/privacy).

Guide: https://app.flaghand.com/docs/agents · Support: support@flaghand.com

## Licence

The files in this plugin are MIT-0 (see LICENSE). Flaghand itself (the hosted service, its server and app code, and the CLI) is not open source and is not licensed here; using it is governed by the [Terms](https://flaghand.com/terms). The Flaghand name and logo are trademarks of Trellisbyte LLC.
