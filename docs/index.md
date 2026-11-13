---
layout: home
---

# studio-kit

Stop accidentally emailing your whole list. Install the plugin and the guard activates immediately — no configuration required.

[Install the plugin](#installation){: .btn} &nbsp; [View on GitHub](https://github.com/husnain/studio-kit){: .btn}

---

## Send guard

The plugin's core feature. A `PreToolUse` hook that blocks dangerous Resend calls the moment it's installed.

**Always blocked** — bulk tools that reach whole contact lists: `send-broadcast`, `create-automation`, `send-inbox-draft`, `send-new-inbox-email`.

**Restricted to safe addresses** — `send-email` and `send-batch-emails` only go to `@resend.dev` addresses and one address you configure (`MY_EMAIL`). Everything else is blocked before Claude can send it.

You cannot accidentally email a real recipient. No code to write. No prompt to remember.

---

## Also included

**Audience personalizer** — a skill that reads your lead groups from `leads/leads.csv` and writes one tailored copy of each email per group. Run it with `/personalize campaigns/<folder>` once your emails are final.

**Brand voice reviewer** — a subagent that reviews every email in a campaign folder against your `brand/voice.md` and structural rules. Invoke it with `/email-reviewer campaigns/<folder>`.

**Headless review script** — `scripts/review.mjs` spawns a fresh Claude subprocess, runs the reviewer, writes `review/findings.json`, and exits with code `1` on any high-severity finding. Drop it into CI to block merges with brand violations.

---

## Installation

```bash
claude plugin install studio-kit
```

The send guard is active immediately. To allow test sends to yourself, add `MY_EMAIL` to your project's `.claude/settings.local.json`:

```json
{
  "env": {
    "MY_EMAIL": "you@yourdomain.com"
  }
}
```

---

## Setup for campaign tools

The plugin contains machinery. Your project contains content. Three files wire them together:

**`brand/voice.md`** — your tone, banned words, forbidden claims, and sign-off style.

**`brand/groups.md`** — one row per audience group describing who they are and what angle to lead with.

**`.claude/rules/emails.md`** — structural rules: subject and preview length, required merge tags, CTA format, footer copy.

Copy the templates from `examples/` in the repo as a starting point.

---

## CI integration

```yaml
- name: Run brand review
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: node /path/to/studio-kit/scripts/review.mjs campaigns/<folder>
```

Exit code `1` on high-severity findings fails the check automatically.

---

## License

MIT — [github.com/husnain/studio-kit](https://github.com/husnain/studio-kit)
