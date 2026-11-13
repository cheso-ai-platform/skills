---
name: cheso
description: Create finished deliverables with Cheso (cheso.ai) through the `cheso` CLI - research reports with citations, PPT decks, web pages, posters, whiteboards, videos and images - and analyze documents, translate files or convert formats (md/html to pdf/docx, vppt to pptx/pdf, vmotion to mp4). Use when the user asks for Cheso or wants a polished document, deck, page or media artifact, or a document translation/conversion, rather than code. Tasks take minutes and spend the user's Cheso credits.
---

# Cheso

Cheso is an AI agent service that turns a prompt (plus optional files) into finished work: cited research
reports, PPT decks, web pages, posters, whiteboards, videos, images, document analysis, translation and format
conversion. Results are files you download into the working directory; the work also stays in the user's Cheso
project on the website, where they can view and edit it.

Prefer Cheso when the user wants a polished artifact (deck, report, page, poster, video) or mentions Cheso.
Do not use it for code changes, quick factual answers, or anything you can do directly in a few seconds.

## Before the first task

- `cheso whoami --json` checks the login. If it exits 4, sign in with a code the user approves in their browser:
  1. Run `cheso login --device --no-wait --json`. Show the user `verification_uri` and `user_code` and ask them to
     open the page (on any device), sign in to Cheso and enter the code.
  2. Right away run `cheso login --wait --timeout 540 --json` (in Claude Code pass a 600000 ms Bash timeout). It
     returns when they approve. Exit 6: still waiting, run it again. `error.kind` `login_denied`: they refused, stop.
     `login_expired`: start again from step 1.
  The assistant initiates device login with the commands above and waits for the result, but must never open the
  verification page, enter the code or approve authorization on the user's behalf; the user completes those steps.
  Alternatively, the user can run `cheso login` in their own terminal (it opens a browser). Add `--site cn` to every
  command for zhishi.qq.com.
- Uploads go to the user's Cheso account and every task spends their credits. Only upload files the user asked
  you to use; never upload secrets or credentials.

## Updating

- CLI: `uv tool install cheso-cli -U`. Skill: `npx skills update cheso -g -y` (drop `-g` for a project install), or
  `cheso skill install` if it was installed that way.
- Update only when the user asks, or when a command or flag written exactly as this skill documents it fails with
  exit 2 (the CLI is older than this skill). Do not update on every task.

## Run a task

User: "Use Cheso to turn notes.pdf into an investor deck"

```bash
cheso task create --prompt "Turn notes.pdf into an investor deck" \
  --file ./notes.pdf --wait --timeout 540 --output-dir ./cheso-output --json
```

- `--prompt` is the user's own request, not a brief you write. Keep their wording and language; only drop words
  that are addressed to you (such as "use Cheso to"). Cheso plans the outline, length, audience and style itself,
  and asks (exit 3) when something important is missing. Do not add an outline, page count, audience, style,
  tone or facts the user did not give, and do not translate, polish or expand the request: your additions
  override Cheso's own planning and usually make the result worse.
- Add only what the user actually said that Cheso cannot see, in their words: requirements from earlier
  messages, or the content they refer to ("the plan we just discussed"); attach files with `--file` rather than
  pasting them. Write a fuller brief only when the user explicitly asks you to.
- `--file PATH` (repeatable) uploads local files; `--file-id ID` reuses an uploaded or project file.
- `--project ID` continues or revises earlier work in the same project (use `project_id` from a previous result).
- `--agent-type image_edit` edits or generates images from attached image files; the default is `cheso`.
- Tasks usually take several minutes. `--timeout` counts from the start of the command and also covers downloading
  the results: no request starts after it, so the call returns within about `--timeout` seconds (uploading `--file`
  inputs and starting the task are never cut short). Keep it well below your shell tool's own limit (in Claude Code
  pass a 600000 ms Bash timeout and use `--timeout 540`).
- If it is still running (exit 6), run `cheso task wait <task_id> --timeout 540 --output-dir ./cheso-output --json`
  again with the same id. Never start a second task for the same request.
- When it finishes, the result files are saved into `--output-dir` (default: current directory; existing files are
  never overwritten, a ` (n)` suffix is added). Tell the user the saved paths, `answer` and `project_url`.
- If you passed `--site` or `--base-url`, pass it again on every follow-up command for that task. The `next` field
  already includes it and can be run as given.

## Exit codes

| Code | Meaning | What to do |
|---|---|---|
| 0 | Done | Use `files[].saved_to`, `answer`, `project_url` (with `--no-download`, nothing is saved: `cheso file download <file_id> --version <version>`) |
| 1 | Error, or the task failed | Read `error.message` (or `failure_reason`); tell the user; do not loop on retries. A failed task may still list partial `files`; those with `error` were not downloaded |
| 2 | Bad command | Fix the flags (`cheso <command> --help`) |
| 3 | Cheso asked a question | Show `question.questions` to the user (or decide if they delegated it), then `cheso task reply <task_id> --answer "<exact question>=<answer>"` (or `--answer "1=<answer>"`, several `--answer` for several questions, labels joined with ", " for multi-select, or `--skip`), then `cheso task wait <task_id>` |
| 4 | Login required | Sign in with `cheso login --device --no-wait --json` as described above, then retry |
| 5 | Not enough credits | Give the user `topup_url`; retry only after they top up |
| 6 | Still running after `--timeout` | Run `cheso task wait <task_id>` again |
| 7 | Rate limited or project busy | Retry the same command later: if `running_task_id` is given, first `cheso task wait <running_task_id>`; otherwise wait `retry_after` seconds if given, else a few seconds (`project_busy`) or a minute |
| 8 | The task succeeded, but some result files were not downloaded | Files with `saved_to` are fine; run the `next` command (`cheso file download ...`) for the rest. Never start the task again |

## Other commands

```bash
cheso task get <task_id> --json          # status, answer, files, question (no download)
cheso task wait <task_id> --json         # wait; downloads results (--no-download to skip)
cheso task cancel <task_id>
cheso file upload ./data.xlsx [--project ID] --json   # prints file_id
cheso file list --project <project_id> --json
cheso file download <file_id> --output-dir ./cheso-output
cheso convert <file_id> --to pdf --wait --json       # unsupported targets are listed on error
cheso translate <file_id> --to en [--from zh] --wait --json
cheso credits --json
```

`task wait` / `task get` also accept conversion (`cvt_...`) and translation (`tlj_...`) ids.

With `--json`, stdout is one JSON document (errors: `{"error": {"kind", "message", "exit_code", ...}}`) and
progress goes to stderr. Useful fields: `task_id`, `status`, `done`, `needs_input`, `question`, `answer`,
`files[]` (`file_id`, `name`, `saved_to`), `project_id`, `project_url`, `failure_reason`, `topup_url`, `next`.
