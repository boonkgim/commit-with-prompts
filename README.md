# commit-with-prompts

A [Claude Code](https://claude.com/claude-code) skill that commits your work with a
commit message documenting **the prompts that produced it**.

Instead of a commit message that describes the diff, you get one that records the
conversation: every prompt you typed, verbatim and in order, each followed by a short
summary of what changed in response. The result is a durable record of *why* the code
looks the way it does — readable in `git log` months later, and reviewable in a PR.

## Example

```
Add rate limiting to the upload endpoint

--

add a rate limit to the upload endpoint, 10 per minute per user

Added `RateLimiter` in src/middleware/rate_limit.py and wired it into the
`/upload` route in src/api/routes.py.

--

use redis instead of in-memory so it works across workers

Swapped the in-memory counter for a Redis-backed sliding window; added
`REDIS_URL` to config with a localhost default.

--
```

## Install

Clone the repo and symlink it into your skills directory:

```bash
git clone https://github.com/boonkgim/commit-with-prompts.git
ln -s "$PWD/commit-with-prompts" ~/.claude/skills/commit-with-prompts
```

Use `.claude/skills/` inside a project instead of `~/.claude/skills/` to scope it to a
single repo. Restart Claude Code (or start a new session) to pick it up.

## Usage

Ask for a commit in the normal way, or invoke it explicitly:

```
/commit-with-prompts
```

The skill stages what the conversation touched, writes the message, and commits. It does
not push unless you ask.

## What it does to your prompts

Prompts are reproduced verbatim, with three deliberate exceptions:

- **Grammar and spelling are corrected** — typos and capitalization only. Wording, order,
  and intent stay yours.
- **Sensitive values are masked** — API keys, tokens, passwords, connection strings,
  private hostnames, personal data, and flagged third-party names become bracketed
  placeholders like `[API_KEY]` or `[EMAIL]`. When in doubt, it masks.
- **Long pasted logs are summarized** — a pasted blob over ~15 lines is replaced by a
  bracketed summary line so no reader mistakes the condensation for your words. Your own
  prose around it stays verbatim.

Masking is best-effort, not a secret scanner. Review the message before pushing a commit
from a conversation that handled credentials.

## Caveats

- Commit messages get long. That is the point, but it makes them a poor fit for repos
  that enforce a strict message format or a hard body-length lint.
- Rewriting history (`rebase -i`, squash merges) collapses the prompt log along with
  everything else. Squash-merge repos will lose per-commit detail.
- The prompt log *is* the attribution — the skill deliberately adds no `Co-Authored-By`
  or tool-attribution trailers.

## License

MIT — see [LICENSE](LICENSE).
