# Front Of House CLI

AI-operator provisioning CLI for Front Of House.

Public mirror: https://github.com/iiko38/front-of-house-cli

Current published baseline: `@f-o-h/cli@0.1.48`

This mirror is a generated release artifact. The private product monorepo is not
published here, and no open-source license is granted unless stated separately.

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

## Install

```bash
npx @f-o-h/cli setup
```

Or install globally:

```bash
npm install -g @f-o-h/cli
foh --help
```

Verify the package version:

```bash
npx @f-o-h/cli --version
```

## First Run

```bash
foh auth signup --web
foh auth login --web
foh auth login
foh org list
foh org use --org <org-id>
foh setup
foh prove --agent <agent-id> --json
```

For AI agents and text-only terminals:

```bash
foh auth signup --web --json
foh auth login --web --json
foh auth login --email "$FOH_EMAIL" --password "$FOH_PASSWORD" --json
foh org list --json
foh org use --org <org-id> --json
FOH_CLI_SPEND_POLICY=no_spend foh setup --org <org-id> --agent-template <template-id> --agent-name "Demo Agent" --phone-mode observe --json
foh prove --agent <agent-id> --json --out foh-proof.json
foh test run --suite ./suite.yml --agent <agent-id> --json --out foh-test-report.json
foh agent replay --file ./transcript-export.json --json
foh bug improve --from-file foh-proof.json --out foh-improvement.json --json
```

Trusted server-side automation can use a scoped service token without browser
approval:

```bash
FOH_SERVICE_TOKEN="$FOH_SERVICE_TOKEN" FOH_ORG_ID="$FOH_ORG_ID" foh auth whoami --json
```

`auth whoami --json` includes `auth.expires_at` and `auth.token_ttl_seconds`
without exposing the token. Refresh short-lived eval tokens before long
external-agent runs; controlled live eval execution fails closed when explicit
`FOH_EXTERNAL_AGENT_EVAL_*` credentials are missing, expired, or expire before
the run budget.

`auth signup --web` opens the console signup page when possible and always
prints the fallback URL. `auth login --web` starts browser device
authorization, opens `/cli-auth`, waits for console approval, and stores the
returned short-lived token. Credential auth remains available as fallback.

`foh prove` produces a compact signed proof report across auth, org context,
agent validation, contact phone readiness, voice provider health, widget
channel/embed readiness, and widget smoke. It does not run release
certification by default; run `foh certify run --agent <id> --profile release`
before publish, or pass `--include-certification` only when you intentionally
want the slower certification check inside proof. It is read-only by default;
pass `--mutation-mode ensure` or `--repair` only when you explicitly want proof
to ensure missing widget state. Use `--strict` in automation when holds should
fail the command, and `--mission voice` or `--require-phone` when a
voice/contact number is mandatory for the demo.
Use `--contact-path byon` when the proof is meant to validate a
customer-owned/BYON phone route; missing BYON config then reports
`byon_voice_number_not_configured` instead of suggesting FOH number purchase.
For repeated or parallel explicit-certification proof missions in AI-agent
evals, pass `--include-certification --proof-cache-dir .foh/proof-cache` so
simulation certification runs once and sibling proofs reuse the same signed
certification detail.

For mass AI-agent evals and repeated demo rehearsals, keep setup on the free
scaffold lane:

```bash
FOH_CLI_SPEND_POLICY=no_spend foh setup --org <org-id> --agent-template <template-id> --agent-name "Demo Agent" --phone-mode observe --json
```

`--phone-mode observe` checks whether a contact phone already exists without
buying one. `--phone-mode skip` bypasses the phone step. `--phone-mode purchase`
is the explicit paid contact path and is fail-closed when
`FOH_CLI_SPEND_POLICY=no_spend` is set.

If managed-number provisioning is blocked by account/provider capacity or empty
reserve inventory, proof reports `provider_capacity_blocked`; fix capacity or
switch to BYON rather than retrying blindly.

Attach a customer-owned voice number without FOH purchasing inventory:

```bash
foh provision byon attach --phone-number <e164> --confirm-owned --org <org-id> --json
```

The attach command requires explicit ownership confirmation, writes the org's
voice channel, returns Twilio webhook URLs, and preserves the cost boundary:
the customer owns the number/provider account.

The CLI defaults to the production API at `https://api.frontofhouse.okii.uk`.

## External-Agent Eval Capture

Use this when testing whether a clean coding agent can start from public docs
and the public npm package without private repo context:

```bash
foh eval external-agent batch \
  --models openai/codex,anthropic/claude,cursor/agent \
  --prompt-version blank-setup.v1 \
  --json
```

Run each returned launch command in a clean agent terminal:

```bash
foh eval external-agent run \
  --model-provider openai \
  --model-name codex \
  --prompt-version blank-setup.v1
```

The command writes a versioned prompt, launches an instrumented shell, captures
FOH CLI commands into `commands.ndjson`, and finalizes `run.json` as an
`external_agent_run.v1` artifact when the shell exits.

Run artifacts include `eval_state` so repeated benchmark runs make lifecycle
intent explicit: reuse the existing eval org, agent, and widget by default;
create ephemeral orgs only for explicit lifecycle tests; do not create paid
phone resources unless a BYON/customer-owned or operator-approved paid lane is
selected.

For no-spend evals, inspect reusable state before creating anything:

```bash
foh org status --json
foh agent list --json
```

If a reusable eval agent already exists, configure and prove that agent rather
than creating a second bronze-tier agent.

## Common Missions

| Mission | Command |
|---|---|
| Start | `foh start` |
| Setup | `foh setup --phone-mode observe --json` |
| Prove | `foh prove --agent <agent_id> --mission widget --json` |
| Certify | `foh certify run --agent <agent_id> --profile release --json` |
| Debug | `foh debug --out test-results/foh-cli-diag.latest.json --json` |
| Platform feedback | `foh bug improve --from-file <artifact.json> --json` |
| Publish | `foh agent publish --agent <agent_id> --json` |

For a planted knowledge-miss benchmark:

```bash
foh eval external-agent batch \
  --models openai/codex \
  --prompt-version knowledge-miss.v1 \
  --knowledge-question "Does 12 Acacia Avenue allow Saturday viewings?" \
  --expected-answer "Saturday viewings are available by appointment only." \
  --json
```

The prompt instructs the clean agent to run `foh knowledge query`, preserve the
failure packet, and convert it with `foh bug improve`.

For guarded programmable-runner planning:

```bash
foh eval external-agent execute \
  --runner codex \
  --batch test-results/external-agent-runs/<batch>/batch.json \
  --codex-model gpt-5.3-codex \
  --dry-run \
  --json
```

This writes `executor-plan.json`, creates intentionally empty clean workspaces
outside the private repo, validates the local Codex binary/help flags, and
prints exact `codex exec` commands without executing them. Use `--codex-model`
to compare the same prompt across explicit Codex models; the selected runner
model is recorded in `executor-plan.json` and controlled-run `run.json`.

Before promoting run artifacts, scan and redact them:

```bash
foh eval external-agent scan-artifacts \
  --run-dir test-results/external-agent-runs/<batch>/<run-id> \
  --private-repo-root <private-repo-root> \
  --write-redacted \
  --json
```

After dry-run review, one controlled Codex run can be launched explicitly with
`--live`. Live mode is intentionally limited to one run per batch and finalizes
`run.json` even on timeout or non-zero exit:

```bash
foh eval external-agent execute \
  --runner codex \
  --batch test-results/external-agent-runs/<one-model-batch>/batch.json \
  --timeout-minutes 30 \
  --live \
  --json
```

For Linux eval hosts where Codex's default Bubblewrap sandbox cannot configure
loopback networking, keep sandboxing enabled and select the legacy Landlock
backend explicitly:

```bash
foh eval external-agent execute \
  --runner codex \
  --batch test-results/external-agent-runs/<one-model-batch>/batch.json \
  --codex-sandbox-backend legacy-landlock \
  --codex-network-access \
  --timeout-minutes 30 \
  --live \
  --json
```

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
