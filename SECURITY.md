# Security Policy

## Reporting a vulnerability

Please report suspected security issues privately. Do not open a public issue,
pull request, or discussion for a vulnerability.

Email migueldiaznunez@gmail.com with:

- A description of the issue and its impact
- The affected files, configuration, or version
- Steps to reproduce, if you have them
- Any suggested fix, if you have one

You should receive an acknowledgement within 7 days. If the report is accepted,
a fix or mitigation note will follow as soon as practical. Please give the
maintainer a reasonable window to respond before any public disclosure.

This repository contains deployment configuration, not application source.
Reports are in scope when they affect this Compose stack, its default
configuration, or the way secrets and published ports are handled here.
Vulnerabilities in upstream images (LiteLLM, PostgreSQL, vLLM, Qdrant) should
also be reported to those projects.

## Operator guidance

The stack is intended for a trusted host. Treat the published endpoints as
administrative interfaces.

- Copy `.env.example` to `.env` and replace every default secret before the
  first start. `LITELLM_MASTER_KEY` must start with `sk-`. Generate one with
  `task key`.
- Never commit `.env`, API keys, database passwords, or Hugging Face tokens.
- LiteLLM listens on port `4000` and Qdrant on port `6333`, on all host
  interfaces. Do not expose those ports to an untrusted network. Bind them to
  localhost or place them behind a firewall or reverse proxy with
  authentication if the host is reachable by others.
- The LiteLLM UI password and the master key are equivalent to administrative
  credentials. Rotate them if they leak.
- PostgreSQL is not published to the host. Leave it that way unless you have a
  specific need, and then bind it to `127.0.0.1` only.
- Model weights are read from the host Hugging Face cache. That cache can
  contain access tokens. Mount only the cache you intend the containers to see.
- Pull images by the pinned tags in `compose.yml` and review upstream security
  advisories before moving to a newer tag.

## Supported versions

There are no numbered releases. Security fixes apply to the current `master`
branch.
