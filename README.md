# studio-kit

A Claude Code plugin with the reusable infrastructure for email campaign projects: a safe send guard, an audience personalizer, a brand voice reviewer, and the scripts to run them.

---

## What is in this kit

- **`hooks/send-guard.mjs`** — a `PreToolUse` hook that blocks bulk Resend tools and restricts `send-email` / `send-batch-emails` to `@resend.dev` addresses and one configured address (`MY_EMAIL`). Activates automatically when the plugin is installed.

- **`skills/personalize`** — a Claude Code skill that reads your audience groups from `leads/leads.csv`, then writes one copy of each email per group into `campaigns/<name>/groups/<group>/emails/`. It reads persona angles from `brand/groups.md` if you create one.

- **`agents/email-reviewer.md`** — a subagent you can invoke directly in a session (`/email-reviewer campaigns/<folder>`) to review emails against your brand voice and structural rules.

- **`review/findings.schema.json`** — the JSON contract every reviewer output must match. Each finding has: file path, line number, severity (`high` / `medium` / `low`), problem, fix.

- **`scripts/review.mjs`** — a headless review runner. Spawns a fresh Claude subprocess with read-only tools, reviews a campaign folder, writes `review/findings.json` to your project, and exits with code `1` if any finding is `high`. Use it on the command line or in CI.

---

## Why brand, leads, and campaigns stay in the client repo

This plugin contains machinery. Your repo contains content.

The plugin has no brand awareness — it does not know who you are, what you sell, or what your voice sounds like. That knowledge lives in `brand/voice.md` and `brand/groups.md` in your project. The plugin reads those files when it runs; it does not ship with them.

Leads are audience data. They are specific to a campaign, subject to consent obligations, and change frequently. A plugin that shipped leads would share one list across every project that installs it.

Campaigns are creative output. They belong in version control alongside the brief and the notes that explain the decisions.

Keeping content in the client repo also means you can update the plugin — a fixed hook, a better reviewer prompt, a script improvement — without touching any campaign files.

---

## Setup

### 1. Register the plugin

In your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "studio-kit-local": {
      "source": {
        "source": "local",
        "path": "/absolute/path/to/studio-kit"
      }
    }
  },
  "enabledPlugins": {
    "studio-kit@studio-kit-local": true
  }
}
```

### 2. Create your brand files

Copy the templates from `examples/` into your project root as a starting point:

```
examples/brand/voice.md         → brand/voice.md
examples/brand/groups.md        → brand/groups.md
examples/leads/leads.csv        → leads/leads.csv
examples/.claude/rules/emails.md → .claude/rules/emails.md
```

**`brand/voice.md`** — at minimum:
- Three words that describe your tone
- Words and phrases you avoid
- Claims you must never make
- How you sign off

**`brand/groups.md`** — one row per audience group that appears in `leads/leads.csv`:

```markdown
| Group | Who they are |
|---|---|
| `freelancers` | Solo operators focused on time. Lead with how fast they can get started. |
| `agency-owners` | Manage multiple clients. Lead with control and visibility across accounts. |
| `in-house-teams` | Work inside larger orgs. Lead with how this fits existing approval workflows. |
```

Without this file the personalize skill will still run, but opening lines will be generic rewrites rather than targeted copy.

### 3. Create your structural rules

**`.claude/rules/emails.md`** — the reviewer and the review script both read this file. At minimum define:
- Subject line character limit
- Preview line character limit
- Required merge tags (greeting, footer)
- CTA link format and URL
- Footer copy

### 4. Set `MY_EMAIL`

In your project's `.claude/settings.local.json`:

```json
{
  "env": {
    "MY_EMAIL": "you@yourdomain.com"
  }
}
```

Without this, the send guard only permits `@resend.dev` addresses. You will not be able to send test emails to yourself.

---

## Running the reviewer

```bash
node /path/to/studio-kit/scripts/review.mjs campaigns/<folder>
```

The script:
1. Launches a fresh Claude subprocess with only `Read`, `Glob`, and `Grep` available
2. Asks it to review every email in the folder against `brand/voice.md` and `.claude/rules/emails.md`
3. Writes results to `review/findings.json` in your project
4. Exits with code `1` if any finding is `high`, `0` otherwise

**Prerequisites:** Claude Code must be installed (`npm install -g @anthropic-ai/claude-code`) and authenticated. In CI, set `ANTHROPIC_API_KEY` as an environment secret. Node.js 18 or later is required.

---

## GitHub Actions CI

Add a workflow that calls the script on every PR that touches email files:

```yaml
- name: Run brand review
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: node /path/to/studio-kit/scripts/review.mjs campaigns/<folder>
```

Exit code `1` on high-severity findings will fail the check automatically.
