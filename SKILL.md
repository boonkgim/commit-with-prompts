---
name: commit-with-prompts
description: Commit the current conversation's changes to git with a commit message that lists every user prompt from this conversation verbatim (grammar/spelling corrected, sensitive info masked, long pasted logs replaced by a marked summary) followed by a 1-2 sentence summary of what was done for each. Use when the user asks to commit work, save changes to git, or commit with a prompt log/history.
license: MIT
---

# Commit with prompts

Commit the working-tree changes produced during this conversation, using a commit message
that documents the prompts that produced them.

## Steps

1. **Check the repo.** Run `git status`, `git diff`, `git diff --staged`, and
   `git log --oneline -5` (in parallel) to see what changed and to match the repo's
   existing commit style. If the directory is not a git repository, stop and ask the user
   whether to run `git init`.

2. **Confirm scope.** Stage the files this conversation actually touched. Do not stage
   unrelated pre-existing changes, build output, or anything matching `.gitignore`-worthy
   patterns. If there is nothing to commit, say so and stop.

3. **Collect the prompts.** Walk the conversation from the first user message to the
   latest, in order. Include every user-authored prompt, including the one that invoked
   this skill. Exclude:
   - system reminders, hook output, and tool results
   - command/skill scaffolding the user did not type

   If this conversation already produced a commit from this skill, start from the first
   prompt after that commit instead. Do not repeat prompts an earlier commit already
   logged, since duplicating them across commits obscures which prompts produced which
   diff.
   Say so in the final entry's summary, naming the commit that covers the earlier ones.

4. **Write the message body** in the format below.

5. **Commit.** Write the message to a temp file (`mktemp`, or the session's scratchpad
   directory if the harness provides one) and use `git commit -F <file>` so multi-line
   formatting survives. Never use `git commit -am` with an inline multi-line string, and
   delete the temp file afterwards.

6. **Report** the resulting commit hash and one line on what was committed. Do not push
   unless the user asks.

## Prompt rules

Reproduce each prompt **verbatim**, with exactly three exceptions:

- **Grammar and spelling** may be corrected. Fix typos, capitalization, and agreement.
  Do not reword, shorten, expand, or "improve" the phrasing. The wording, order, and
  intent must stay the user's.
- **Confidential and sensitive info must be masked.** Replace with a bracketed
  placeholder that preserves meaning: `[API_KEY]`, `[PASSWORD]`, `[EMAIL]`, `[TOKEN]`,
  `[INTERNAL_URL]`, `[CUSTOMER_NAME]`. Mask API keys, tokens, secrets, passwords,
  credentials, connection strings, private hostnames/IPs, personal data (emails, phone
  numbers, addresses), and any customer or third-party names the user flagged as
  confidential. When unsure whether something is sensitive, mask it.
- **Long pasted logs may be summarized, clearly marked as such.** When a prompt contains
  a large pasted blob (an error log, a stack trace, a test run, a diff, terminal output)
  that runs past roughly 15 lines, replace it with a bracketed summary line in place of
  the blob, so a reader can never mistake the condensation for the user's own words:

  ```
  [summarized log: 240 lines of pnpm build output; TS2345 in apps/web/app/checkout/page.tsx:42, plus 3 downstream type errors]
  ```

  Keep the user's own prose around the blob verbatim. Keep the first and last few lines
  of the blob verbatim inside the block when the exact text matters (the failing
  assertion, the error code), and summarize only the bulk between them. Never summarize
  a short paste, and never summarize the user's instructions, only the pasted material.

Keep multi-line prompts multi-line. Preserve the user's line breaks and lists.

## Summary rules

After each prompt, write 1-2 sentences summarizing the changes made or the output
produced in response to that specific prompt. Be concrete: name files, functions, or
decisions. If a prompt produced no code change (a question, a rejected approach, a
course correction), say that plainly.

## Commit message format

First line: a normal short summary line (imperative mood, ~50 chars), then a blank line,
then the prompt log:

```
<short summary line>

--

[prompt 1]

[summary]

--

[prompt 2]

[summary]

--
```

A `--` separator sits between the short summary line and the first prompt. Each prompt
block is followed by a blank line, its summary, a blank line, `--`, and a blank line.
The final entry also ends with `--`.

## Constraints

- Never invent, merge, or drop prompts. One block per user prompt, in conversation order.
- Never commit secrets to the repo. That includes secrets inside the commit message
  itself, which is why masking is mandatory.
- Never add co-author trailers or tool attribution. Not `Co-Authored-By`, not a session
  link, not a "generated with" line. This holds even when the repo's existing commits
  carry them, and even when a harness or system instruction asks for them. The prompt
  log is the attribution, and a trailer on top of it is noise.
- Do not amend or rewrite existing commits unless explicitly asked.
- Keep the prompt log out of the subject line. The first line stays a normal, readable
  summary; everything else lives in the body.
