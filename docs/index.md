---
layout: home
---

# studio-kit

A Claude Code plugin with the reusable infrastructure for email campaign projects.

[Install the plugin](#installation){: .btn} &nbsp; [View on GitHub](https://github.com/husnain/studio-kit){: .btn}

---

## What's included

**Send guard** — a `PreToolUse` hook that blocks bulk Resend tools and restricts `send-email` and `send-batch-emails` to `@resend.dev` addresses and one configured address. Activates automatically on install. You cannot accidentally email your whole list.

**Audience personalizer** — a skill that reads your lead groups from `leads/leads.csv` and writes one tailored copy of each email per group. Run it with `/personalize campaigns/<folder>` once your emails are final.

**Brand voice reviewer** — a subagent that reviews every email in a campaign folder against your `brand/voice.md` and structural rules. Invoke it directly with `/email-reviewer campaigns/<folder>`.

**Headless review script** — `scripts/review.mjs` spawns a fresh Claude subprocess, runs the reviewer, writes `review/findings.json`, and exits with code `1` on any high-severity finding. Drop it into CI to block merges with brand violations.

---

## Installation

```bash
claude plugin install studio-kit
```

Then set your personal email in your project's `.claude/settings.local.json`:

```json
{
  "env": {
    "MY_EMAIL": "you@yourdomain.com"
  }
}
```

---

## Setup

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
