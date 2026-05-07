# Front Of House CLI

AI-operator provisioning CLI for Front Of House.

Public mirror: https://github.com/iiko38/front-of-house-cli

Current published baseline: `@f-o-h/cli@0.1.76`

This mirror is a generated release artifact. The private product monorepo is not
published here, and no open-source license is granted unless stated separately.

## Install

Use npm/npx. Do not clone the private monorepo for public setup.

```bash
npx --yes @f-o-h/cli@latest --version
npx --yes @f-o-h/cli@latest start
```

For repeat use:

```bash
npm install -g @f-o-h/cli@latest
foh --help
```

## First Customer Path

This is the path a blank AI coding agent or developer should try first:

```bash
npx --yes @f-o-h/cli@latest auth signup --web --json
npx --yes @f-o-h/cli@latest auth login --web --json
npx --yes @f-o-h/cli@latest setup --phone-mode observe --json
npx --yes @f-o-h/cli@latest prove --agent <agent-id> --mission widget --json
npx --yes @f-o-h/cli@latest publish --agent <agent-id> --json
```

`publish` consumes existing release evidence. If evidence is missing or stale,
it blocks with a reason code and `next_commands`; follow those commands exactly.

## Common Missions

| Mission | Command |
|---|---|
| Start | `foh start` |
| Setup | `foh setup --phone-mode observe --json` |
| Prove | `foh prove --agent <agent_id> --mission widget --json` |
| Publish | `foh publish --agent <agent_id> --json` |
| Debug | `foh debug --out test-results/foh-cli-diag.latest.json --json` |

For AI agents: prefer `--json`, preserve command outputs as evidence, and follow
`next_commands` exactly when a command blocks.

## Cost-Safe Phone Setup

Use the free scaffold lane for demos and repeated AI-agent tests:

```bash
FOH_CLI_SPEND_POLICY=no_spend foh setup --phone-mode observe --json
```

`--phone-mode observe` checks whether a contact phone already exists without
buying one. `--phone-mode skip` bypasses the phone step. `--phone-mode purchase`
is the explicit paid contact path and is fail-closed when
`FOH_CLI_SPEND_POLICY=no_spend` is set.

Attach a customer-owned voice number without FOH purchasing inventory:

```bash
foh provision byon attach --phone-number <e164> --confirm-owned --org <org-id> --json
```

The attach command requires explicit ownership confirmation, writes the org's
voice channel, returns provider webhook URLs, and preserves the cost boundary:
the customer owns the number/provider account.

## Auth Modes

`auth signup --web` opens the console signup page when possible and always
prints the fallback URL. `auth login --web` starts browser device authorization,
opens `/cli-auth`, waits for console approval, and stores a short-lived token.
Credential auth remains available as fallback.

Trusted server-side automation can use a scoped service token without browser
approval:

```bash
FOH_SERVICE_TOKEN="$FOH_SERVICE_TOKEN" FOH_ORG_ID="$FOH_ORG_ID" foh auth whoami --json
```

`auth whoami --json` includes `auth.expires_at` and `auth.token_ttl_seconds`
without exposing the token.

## Package Artifacts

Package-local examples and schemas ship with the npm artifact:

- `examples/scenario-suite.viewing.yml`
- `examples/proof-report.example.json`
- `examples/transcript-export.example.json`
- `examples/improvement-packet.example.json`
- `examples/external-agent-run.example.json`
- `schemas/cli-envelope.schema.json`
- `schemas/scenario-suite.schema.json`
- `schemas/transcript-export.schema.json`
- `schemas/improvement-packet.schema.json`
- `schemas/external-agent-run.schema.json`

## Advanced Operator Surfaces

These commands are intentionally outside the first customer path:

- `foh certify run --agent <agent_id> --profile release --json` produces release evidence when `publish` asks for it.
- `foh bug improve --from-file <artifact.json> --json` converts FOH product/docs/runtime failures into redacted improvement packets.
- `foh eval external-agent ...` runs the controlled external-agent benchmark harness.
- `foh sim ...` and Dojo-related certification flows are platform/operator testing surfaces.
## Local Scenario Suites

`foh test run --suite <file>` runs deterministic widget-runtime checks for a
specific agent. The suite format supports reply text checks plus structured
runtime assertions for trace/correlation IDs, action or terminal state, latency,
variables, tool calls, escalation/handoff, lead capture, and exact response
field paths.

```yaml
agent: agent_123
scenarios:
  - id: viewing
    turns:
      - user: Can I book a viewing this week?
        expect:
          contains: viewing
          trace_present: true
          correlation_present: true
          action: text
          latency_ms:
            max: 3000
```

Use transcript fixtures when turning real user conversations into regression
tests:

```yaml
agent: agent_123
scenarios:
  - id: replay-viewing
    fixture_transcript: ./fixtures/viewing-transcript.json
```

## Transcript Export

Use hydrated transcript export to turn real behavior into replay/debug artifacts:

```bash
foh transcripts export \
  --agent <agent-id> \
  --hydrate \
  --include-traces \
  --format json \
  --out foh-transcripts.json \
  --json
```

Exports redact obvious emails, phone numbers, and secret-like tokens by default.
Each exported conversation includes a `replay_command` and `test_fixture` seed
so operators or AI agents can move from observed failure to replay or scenario
regression without opening the console.

Replay a local export without API access:

```bash
foh agent replay --file foh-transcripts.json --json
```

## Improvement Packets

Use `foh bug improve` when a setup, proof, replay, knowledge, runtime, or
live-proof failure should become actionable backlog/test/config/docs work:

```bash
foh bug improve \
  --from-file test-results/proof-or-replay-failure.json \
  --out test-results/improvement-packet.json \
  --json
```

The command emits a redacted `foh_improvement_packet.v1` with stable IDs,
reason code, promotion decision, evidence summary, and deterministic next
commands.


