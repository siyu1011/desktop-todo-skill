---
name: desktop-todo
description: >-
  When the user asks to remember, add, list, check off, reschedule, or delete a
  to-do (e.g. 记一下/加个待办/今天要做什么/做完了), manage it in their self-hosted todo service
  so it shows on their desktop widget.
---
# Desktop todo

The user's to-dos live in a self-hosted todo service and show up on a desktop widget that refreshes about every minute. Everything goes through the bundled `todo` CLI over HTTP.

## CLI

The script is next to this skill, in `scripts/todo`. Run it as:

```bash
python3 <skill-dir>/scripts/todo <command> [args]
```

`<skill-dir>` is the directory that contains this `SKILL.md` (usually named `desktop-todo`). On Windows, use `python` if `python3` is not available.

## Setup check

1. Config comes only from environment variables `TODO_API_URL` + `TODO_API_KEY`, or from the CLI config file (default `~/.config/todo/config.json`, or the path in `TODO_CLI_CONFIG`). Never hardcode a server address or a key, and never print, echo, or paste the key into chat.
2. `python3 <skill-dir>/scripts/todo whoami` confirms the account the key belongs to.

## The AI key

- Each account has one AI binding key, shown in the widget under 设置 → AI 绑定密钥 (show/hide, copy, regenerate). Regenerating there invalidates the old key at once.
- Exit code 3 almost always means the key is missing or the user regenerated it in the app. Tell them, and ask them to copy the new key from 设置 → AI 绑定密钥 and send it. Write it into the environment or the config file without echoing it, then retry the original command.
- Never ask for the account password in chat.

## Turning speech into a to-do

- Keep the user's own wording, trimmed to one clear line. Split several tasks into several `add` calls.
- Dates use the user's local timezone. 今天 -> `--due today`, 明天 -> `--due tomorrow`, a weekday or a calendar date -> compute `YYYY-MM-DD` in that local timezone; no date -> leave `--due` off (goes to 以后).
- The server resolves `today` / `tomorrow`. Trust the `today` field in `list` output as the server's current date; do not invent a timezone of your own.
- Time of day (下午三点) goes into the text, e.g. `15:00 交报告`, since due is date-only.
- Always `--source ai`.

## Commands (add `--json` when you need to read results)

- List: `todo list --json` returns `{ok, today, todos:[...]}`. Each todo has id, text, due (date or null), done, group (今天/明天/以后), overdue, source, created_at, and when done, completed_at.
- Add: `todo add "文字" [--due today|tomorrow|YYYY-MM-DD] --source ai --json`
- Done / undo: `todo done <id>` / `todo undone <id>` (sets or clears completed_at)
- Edit: `todo edit <id> [--text "..."] [--due today|tomorrow|YYYY-MM-DD|以后] --json`
- Delete: `todo delete <id>`
- ids accept a unique prefix.

For done/edit/delete, run `list --json` first and match the item by text; if more than one matches, ask the user which one. Confirm before deleting unless the user named the item clearly.

## How the widget shows things (match it when replying)

- Unfinished items are grouped 今天 / 明天 / 以后; overdue ones are flagged.
- Done items are folded into 「已完成 N 条」 at the bottom of each group, and anything completed more than a day ago is hidden from the widget (it is still returned by `list`).

## Exit codes

0 ok (parse stdout), 1 bad usage or ambiguous id prefix (also missing `TODO_API_URL` / config url), 2 server unreachable or error, 3 unauthorized (see The AI key), 4 not found. On non-zero, tell the user plainly what failed (from stderr) in their language.

## Reply

One short line in the user's language confirming what changed, e.g. 已加到今天：交报告. When listing, show only unfinished items grouped by 今天 / 明天 / 以后 with overdue marked, then one line like 今天已完成 2 条; list done items only if asked.
