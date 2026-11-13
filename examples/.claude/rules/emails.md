# Email Structural Rules

## Subject line

- Maximum 50 characters (including spaces)
- Sentence case — no title case, no all-caps
- No emoji

## Preview line

- Maximum 90 characters
- Must not duplicate the subject line word-for-word

## Required merge tags

Every email must contain:

- `{{first_name}}` — in the opening greeting (`Hi {{first_name}},`)
- `{{unsubscribe_url}}` — in the footer, as the href of the unsubscribe link

## CTA link

- Must use `https://`
- Link text must be a verb phrase, not a bare URL (`Get started` not `https://…`)
- One primary CTA per email — do not add secondary links in the body

## Footer

Every email must end with exactly this block (merge tags filled at send time):

```
---
You're receiving this because you signed up at [yourdomain.com].
[Unsubscribe]({{unsubscribe_url}}) · [Company Name] · [Street Address]
```
