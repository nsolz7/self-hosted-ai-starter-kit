# Local-only Hardening Audit

This document records the outbound-communication audit and the hardening changes applied to this repository on 2026-03-29.

## Scope

Audited repository content:

- `docker-compose.yml`
- `.env.example`
- `README.md`
- bundled n8n demo workflow and credentials metadata under `n8n/demo-data/`
- bundled static page under `shared/extracted-images/chat.html`

Not fully auditable from this repository:

- the optional `admo` sibling repository referenced by `ADMO_REPO_PATH`
- upstream container image contents beyond their published source and documentation

## Hardening changes applied

- Runtime traffic is split between an internal `backend` network and a host-facing `edge` network.
- The `backend` network is `internal`, and host-facing services keep `backend` as their default gateway.
- Published ports now bind to `127.0.0.1` by default through `LOCAL_BIND_ADDRESS`.
- n8n now explicitly disables diagnostics, version notifications, template fetching, and community package installs.
- The repo intentionally enables `Local File Trigger` for self-hosted local ingestion while still blocking `Execute Command` with `NODES_EXCLUDE=["n8n-nodes-base.executeCommand"]`.
- Qdrant telemetry is explicitly disabled with `QDRANT__TELEMETRY_DISABLED=true`.
- Docling remote services are explicitly disabled, and OTLP export remains unset.
- Automatic `ollama pull` services were removed. Models must be staged deliberately.
- The bundled demo workflow now targets `llama3.2` instead of the floating `llama3.2:latest` alias.
- The bundled static HTML page no longer loads CDN JavaScript, CDN CSS, or Google Fonts.
- The static file server no longer sends `Access-Control-Allow-Origin: *`.
- `.env.example` now uses placeholder secrets instead of weak demo credentials.
- Compose image references are now pinned to immutable digests where registry access allowed reliable verification.

## Network topology after the Docker Desktop for Mac fix

Why the old design was a problem:

- The earlier hardening pass put every service on one `internal: true` bridge network.
- That preserved no-egress intent, but it also made every published localhost port depend on the same isolated network path.
- On Docker Desktop for Mac, that topology proved unreliable for host port forwarding even when the containers were healthy and reachable from each other.

What the stack uses now:

- `backend`: `internal: true`, used for service-to-service traffic and kept as the default route with Compose `gw_priority`.
- `edge`: normal bridge, attached only to services that need host localhost access.
- Published ports remain explicitly bound to `127.0.0.1`, and the `edge` bridge also defaults host bindings to loopback as defense in depth.

Security impact:

- Containers still use the isolated `backend` network as their default gateway, so adding `edge` does not become a silent egress bypass.
- Services that do not need host access, such as `postgres` and `n8n-import`, stay on `backend` only.

## Container image digest pinning

Why this matters:

- Mutable tags such as `latest` and `main` can change underneath the same Compose file.
- `image:tag@sha256:...` keeps deployments reproducible and makes image refreshes an explicit operator action.

Pinned in `docker-compose.yml`:

- `n8nio/n8n:2.13.4@sha256:ab216dc8d10d1940a07f41f04355011ab03d0dec1fc03d62d5db3fc1bad815f5`
- `ollama/ollama:0.19.0-rc0@sha256:ff17815f73e64408bbd723bb8e570770dfb612238e59e4c9bc5832ec7928df82`
- `ollama/ollama:0.19.0-rc0-rocm@sha256:712e4b2fb26840eb4c5cca0cb0068f67b1952cfc756a2a4e3e990ed45efa1c84`
- `postgres:16.13-alpine3.23@sha256:20edbde7749f822887a1a022ad526fde0a47d6b2be9a8364433605cf65099416`
- `qdrant/qdrant:v1.17.1@sha256:94728574965d17c6485dd361aa3c0818b325b9016dac5ea6afec7b4b2700865f`
- `nginx:1.29.7-alpine@sha256:e7257f1ef28ba17cf7c248cb8ccf6f0c6e0228ab9c315c152f9c203cd34cf6d1`
- `ghcr.io/docling-project/docling-serve:main@sha256:602777945860ba49c27181b72c90294e7fc3fa4a7facb2165f50a8254422d6a6`
- `ghcr.io/docling-project/docling-serve-cu126:main@sha256:01521110609f51f20ba87b96464a49dfc797eddcb6423da92036aa10ec5b1873`

Still unpinned in this pass:

- `ghcr.io/docling-project/docling-serve-rocm:main`
  GHCR denied anonymous manifest access for that repository during this audit, so there was no reliable digest to record from this environment.

How to refresh digests intentionally:

1. Choose the exact upstream release tag you want to run.
2. Inspect its manifest digest with `docker buildx imagetools inspect IMAGE:TAG` or `docker manifest inspect IMAGE:TAG`.
3. Update `docker-compose.yml` to `IMAGE:TAG@sha256:...`.
4. Review upstream release notes before pulling the new image.

## Inventory of outbound-capable behavior found

### Setup-time external network use

- `git clone` of this repository.
- Container image pulls for:
  - `n8nio/n8n:2.13.4@sha256:ab216dc8d10d1940a07f41f04355011ab03d0dec1fc03d62d5db3fc1bad815f5`
  - `ollama/ollama:0.19.0-rc0@sha256:ff17815f73e64408bbd723bb8e570770dfb612238e59e4c9bc5832ec7928df82`
  - `ollama/ollama:0.19.0-rc0-rocm@sha256:712e4b2fb26840eb4c5cca0cb0068f67b1952cfc756a2a4e3e990ed45efa1c84`
  - `qdrant/qdrant:v1.17.1@sha256:94728574965d17c6485dd361aa3c0818b325b9016dac5ea6afec7b4b2700865f`
  - `postgres:16.13-alpine3.23@sha256:20edbde7749f822887a1a022ad526fde0a47d6b2be9a8364433605cf65099416`
  - `nginx:1.29.7-alpine@sha256:e7257f1ef28ba17cf7c248cb8ccf6f0c6e0228ab9c315c152f9c203cd34cf6d1`
  - `ghcr.io/docling-project/docling-serve:main@sha256:602777945860ba49c27181b72c90294e7fc3fa4a7facb2165f50a8254422d6a6`
  - `ghcr.io/docling-project/docling-serve-cu126:main@sha256:01521110609f51f20ba87b96464a49dfc797eddcb6423da92036aa10ec5b1873`
  - `ghcr.io/docling-project/docling-serve-rocm:main`
- Optional clone/build of the external Admo repository.
- Optional model download from Ollama if an operator intentionally relaxes the no-egress default or uses a separate machine to preload models.

### Runtime outbound-capable behavior found before hardening

- n8n telemetry, version notifications, and template fetching.
- n8n community package installation from npm.
- n8n's broad node surface, including explicit re-enablement of nodes that n8n excludes by default for safety (`Execute Command`, `Local File Trigger`).
- Docling runtime remote-service support was explicitly enabled in Compose.
- Qdrant usage statistics were not explicitly disabled.
- Ollama model pulling was automated at startup.
- The bundled static HTML page fetched JavaScript/CSS from jsDelivr and fonts from Google Fonts.
- Published container ports were exposed on all host interfaces by default.

### Runtime state after hardening

- Under the default Compose file, containers talk to each other on the isolated `backend` network while host-published services also join the `edge` network for localhost access.
- Host access to services is limited to loopback unless `LOCAL_BIND_ADDRESS` is changed.
- The stack still depends on operator discipline:
  - relaxing the `backend` network or changing the default gateway away from `backend` restores container egress
  - importing new workflows into n8n can reintroduce risky logic if the network boundary is relaxed
  - enabling the optional Admo profile adds an external codebase that is not covered by this repo audit

## Component review

### n8n

Local role:

- workflow orchestration
- local calls to PostgreSQL, Ollama, Qdrant, Docling, and optionally Admo

Outbound behavior to expect from upstream by default:

- diagnostics
- version notifications
- template fetching
- community package installation
- arbitrary external HTTP/API calls via workflows and many built-in nodes

Repo state after hardening:

- diagnostics/version/template lookups disabled
- community packages disabled
- `Execute Command` remains blocked
- `Local File Trigger` is intentionally enabled for bind-mounted local ingestion under `/data/shared/watched-files`
- container egress blocked by the Docker network

Assessment:

- Safer than before, but not intrinsically "safe" if an operator later relaxes network isolation, imports unreviewed workflows, or points Local File Trigger at untrusted local inputs.

### Qdrant

Local role:

- vector storage and retrieval

Outbound behavior to expect from upstream by default:

- anonymous usage statistics unless disabled

Repo state after hardening:

- telemetry disabled explicitly
- no external snapshot backends configured
- container egress blocked by the Docker network

Assessment:

- Safe for local-only runtime in the current hardened Compose configuration.

### Docling / Docling Serve

Local role:

- local OCR and document conversion

Outbound behavior to expect from upstream by default:

- optional remote-service calls for VLM/API-backed pipelines when enabled
- optional OTLP telemetry if configured
- upstream distributed image build downloads Docling model weights during image creation

Repo state after hardening:

- `DOCLING_SERVE_ENABLE_REMOTE_SERVICES=false`
- no OTLP endpoint configured
- container egress blocked by the Docker network

Assessment:

- Safe for local-only runtime in the current hardened Compose configuration.
- Not air-gapped at supply-chain/setup time unless you mirror the trusted image or build it yourself from audited source.

### PostgreSQL

Local role:

- n8n application database

Outbound behavior found in this repo:

- none configured

Repo state after hardening:

- not published to the host
- reachable only inside the Docker network

Assessment:

- Local-only in the current Compose configuration.

### Ollama

Local role:

- local model serving

Outbound behavior to expect from upstream by default:

- model pulls use the internet

Repo state after hardening:

- automatic pull containers removed
- container egress blocked by the Docker network

Assessment:

- Safe for local-only runtime only after models are already present.
- Not ready for use until you preload the required models.

### Static file server

Local role:

- serve extracted images or other local artifacts from `shared/extracted-images`

Outbound behavior found before hardening:

- default `chat.html` page loaded assets from public CDNs

Repo state after hardening:

- published only on loopback
- bundled page is self-contained and offline-safe

Assessment:

- Local-only in the current hardened configuration.

### Admo integration

Local role:

- optional ServiceNow XML microservice, built from a sibling repo

Repo-visible outbound behavior:

- integration variables suggest local-only intent (`ALLOW_NETWORK=false`, `RETRIEVAL_MODE=local_only`)

What remains unresolved:

- the service source is not present in this repository
- the linked upstream repository was not available for direct code audit during this pass

Assessment:

- Do not treat the `admo` profile as safe for sensitive data until the Admo codebase itself is audited and pinned.

## Runtime risk map

### Default hardened runtime

- `n8n` may call other containers on the isolated `backend` Docker network.
- Host-published services stay reachable on localhost through the separate `edge` bridge.
- `Local File Trigger` is available for self-hosted workflows and should watch `/data/shared/watched-files`.
- `ollama` serves only locally staged models.
- `docling-*` processes documents locally without remote services.
- `qdrant` stores vectors locally with telemetry disabled.
- `postgres` stays internal-only.
- `static-files` serves local files only on loopback.

### Setup-time or optional network use

- Docker image pulls
- Git clones
- Optional Admo clone/build
- Optional model staging from the public Ollama registry

### Data that could still leave the machine if protections are relaxed

- prompts, documents, OCR results, embeddings, and workflow payloads from n8n or Docling
- model names and model pull metadata from Ollama
- service metadata if Qdrant telemetry is re-enabled

## Sensitive-data readiness

### As-is before this hardening pass

- Not safe for sensitive data.

### After the hardening changes in this repo

- Conditionally safer, but still not ready for blanket approval without operator controls.

Remaining blockers:

- setup still depends on external registries unless you mirror or preload images
- Ollama models must be staged from a trusted source before use
- `ghcr.io/docling-project/docling-serve-rocm:main` remains unpinned because GHCR denied anonymous manifest inspection during this audit
- the optional Admo service is outside this repository and remains unaudited here
- any future override that removes `backend.internal`, changes the default route away from `backend`, or broadens port bindings reintroduces runtime egress

Recommended operator actions before trusting sensitive data:

1. Mirror the pinned images internally and do not use the `gpu-amd` profile for sensitive data until the Docling ROCm image is pinned or mirrored from a trusted source.
2. Stage Ollama models and any other required artifacts before sensitive documents enter the stack.
3. Keep `LOCAL_BIND_ADDRESS=127.0.0.1` and keep `backend` as the internal default-route network.
4. Do not enable the `admo` profile until that codebase is audited separately.
5. Review every n8n workflow import and keep the Docker egress boundary intact.

## Temporary online bootstrap flow

Use this only before sensitive data is present when you intentionally need setup-time outbound traffic such as:

- first-run n8n owner setup with email registration
- n8n license activation with `N8N_LICENSE_ACTIVATION_KEY`
- one-time Ollama model pulls or other audited setup downloads

Why this works:

- `docker-compose.online-bootstrap.yml` changes `backend.internal` from `true` to `false`
- host-facing services, including `n8n`, still keep `backend` as their default route
- that means bootstrap mode restores the outbound path n8n needs for license activation without changing the normal hardened runtime file

If your network uses TLS inspection or a private CA, place PEM files in `./pki/` before starting bootstrap mode. The n8n services mount that directory to `/opt/custom-certificates`, which is the path n8n uses for custom certificates.

This certificate-issuer problem was observed in practice on a corporate TLS-interception network. If your environment uses tooling such as zScaler, a temporary workaround is to disable that inspection layer briefly for the one-time registration or activation step. The more durable fix is to trust the corporate CA through `./pki/`.

Example n8n setup and activation flow:

```bash
cp .env.example .env
# Set secrets in .env, then optionally set N8N_LICENSE_ACTIVATION_KEY=...
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml up -d
# Complete owner setup in the UI, then verify the license:
docker compose exec -u node n8n n8n license:info
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml down
docker compose up -d
```

Example CPU bootstrap for setup-time model staging:

```bash
cp .env.example .env
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml --profile cpu up -d ollama-cpu
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml --profile cpu exec ollama-cpu ollama pull llama3.2
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml down
docker compose --profile cpu up -d
```

This temporarily disables the `backend` no-egress network guardrail. Do not use that override during sensitive-data processing.

## Upstream references

- n8n isolation guidance: <https://docs.n8n.io/hosting/configuration/configuration-examples/isolation/>
- n8n node environment variables: <https://docs.n8n.io/hosting/configuration/environment-variables/nodes/>
- n8n custom certificates: <https://docs.n8n.io/hosting/configuration/configuration-examples/custom-certificate-authority/>
- n8n CLI commands: <https://docs.n8n.io/hosting/cli-commands/>
- n8n community edition registration and activation: <https://docs.n8n.io/hosting/community-edition-features/>
- Qdrant usage statistics: <https://qdrant.tech/documentation/guides/usage-statistics/>
- Docling Serve configuration: <https://github.com/docling-project/docling-serve/blob/main/docs/configuration.md>
- Docling Serve image build source: <https://github.com/docling-project/docling-serve/blob/main/Containerfile>
- Ollama FAQ: <https://docs.ollama.com/faq>
