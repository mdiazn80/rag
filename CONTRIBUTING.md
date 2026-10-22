# Contributing

Thanks for considering a contribution. This repository holds the Compose stack
and configuration for a local RAG deployment: a LiteLLM proxy, vLLM model
servers, PostgreSQL, and Qdrant.

## Before you start

- Read the [Code of Conduct](CODE_OF_CONDUCT.md).
- Report security issues privately, as described in [SECURITY.md](SECURITY.md).
  Do not file them as public issues.
- Check existing issues before opening a new one.

## Local checks

Install [Task](https://taskfile.dev), Docker Compose v2, and the NVIDIA
Container Toolkit. Then:

```bash
cp .env.example .env
task doctor
task config
```

`task config` prints the rendered Compose file, including secrets from `.env`.
Do not paste that output into an issue.

## Pull requests

- Keep changes focused. A pull request should do one thing.
- Prefer pinned image tags over floating tags.
- Document new services, ports, environment variables, and Task targets in
  `README.md`.
- Do not commit `.env`, credentials, tokens, or model weights.
- Write commit messages and discussion in English.

## Configuration changes

`config.yaml` is the LiteLLM model map. `compose.yml` is the runtime. If you
add a model server, wire it in both files and give it a stable
`--served-model-name` that matches the LiteLLM `model_name`.
