# Front Of House CLI

AI-operator provisioning CLI for Front Of House.

Public mirror: https://github.com/iiko38/front-of-house-cli

Current published baseline: `@f-o-h/cli@0.1.1`

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
foh auth login --web
foh auth login
foh org list
foh org use --org <org-id>
foh setup
```

For AI agents and text-only terminals:

```bash
foh auth login --web --json
foh auth login --email "$FOH_EMAIL" --password "$FOH_PASSWORD" --json
foh org list --json
foh org use --org <org-id> --json
foh setup --org <org-id> --agent-template <template-id> --agent-name "Demo Agent" --json
```

`auth login --web` opens the console sign-in page when possible and always
prints the fallback URL. The CLI still requires explicit credential auth until
device-code browser-token exchange is available.

The CLI defaults to the production API at `https://api.frontofhouse.okii.uk`.
