# Crash of the Amelu API through mailbox creation errors

| | |
|---|---|
| Status | Fixed |
| Reported | 2026-11-02 |
| Reporter | Radhouen Boufateh ([LinkedIn](https://www.linkedin.com/in/radhouen-boufateh-57026129a), [GitHub](https://github.com/radhouen911)) |
| Credit | Listed in the Hall of Fame as Radhouen Boufateh, with his permission |
| Severity | Critical |
| CWE | CWE-674 (Uncontrolled Recursion) |
| Affected | amelu, backend API (mailbox creation) |
| Fixed | commit `e1fcc84` in ordnary-com/amelu, 2026-11-03 |
| Reward | free one-year Amelu Pro plan, accepted and set up 2026-11-03 |

## Report

Found through a source-code review. `writeHTTPError` in
`backend/internal/handlers/mailboxes.go` called itself whenever an error
had no upstream cause. The reporter rated it High and showed it with
`POST /api/domains/{domainId}/mailboxes` and an empty `localPart`.

## Assessment

Confirmed, and worse than reported. The recursion runs until Go's
goroutine stack limit, which is a fatal runtime error, not a panic that
`net/http` recovers from. A single request took down the whole API
process. It needed a signed-in user with helpdesk role or higher, and it
also fired on ordinary errors such as "mailbox limit reached for your
plan" and "already exists". Introduced on 2026-10-13 (`b8016109`).

## Fix

- `e1fcc84` writeHTTPError now writes the status and message the error
  carries (400, 403, 409, ...), keeps 502 for upstream errors and answers
  500 for anything else. Regression test in `write_http_error_test.go`.

## Timeline

- 2026-11-02: report received
- 2026-11-03: confirmed and fixed in code
- 2026-11-03: researcher accepted the Amelu Pro plan and the Hall of Fame listing
- 2026-11-03: Amelu Pro set up, listed in the Hall of Fame
