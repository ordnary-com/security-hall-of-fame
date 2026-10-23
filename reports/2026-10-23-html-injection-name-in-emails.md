# HTML injection through the name into account emails

| | |
|---|---|
| Status | Fixed |
| Reported | 2026-10-23 |
| Reporter | <name or handle> (contact: <email>) |
| Credit | pending, asked in our reply |
| Severity | Low |
| CWE | CWE-79 (Improper Neutralization of Input During Web Page Generation) |
| Affected | accounts (identity), transactional emails |
| Fixed | commits `c97e9fe`, `db42dbb` in ordnary-com/identity; live 2026-10-23 (accounts `7c4f384d`, containers `885bbb98`) |
| Reward | free one-year Amelu plan, offered in our reply |

## Report

The First Name field accepted HTML. A payload such as
`<img src="https://...">` was rendered when the name was shown. The
reporter rated it Medium (provisional) and noted that more serious impact,
such as XSS, would have to be shown separately.

## Assessment

Confirmed, in emails only. Web pages render the name as text (React
encodes it). The shared email template did no HTML encoding, and every
account email starts with `Hi <name>,`, so the payload rendered as HTML in
sign-in codes, security notices and the like.

Email clients do not run scripts, so the impact is content spoofing in
email, not XSS or account takeover. In most emails the recipient is the
person who chose the name. While fixing it we found two emails where the
name reaches someone else: the parental-consent confirmed and revoked
emails sent to a parent, which carried the child's display name raw.

## Fix

- `c97e9fe` fix(accounts): encode user input in emails. Shared
  `escapeHtml`; greetings, email addresses, IPs and other user-supplied
  values encoded; parental-consent emails fixed; the internal
  `/api/v1/email/send` route treats every field as plain text.
- `db42dbb` feat(accounts): refuse markup in names. Signup and the Helm
  admin name edit refuse `<`, `>` and control characters.

## Timeline

- 2026-10-23: report received, confirmed, fixed in code
- 2026-10-23: fix live in production
- YYYY-MM-DD: reply sent
- YYYY-MM-DD: researcher listed in the Hall of Fame (if they agree)
