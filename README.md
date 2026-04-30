# Front Of House CLI

AI-operator provisioning CLI for Front Of House.

Public mirror: https://github.com/iiko38/front-of-house-cli

Current published baseline: `@f-o-h/cli@0.1.5`

This mirror is a generated release artifact. The private product monorepo is not
published here, and no open-source license is granted unless stated separately.

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
foh setup --org <org-id> --agent-template <template-id> --agent-name "Demo Agent" --json
foh prove --agent <agent-id> --json --out foh-proof.json
```

`auth signup --web` opens the console signup page when possible and always
prints the fallback URL. `auth login --web` starts browser device
authorization, opens `/cli-auth`, waits for console approval, and stores the
returned short-lived token. Credential auth remains available as fallback.

`foh prove` produces a compact signed proof report across auth, org context,
agent validation, contact phone readiness, voice provider health, widget
channel/embed readiness, widget smoke, and simulation certification. Use
`--strict` in automation when holds should fail the command, and
`--require-phone` when a voice/contact number is mandatory for the demo.

The CLI defaults to the production API at `https://api.frontofhouse.okii.uk`.

