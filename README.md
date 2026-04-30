# Front Of House CLI

AI-operator provisioning CLI for Front Of House.

Public mirror: https://github.com/iiko38/front-of-house-cli

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

## First Run

```bash
foh auth login
foh org list
foh org use --org <org-id>
foh setup
```

The CLI defaults to the production API at `https://api.frontofhouse.okii.uk`.
