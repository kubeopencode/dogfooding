# KubeOpenCode Dogfooding

Deployment configurations for the [KubeOpenCode](https://github.com/kubeopencode/kubeopencode) dogfooding environment, where KubeOpenCode is used to automate tasks on its own repository.

## Background

This repository was extracted from `kubeopencode/kubeopencode`'s `deploy/dogfooding/` directory ([commit b5b20bc](https://github.com/kubeopencode/kubeopencode/commit/b5b20bc)) for:

- **Security** — avoid plaintext secrets in the main project's git history
- **Separation of concerns** — deployment-specific configs don't belong in project source

## Structure

```
dogfooding/
├── base/           # Core resources (agents, contexts, RBAC, namespace)
├── github/         # GitHub webhook integration (Argo Events, smee-client)
├── scheduled/      # CronJobs (opencode update, tiny refactor)
└── slack/          # Slack bot integration (socket-mode gateway)
```

See [`dogfooding/README.md`](dogfooding/README.md) for detailed architecture and deployment instructions.
