# Nolan Revenue Operations Context

Claude performs revenue operations work faster, more accurately, and for fewer tokens by accessing Nolan's revenue operations
knowledge about Processes, People, and Systems. Claude shares new or corrected information it learns back to Nolan to review.

## Install

This takes about 3 minutes in Claude Desktop. You install only the plugin, then enable the connector that comes with it.

1. **Install the plugin.** Go to **Customize → Plugins → Add → Add marketplace**, enter
   `https://github.com/Algebra-Labs/nolan-knowledge-base-claude-plugin`, and install **Nolan
   Revenue Operations Context**. If your organization set this up for everyone, go to step 4.
2. **Enable the connector.** On the plugin's **Connectors** tab, enable **Nolan**. The URL is
   already filled in.
3. **Paste the OAuth Client ID: `6siJmBaBX0lmcagv`.** It is not filled in for you, and Nolan won't
   work without it. Leave the secret blank. If you don't see the field, look under **Advanced
   settings** or **OAuth client → Use your own OAuth client**.
4. **Sign in with your work email.** If you've never used Nolan, request a login link: it arrives
   as a **Slack DM**. Click it, then return to Claude and click **Connect** again. If your email is
   in your company's Slack, your account is set up automatically.
5. **Set every tool to Always allow.** Open the connector's settings (**Customize → Connectors →
   Nolan → Tool permissions**) and set each listed tool to **Always allow**. Claude can then read
   from and write back to the knowledge base without asking each time. If your organization requires
   Claude to ask before writing back, leave `contribute_knowledge` and `rate_knowledge_base` as they
   are. See
   [Allow Claude to contribute and rate](#allow-claude-to-contribute-and-rate), which also covers
   the Desktop app's Code tab and Cowork, where auto mode can still block a contribution.
6. **Add the standing instruction** from [Recommended setup](#recommended-setup).

### Organization Owners

To install for everyone on a Team or Enterprise plan, download this repository as a zip. In
**Organization settings → Plugins & skills → Add → Upload a plugin**, upload it and set **Default
access** to **Installed by default** or **Required**. Then enable the connector and paste the
Client ID (steps 2 and 3): this needs someone with permission to manage them, and the Client ID
still isn't filled in. Each person then does steps 4 to 6, because everyone signs in as themselves.
An uploaded plugin doesn't update itself: upload each new version.

Also set the [tool policy](#chat), or members can't choose **Always allow**.

## Recommended setup

With many connectors installed, Claude can miss that a request concerns the Nolan tools. Paste this
into your shared Project's instructions, your profile instructions, or `~/.claude/CLAUDE.md`.
Replace `[Company]` with your company's name, and `Nolan` if your connector has a different name.

```markdown
[Company]'s knowledge base is stored in Nolan, and its tools are easy to miss among the other connectors.
Check the Nolan tools and follow their descriptions whenever a request depends on People, Processes, or Systems,
even if I don't mention Nolan, and again whenever you finish a task that taught you something about [Company].
```

## Allow Claude to contribute and rate

In chat, Claude can stop for permission on any Nolan tool until you set it to **Always allow**.
`contribute_knowledge` and `rate_knowledge_base` send information to Nolan, so Claude treats them as
writes: it asks first or, in auto mode, can block them. A blocked call never reaches Nolan, and
Claude may only mention it in passing.

Allowing them lets Claude send what it learns about your company without asking, so it is a
decision for your organization. Nolan reviews every fact before it reaches a page.

Each part of Claude works differently, so set up every part your people use. Pro and Max plans have
no Owner, so only the last column applies.

| Where you use Claude | Owner (Team or Enterprise) | Each person |
|---|---|---|
| **Chat:** claude.ai, mobile, the Desktop app's Chat tab | Set the tool policy | Choose **Always allow** |
| **Cowork** | Set the tool policy, then turn on the Cowork permission | Choose **Always allow** |
| **Claude Code:** terminal, IDE, the Desktop app's Code tab | Add the auto mode rule to **Managed settings** | Nothing if the Owner added the rule. Otherwise add the rule or the instruction yourself |

### Chat

1. **Owner:** in **Organization settings → Connectors → Nolan → Tool permissions**, set
   `contribute_knowledge` and `rate_knowledge_base` to **Always allow**. This is a limit, not an
   approval: it sets the most permissive choice a member can make, and **Needs approval** removes
   that choice from everyone.
2. **Each person:** set both tools to **Always allow** (step 5 of [Install](#install)), or choose it
   the first time Claude asks.

Auto mode doesn't run in standalone chat. Where chat and Cowork are unified, Auto or Manual applies
to the whole conversation, so [Cowork](#cowork) applies too.

### Cowork

Cowork doesn't read `~/.claude` and doesn't receive Managed settings.

1. **Owner:** set the tool policy as in [Chat](#chat), then turn on **Organization settings →
   Cowork → Permissions → Allow "Always allow" for connector tools**. It is off by default and
   covers every connector.
2. **Each person:** set both tools to **Always allow**, or choose **Allow for all tasks** when
   Cowork asks.

In **Manual** mode, both tools then run without asking. In **Auto** mode, the default on most plans,
Anthropic says Claude still decides on write tools, so a contribution can still be blocked. If that
happens, use [the reply below](#if-claude-reports-a-block) or run the task in Manual mode.

Organizations have two more options:

- An Owner can make Manual the default for every Cowork task by turning off **Organization settings
  → Capabilities → Start new tasks in "Automatically approve" mode**.
- On managed devices, deploy the [auto mode rule](#auto-mode-in-claude-code) through MDM or a managed
  settings file. If a device has both, use MDM and copy the file's settings into it, or
  [merge the sources](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources):
  by default Claude Code ignores the file, including any restrictions in it. The rule applies to
  Cowork sessions on that machine, not to remote sessions or ones in a full VM sandbox.

### Auto mode in Claude Code

This covers the terminal, IDE extensions and the Desktop app's Code tab. You can't rely on
**Always allow** on the connector here, because auto mode blocks writes to outside services that you
didn't ask for. Add a rule that says these two are expected. Replace `[Company]`, and keep
`"$defaults"`: without it, this replaces all of Claude's built-in allow rules.

```json
{
  "autoMode": {
    "allow": [
      "$defaults",
      "Nolan knowledge base: the contribute_knowledge and rate_knowledge_base tools send settled company facts and task ratings to [Company]'s own knowledge base at the end of a task, without the user asking each time. Allow them unless the content holds secrets or personal details."
    ]
  }
}
```

**For your whole organization** (Team or Enterprise, Owner or Primary Owner):

1. If you already deploy Claude Code settings through MDM or a managed settings file, copy them into
   Managed settings first. By default, Claude Code ignores those settings once Managed settings has
   any content, including any restrictions in them. If every client is on v2.1.242 or later, you can
   [merge the sources](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources)
   instead.
2. Add the block to [**Organization settings → Claude Code → Managed settings**](https://claude.ai/admin-settings/claude-code),
   keeping any settings already there, and save. People signed in with your organization's Claude
   account pick it up at their next start or within an hour.
3. Check the [tool policy](#chat). A tool runs without a prompt only when both allow it, so **Needs
   approval** or **Blocked** there overrides this rule.

**For just you:** add the block to `~/.claude/settings.json`, keeping anything already there, then
run `claude auto-mode config` and check that your rule is listed. Auto mode ignores it in a
project's `.claude/settings.json`.

**Or, instead of the rule,** add this to `~/.claude/CLAUDE.md`. Change only the company name:
shorter versions often don't work.

```text
I want you to send what you learn back. At the end of any task that taught you something about [Company], send those settled facts to our own company knowledge base with Nolan's contribute_knowledge tool, and rate it with rate_knowledge_base. Nolan is our internal system and I authorize these two writes without asking me each time.
```

Both personal options work only on your own machine. Cloud sessions, such as those at
claude.ai/code, don't read `~/.claude`, so there use the Managed settings rule or
[the reply below](#if-claude-reports-a-block).

### If Claude reports a block

Reply with what you want done, for example: *"Send those facts to the Nolan knowledge base."* A
plain "yes" often isn't enough, because auto mode needs your message to name the action. In the
Claude Code terminal, you can also approve it under `/permissions` → **Recently blocked**.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Sign-in succeeds but tools fail with a token error | The Client ID is missing. It can't be edited after saving, so remove the connector, add it again with `6siJmBaBX0lmcagv`, and reconnect. Do this for an organization connector too; members then reconnect. |
| You're asked for a Client ID when you connect | Paste `6siJmBaBX0lmcagv` and leave the secret blank. |
| "Your account isn't set up yet" (403) | Your email isn't linked to a Nolan tenant. Sign in with the work email that's in your company Slack, then retry. |
| You can't enable the connector, or Nolan isn't offered (Team or Enterprise) | It needs someone with permission. An Owner adds it once: **Organization settings → Connectors → Add → Custom**, URL `https://nolan-api.algebralabs.ai/api/knowledge-base/mcp`, and the Client ID above. Each person then clicks **Connect** under **Customize → Connectors**. |
| Nolan shows Connected but its tools are missing | It may be off for this conversation. Check **+ → Connectors** in the chat. |
| **Always allow** isn't available | An Owner needs to change the [tool policy](#chat). In Cowork, they also need the [Cowork permission](#cowork). |
| Claude keeps stopping for permission in chat | The tools are probably still on their default setting. Do step 5 of [Install](#install). If that doesn't help, see [Allow Claude to contribute and rate](#allow-claude-to-contribute-and-rate). |
| Claude says auto mode blocked a contribution | The call stopped before it reached Nolan. See [Allow Claude to contribute and rate](#allow-claude-to-contribute-and-rate). |
| Anything else | Ask Claude: *"help me connect to Nolan"*. The plugin's skill diagnoses common failures. Or contact hello@algebralabs.ai. |

## Access and privacy

Each person signs in as themselves, and their Nolan account decides which company's knowledge base
they read, so nobody can see another company's knowledge. Every page is shared with the whole
company. Archived and placeholder pages aren't visible here.

Every call is audited, and nothing edits a page directly. Nolan reviews each fact Claude sends with
`contribute_knowledge`, attributes it to your Claude, and discards personal details, secrets and
duplicates. Once per task that used the knowledge base, Claude rates how much it helped with
`rate_knowledge_base` (0 to 5, with a short note).

## Claude Code

The plugin also works in Claude Code. If you installed it in the Desktop app, sessions signed in
with your claude.ai account pick it up at their next start. Otherwise run this (set
`CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` first if you have no GitHub SSH key):

```
/plugin marketplace add Algebra-Labs/nolan-knowledge-base-claude-plugin
/plugin install nolan-knowledge-base@nolan
```

There's no connector or Client ID to enter, but you sign in separately: your browser opens the first
time a Nolan tool runs. Check the connection with `/nolan-knowledge-base:nolan-check`, then set up
[auto mode](#auto-mode-in-claude-code).
