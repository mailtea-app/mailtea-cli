# Mailtea CLI

Send, schedule, and manage email with [Mailtea](https://mailtea.app) from your
shell or an AI agent. Wraps the [`mailtea-sdk`](https://www.npmjs.com/package/mailtea-sdk)
SDK for everyday commands, and bridges the full
[`mailtea-mcp`](https://www.npmjs.com/package/mailtea-mcp) tool catalog so an agent
can reach **every** Mailtea capability — the same surface as the MCP server.

This repository hosts the CLI's documentation and issue tracker. The CLI itself
is distributed as the [`mailtea-cli`](https://www.npmjs.com/package/mailtea-cli)
npm package.

## Installation

```bash
npm install -g mailtea-cli   # installs the `mailtea` command
# or run one-off without installing:
npx -y mailtea-cli <command>
```

Requires Node.js 18+.

## Quick start

```bash
npx -y mailtea-cli emails send \
  --from "you@yourdomain.com" \
  --to "user@example.com" \
  --subject "Hello" \
  --html "<p>Sent from the Mailtea CLI</p>"
# -> {"id":"txemail_..."}
```

Authenticate with a personal access token (prefix `mt_pat_`) from **Settings →
API keys**, supplied any of three ways (in order):

1. `--api-key mt_pat_xxx`
2. `MAILTEA_API_KEY` (or `MAILTEA_API_TOKEN`) environment variable
3. `mailtea login --api-key mt_pat_xxx` (saved to `~/.mailtea/config.json`)

Self-hosting or local dev? add `--base-url http://localhost:8787` (or
`MAILTEA_API_BASE_URL`).

## Full capability bridge (every MCP tool)

Beyond the named commands below, the CLI exposes the **entire MCP tool catalog** —
issue drafting/scheduling/sending, analytics, sections, referrals, monetization,
AI drafting, and more — through three generic commands:

```bash
mailtea tools [--names]                 # list every capability (no auth)
mailtea schema <tool>                   # print a tool's JSON input schema (no auth)
mailtea call <tool> [--json '{...}'] [--arg key=value ...]   # invoke any tool
```

```bash
# discover, inspect, then call — exactly how an agent operates
mailtea tools --names | jq -r '.[]' | grep '^issue\.'
mailtea schema issue.send_now
mailtea call issue.send_now --json '{"issueId":"iss_123"}'

# --arg overlays string values onto the --json base
mailtea call email.send --json '{"from":"you@d.com","to":"u@e.com","subject":"Hi"}' --arg html='<p>hi</p>'
```

`mailtea call` prints the tool's structured result on stdout; a tool/API error is
JSON on stderr with a non-zero exit code. New MCP tools appear automatically.

## Named commands

```bash
mailtea emails send --from <a> --to <a> --subject <s> (--html <h> | --text <t> | --template-id <id>)
mailtea emails batch --json '[{...}, ...]'
mailtea emails list [--status <s>] [--limit <n>] [--offset <n>] [--tag-name <n>] [--tag-value <v>]
mailtea emails get|cancel <id>
mailtea emails reschedule <id> --scheduled-at <iso>
mailtea emails analytics [--from-date <iso>] [--to-date <iso>]
mailtea posts create --publication-id <pub> --subject <s> (--html <h> | --template-id <id>) [--text <t>] [--from <a>] [--reply-to <a>] [--name <n>] [--kind newsletter|broadcast]
mailtea posts list --publication-id <pub> [--limit <n>] [--offset <n>] [--status <s>] [--kind newsletter|broadcast]
mailtea posts get|delete <post-id>
mailtea posts update <post-id> [--subject <s>] [--html <h>] [--text <t>] [--from <a>] [--reply-to <a>] [--name <n>]
mailtea posts send <post-id> [--scheduled-at <iso>]
mailtea posts send-test <post-id> --to <addr> --from <addr> [--reply-to <addr>]

mailtea contacts create|list|get|update|delete --publication-id <pub> ...
mailtea segments create|list|get|update|delete --publication-id <pub> ...
mailtea tags create|list|get|update|delete --publication-id <pub> ...
mailtea senders create|list|get|update|delete --publication-id <pub> [--name <n>] [--email <a>] [--reply-to <a>] [--is-default]
mailtea suppressions add|remove --email <a> [--email <a> ...] [--reason <r>]
mailtea suppressions list [--reason <r>] [--q <search>] [--created-after <iso>] [--created-before <iso>] [--limit <n>] [--starting-after <c>]
mailtea suppressions export
mailtea domains create|list|get|verify|update|delete --publication-id <pub> ...
mailtea domains tracking create|list|verify|delete --publication-id <pub> --domain-id <id> ...
mailtea webhooks create|list|get|update|delete --publication-id <pub> [--event <e> ...]
mailtea contact-properties create|list|update|delete ...        # team-scoped, no --publication-id
mailtea api-keys create|list|revoke ...
mailtea templates create|list|get|update|publish|duplicate|delete --publication-id <pub> ...
mailtea inbound list --publication-id <pub> [--limit <n>] [--cursor <c>]
mailtea inbound get|attachments <id>
mailtea inbound attachment <id> <attachment-id>
mailtea inbound reply <id> (--html <h> | --text <t>) [--from <addr>] [--from-name <n>] [--subject <s>] [--cc <a>] [--bcc <a>] [--idempotency-key <k>]

mailtea login --api-key mt_pat_xxx [--base-url <url>]
mailtea version | help
```

`emails send` extras: `--cc` / `--bcc` / `--reply-to` (repeatable or comma-separated),
`--scheduled-at <iso>`, `--tag name=value`, `--header name=value`,
`--template-var name=value` (all repeatable), and `--dry-run` to print the
resolved payload without sending.

## Agent-friendly

Output is **JSON on stdout**; errors are **JSON on stderr** with a non-zero exit
code — so an agent in a non-TTY shell can parse results and detect failures.

```bash
ID=$(mailtea emails send --from you@d.com --to u@e.com --subject Hi --text hi | jq -r .id)
mailtea emails get "$ID" | jq -r .last_event
```

## Support

Found a bug or missing a command? [Open an issue](https://github.com/mailtea-app/mailtea-cli/issues).

## License

The `mailtea-cli` npm package is released under the [MIT License](./LICENSE).
