---
name: personalize
description: Write a version of each campaign email for every lead group in leads/leads.csv. Use when a campaign's emails are final and you want one copy per group.
argument-hint: "[campaign folder]"
context: fork
allowed-tools: Read, Glob, Grep, Write(campaigns/**)
---

You are running the **personalize** skill. Work through the steps below, then stop with a short summary. Do not explain what you are doing as you go.

## Step 1 — Check for a campaign folder

The argument passed to this skill is: `$ARGUMENTS`

If `$ARGUMENTS` is empty or blank:
- Use Glob to list all folders matching `campaigns/*/`
- Print them as a plain list
- Stop. Do not proceed further.

## Step 2 — Read the groups

Read `leads/leads.csv`. The `group` column contains the group names.

If `brand/groups.md` exists, read it. It describes each group and what angle to lead with when writing for them. Use those descriptions to guide the opening line and example you write in Step 4.

If `brand/groups.md` does not exist, use only the group names from leads.csv with no persona direction. Note in your Step 5 summary that persona guidance was absent and suggest creating `brand/groups.md`.

## Step 3 — Find the source emails

Use Glob to find all files matching `$ARGUMENTS/emails/*.md`. Read every file you find.

## Step 4 — Write personalized versions

For each source email, write one version per group. Save each version to:

```
$ARGUMENTS/groups/<group>/emails/<original-filename>
```

The Write tool creates parent directories automatically.

**Keep these identical to the source:**
- `Subject:` line
- `Preview:` line
- `Hi {{first_name}},` greeting
- The main CTA link — both the text and the URL. Do not change either.
- The sign-off line and footer. Do not change any footer text.
- The `---` separator and everything below it (Production notes)

**Change only these two things:**
1. **Opening line** — the first sentence of the body after `Hi {{first_name}},`. Replace it with one sentence that speaks directly to why that group is on the list.
2. **One example** — if the body contains a specific example (a scenario, a workflow pain point, a feature use case), swap it for one that fits the group better. If there is no clear example to swap, leave the rest of the body unchanged.

Write in the brand voice: plain, direct, warm. No jargon. Short sentences.

## Step 5 — Reply with a short summary only

When every file is written, reply with exactly this:

1. Total files written, in the form: `N files written — X emails × Y groups`
2. One example opening line per group, labelled with the group name
3. If `brand/groups.md` was absent, one line noting this and suggesting the client create it

Nothing else. No file paths, no explanations, no next steps.
