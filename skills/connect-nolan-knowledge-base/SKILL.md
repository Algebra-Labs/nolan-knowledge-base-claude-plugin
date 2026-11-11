---
name: connect-nolan-knowledge-base
description: Connect, authorize, or troubleshoot the Nolan knowledge base connection. Use when the nolan-knowledge-base tools are missing, when a Nolan tool call fails with an authorization or rate-limit error, or when the user asks how to set up, connect, sign in to, or fix Nolan.
---

# Connect Nolan Knowledge Base

Installing the plugin registers the MCP server. It does **not** authorize the user — that is a
separate per-user sign-in against the company's Nolan account. This skill covers getting from
"installed" to "working", and decoding the failures that look alike but aren't.

Diagnose before instructing. Do not read out a whole setup guide when one specific thing is wrong.

## First: which situation is this?

1. **No `nolan-knowledge-base` tools in the session at all** → the plugin is installed but the
   MCP server isn't loaded, or the plugin isn't enabled. Go to *Installing*.
2. **Tools are present but a call failed** → go to *Decoding failures*. The error text
   distinguishes the cases; do not guess.
3. **Tools are present and calls succeed** → nothing to do. Say so and stop.

## Installing

Give instructions for the surface actually in use — not all three.

**Claude Code.** The plugin ships the server config, so no manual `claude mcp add` is needed.
Confirm the plugin is enabled (`/plugin`), then trigger any Nolan tool: Claude Code opens a
browser for sign-in on first use.

**Claude Desktop, claude.ai and Cowork.** These use the plugins and connectors on the user's
claude.ai account, not Claude Code's. Installing the plugin doesn't connect anything, so take the
user through the steps they haven't done:

1. Install the plugin from **Customize → Plugins** (the organization may have installed it
   already).
2. On the plugin's **Connectors** tab, enable **Nolan**. The URL is already filled in.
3. Paste the OAuth Client ID `6siJmBaBX0lmcagv`. It is never filled in for them, and a connector
   saved without it fails at sign-in. Leave the secret blank. The field is under **Advanced
   settings** or **OAuth client → Use your own OAuth client**. Nolan's authorization server
   supports neither automatic registration nor Claude's published identity. The Client ID can't be
   edited later: remove the connector, add it again, and reconnect.
4. Sign in with their work email.
5. Open **Customize → Connectors → Nolan → Tool permissions** and set every listed tool to
   **Always allow**. If the organization requires Claude to ask before writing back, leave
   `contribute_knowledge` and `rate_knowledge_base` as they are. The Code tab and Cowork have more
   to it: see *What this connection can and cannot do*.

An organization-wide install needs someone with permission to install the plugin, enable the
connector and paste the Client ID. Each person then does steps 4 and 5, signing in as themselves.
If a member is asked for the Client ID when connecting, they paste it as in step 3.

In chat, permission blocks on Nolan tools usually mean the tools are on their default setting, and
step 5 fixes them. It won't if the organization requires the prompts, or if an Owner's tool policy
or a role stops the user choosing **Always allow**. Where chat and Cowork are unified, Auto or Manual
applies to the whole conversation, so treat a block there as Cowork.

If a Team or Enterprise member can't add the connector, or Nolan isn't offered, an Owner adds it
once under **Organization settings → Connectors → Add → Custom**, with URL
`https://nolan-api.algebralabs.ai/api/knowledge-base/mcp` and the Client ID above. Each member then
clicks **Connect** under **Customize → Connectors**. Say that plainly rather than walking a member
through a settings page they can't complete.

If Nolan shows **Connected** but its tools are missing, it may be turned off for this conversation:
check **+ → Connectors** in the chat.

## Decoding failures

**`401`, immediately, before any browser window opens.**
The client's OAuth Client ID doesn't match what the server expects, or it's pointed at the wrong
environment (a local API URL against production credentials, or the reverse). Check the connector
URL and the Client ID together — they must be from the same environment.

**`403` — "your account isn't set up yet".**
The signed-in account isn't linked to a Nolan tenant yet. If the user's work email is in a Slack
workspace connected to Nolan, connecting again usually resolves it on its own. Otherwise: visit
the Nolan dashboard, sign in with the **work** email (this sends a Slack magic link), click the
link, then return here and **press Connect again**. The second Connect is required — the sign-in
is deliberately not resumed automatically, because carrying a caller-supplied authorization URL
through a magic link would be a login-CSRF vector. Don't describe the retry as a bug.

**`429`.**
Sixty calls per minute per user, shared across every client that user has connected. Wait a
minute; don't retry in a loop.

**Tool calls return content, but it's the wrong company's.**
Not possible by construction — the tenant comes from the signed-in account, never from a tool
argument. If the content looks wrong, the user is signed in as the wrong account.

## What this connection can and cannot do

Four tools: `search_knowledge_base` and `read_page` read the knowledge base;
`contribute_knowledge` sends facts back for Nolan to review, and `rate_knowledge_base` tells Nolan
how much the knowledge base helped a task. Nothing edits a page, and nothing tells Claude what
happened to a contribution, so never tell the user a fact is now in the knowledge base. If Claude
asks for permission before every contribution or rating, the user can set both tools to "Always
allow" in the connector's tool permissions, unless their organization requires the prompts. On a Team or Enterprise plan, an Owner's tool policy is
only a limit: it has to allow "Always allow", and each person still chooses it. In Cowork on those
plans, write tools ask in every task until the Owner turns on
**Organization settings → Cowork → Permissions → Allow "Always allow" for connector tools**.

Auto mode is separate. If it blocks a contribution or rating, don't look for another way to send
it: tell the user, and point them to **Allow Claude to contribute and rate** in the plugin README.
In Claude Code, including the Desktop app's Code tab, "Always allow" can't be relied on for auto
mode, so they need the rule or instruction given there. In Cowork, a write tool can still be blocked in
Auto mode after "Always allow". For a single block, the user can reply naming the action, such as
"Send those facts to the Nolan knowledge base."

Once the connection works, [use-company-knowledge-base](../use-company-knowledge-base/SKILL.md)
governs when and how to look things up.
