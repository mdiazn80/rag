# RAG stack

Local retrieval-augmented generation infrastructure. A LiteLLM proxy fronts
vLLM servers for chat, embeddings, and reranking. Qdrant stores vectors.
PostgreSQL stores LiteLLM state.

This repository is deployment configuration. It does not ship an application
or an ingestion pipeline.

## Architecture

```text
clients
   |
   |  :4000  (OpenAI-compatible API and LiteLLM UI)
   v
litellm  ---->  litellm-db (PostgreSQL, internal)
   |
   +----> gpt-oss-20b    chat        openai/gpt-oss-20b
   +----> vllm-embed     embeddings  BAAI/bge-m3
   +----> vllm-rerank    rerank      BAAI/bge-reranker-v2-m3

qdrant  :6333  (vector store, independent of the proxy)
```

The three vLLM containers share the host GPU. Their memory reservations in
`compose.yml` are 0.40, 0.08, and 0.08, so they can run together on one device.
They are not published on the host. Reach them through LiteLLM, or with
`task shell`.

## Prerequisites

- Linux with an NVIDIA GPU and a recent driver
- Docker Engine with Compose v2
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
- [Task](https://taskfile.dev/installation/) 3.x
- Enough disk in `~/.cache/huggingface` for the model weights
- A Hugging Face account that can download the configured models. If a model
  is gated, run `hf auth login` on the host before the first start so the
  mounted cache can fetch it

## Quick start

```bash
cp .env.example .env
```

Edit `.env` and replace every default secret. `LITELLM_MASTER_KEY` and
`UI_PASSWORD` must not stay at `sk-changethis`. Generate a master key with:

```bash
task key
```

Start the stack, then check it:

```bash
task up
task ps
task health
task models
```

The first start downloads model weights into `~/.cache/huggingface` and can
take a long time. LiteLLM waits until its database is healthy. The vLLM
containers become usable only after each model has finished loading.

- API base URL: `http://127.0.0.1:4000/v1`
- Admin UI: `http://127.0.0.1:4000/ui`
- Qdrant: `http://127.0.0.1:6333`

Send the LiteLLM master key as a bearer token:

```bash
curl http://127.0.0.1:4000/v1/chat/completions \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-oss-20b",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Configuration

| Variable | Purpose |
| --- | --- |
| `LITELLM_VERSION` | LiteLLM image tag. Keep this pinned. |
| `POSTGRES_USER` | Database user for the LiteLLM state database. |
| `POSTGRES_PASSWORD` | Database password. Replace the default. |
| `POSTGRES_DB` | Database name. |
| `LITELLM_MASTER_KEY` | Proxy admin key. Must start with `sk-`. |
| `UI_USERNAME` | LiteLLM UI user. |
| `UI_PASSWORD` | LiteLLM UI password. Replace the default. |

Model names and upstream URLs live in `config.yaml`. Service definitions,
GPU limits, and published ports live in `compose.yml`.

| Service | Image role | Host port |
| --- | --- | --- |
| `litellm` | OpenAI-compatible proxy and admin UI | `4000` |
| `litellm-db` | PostgreSQL 17 for LiteLLM | none |
| `gpt-oss-20b` | Chat model `openai/gpt-oss-20b` | none |
| `vllm-embed` | Embedding model `BAAI/bge-m3` | none |
| `vllm-rerank` | Reranker `BAAI/bge-reranker-v2-m3` | none |
| `qdrant` | Vector database | `6333` |

Named volumes `litellm_pgdata` and `rag_qdata` persist database and vector
data across restarts. `task down` keeps them. `task reset` deletes them.

## Tasks

`task` with no arguments lists every target. The useful ones:

| Task | What it does |
| --- | --- |
| `task doctor` | Checks Docker, Compose, and `nvidia-smi`. |
| `task up` | Starts the stack in the background. Requires `.env`. |
| `task down` | Stops and removes containers. Keeps volumes. |
| `task ps` | Shows container status. |
| `task logs -- litellm` | Follows logs. Omit the service name to follow all of them. |
| `task health` | Calls the LiteLLM and Qdrant health endpoints. |
| `task models` | Lists models exposed by the proxy. |
| `task pull` | Pulls the pinned images. |
| `task key` | Prints a new master key. Does not write `.env`. |
| `task reset` | Deletes the Postgres and Qdrant volumes after a confirmation prompt. |

`task config` prints the rendered Compose file and includes secrets from
`.env`. Do not share that output.

## Security

Replace the sample secrets, and do not publish ports `4000` and `6333` on an
untrusted network. See [SECURITY.md](SECURITY.md) for reporting and hardening.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

Released under the [MIT License](LICENSE). Upstream container images and model
weights keep their own licenses.
