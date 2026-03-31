# Self-hosted AI starter kit (incl. Docling, optional Admo)

**Self-hosted AI Starter Kit** is an open-source Docker Compose template designed to swiftly initialize a comprehensive local AI and low-code development environment.

> [NOTE]
> This repository is a fork of [n8n-io/self-hosted-ai-starter-kit](https://github.com/n8n-io/self-hosted-ai-starter-kit).

![n8n.io - Screenshot](assets/n8n-demo.gif)

Curated by <https://github.com/n8n-io>, it combines the self-hosted n8n
platform with a curated list of compatible AI products and components to
quickly get started with building self-hosted AI workflows.

> [!TIP]
> [Read the announcement](https://blog.n8n.io/self-hosted-ai/)

### What’s included

✅ [**Self-hosted n8n**](https://n8n.io/) - Low-code platform with over 400
integrations and advanced AI components

✅ [**Ollama**](https://ollama.com/) - Cross-platform LLM platform to install
and run local LLMs

✅ [**Qdrant**](https://qdrant.tech/) - Open-source, high performance vector
store with an comprehensive API

✅ [**PostgreSQL**](https://www.postgresql.org/) -  Workhorse of the Data
Engineering world, handles large amounts of data safely.

✅ [**Docling**](https://www.docling.ai/) - OCR and document parsing service for extracting structured data from documents

✅ **Static File Server** - Nginx-based file server that serves the shared folder, accessible at http://localhost:8080

🧩 [**Admo**](https://github.com/nsolz7/Admo) - Optional local ServiceNow XML microservice that complements Docling instead of replacing it

### What you can build

⭐️ **AI Agents** for scheduling appointments

⭐️ **Summarize Company PDFs** in a local-first stack

⭐️ **Smarter Slack Bots** for enhanced company communications and IT operations

⭐️ **Local Financial Document Analysis** at minimal cost

## Security-first local-only mode

The default Compose configuration in this repo is now hardened for local-only operation:

- Published ports are bound to `127.0.0.1` by default
- Runtime traffic is split between an isolated `backend` network and a host-facing `edge` network
- The `backend` network is marked `internal`, which blocks container egress by default
- n8n diagnostics, version checks, template fetching, and community package installs are disabled
- Qdrant usage statistics are disabled
- Docling remote services are disabled
- Automatic Ollama model pulls have been removed
- Compose image references are pinned to immutable digests where registry access allowed reliable verification

This two-layer network model exists because Docker Desktop for Mac can behave poorly when published localhost ports depend on a single `internal` network. Host-facing services now join both networks: `edge` restores reliable `localhost` access, while `backend` stays the default gateway to preserve the local-only no-egress intent.

Read [LOCAL_ONLY_HARDENING.md](LOCAL_ONLY_HARDENING.md) before putting sensitive data in the stack. Setup still requires pulling container images, and models must be staged deliberately instead of being downloaded automatically at runtime.

## Image digest pinning

`docker-compose.yml` now pins reviewed image manifests with `image:tag@sha256:...` references so a mutable tag cannot silently retarget to different contents later.

Use [LOCAL_ONLY_HARDENING.md](LOCAL_ONLY_HARDENING.md) for the current pinned-image list, the one remaining unpinned image, and the refresh procedure for updating digests intentionally.

## Installation

### Cloning the Repository

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
```

### Running n8n using Docker Compose

#### For Nvidia GPU users

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile gpu-nvidia up
```

> [!NOTE]
> If you have not used your Nvidia GPU with Docker before, please follow the
> [Ollama Docker instructions](https://github.com/ollama/ollama/blob/main/docs/docker.md).

### For AMD GPU users on Linux

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile gpu-amd up
```

#### For Mac / Apple Silicon users

If you’re using a Mac with an M1 or newer processor, you can't expose your GPU
to the Docker instance, unfortunately. There are three supported paths here:

##### Option 1: Ollama installed on the Mac host

Use this when you want n8n in Docker, but Ollama running directly on macOS for
faster local inference and you do not need Docling.

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose up
```

This starts only the default, unprofiled services:

- `postgres`
- `n8n-import`
- `n8n`
- `qdrant`
- `static-files`

It does not start Docker Ollama or Docling. For this path:

1. Install Ollama on the Mac host using the [Ollama homepage](https://ollama.com/).
2. Set `OLLAMA_HOST=host.docker.internal:11434` in your `.env` file.
3. After n8n starts, open <http://localhost:5678/home/credentials>, select
   `Local Ollama service`, and change the base URL to
   `http://host.docker.internal:11434/`.

This path expects Ollama outside the Compose project. n8n reaches host Ollama
through `host.docker.internal:11434`. Docling-backed workflows will not work in
this mode because no Docling container is started.

##### Option 2: Ollama on the Mac host plus Docling in Docker

Use this when you want host-installed Ollama for inference speed, but still want
Docling available in Docker for document parsing workflows.

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile docling up
```

This starts the default services plus:

- `docling-cpu`

It does not start Docker Ollama. For this path:

1. Install Ollama on the Mac host using the [Ollama homepage](https://ollama.com/).
2. Set `OLLAMA_HOST=host.docker.internal:11434` in your `.env` file.
3. After n8n starts, open <http://localhost:5678/home/credentials>, select
   `Local Ollama service`, and change the base URL to
   `http://host.docker.internal:11434/`.

n8n reaches host Ollama through `host.docker.internal:11434`, and it reaches
Docling inside the Compose network at `http://docling:5001`.

##### Option 3: Full CPU stack inside Docker

Use this when you want both Ollama and Docling started inside Docker:

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile cpu up
```

This starts the default services plus:

- `ollama-cpu`
- `docling-cpu`

For this path, leave `OLLAMA_HOST` unset so n8n keeps the default in-network
endpoint `ollama:11434`. n8n reaches Docling inside the Compose network at
`http://docling:5001`. It does not use host-installed Ollama.

If you also use the optional Admo sibling service, add `--profile admo` to any
of these commands after cloning the Admo repository described later in this
README.

#### For everyone else

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile cpu up
```

### One-time bootstrap for owner setup, license activation, and model staging

Use `docker-compose.online-bootstrap.yml` only before sensitive data is present, when you intentionally need outbound traffic during setup. In this repo, that includes:

- first-run n8n setup if you want to use the community email-registration flow
- applying `N8N_LICENSE_ACTIVATION_KEY` against the n8n license server
- one-time Ollama model pulls or other audited setup downloads

Supported first-run flow:

1. Clone the repo and create `.env` from `.env.example`.
2. Replace the placeholder secrets and passwords in `.env`.
3. If you already have an n8n activation key, set `N8N_LICENSE_ACTIVATION_KEY` in `.env` before the first bootstrap start.
4. If your network uses TLS inspection or a private CA, place PEM files in `./pki/` before starting n8n in bootstrap mode.
5. Start the same stack you intend to use, but add the bootstrap override for this one-time setup phase:

```bash
# default stack
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml up -d

# docling stack
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml --profile docling up -d

# full CPU stack
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml --profile cpu up -d
```

Add `--profile admo` to the bootstrap command too if you intend to keep Admo enabled.

6. Open <http://localhost:5678/> and complete the one-time owner account setup.
7. Verify activation in **Settings > Usage and plan** or with:

```bash
docker compose exec -u node n8n n8n license:info
```

8. If `N8N_LICENSE_ACTIVATION_KEY` was already configured, you do not need the UI **Sign up** or **Send me a free license key** flow. That UI path is for email-based registered community edition. If the license is active in **Usage and plan** or `license:info`, ignore that sign-up prompt.
9. When setup is complete, stop the bootstrap stack and restart without the override so the hardened no-egress runtime is restored:

```bash
docker compose -f docker-compose.yml -f docker-compose.online-bootstrap.yml down
docker compose up -d
```

If you started with profiles, use the same profile set again on the final hardened restart. If activation fails with `unable to get local issuer certificate`, bootstrap mode is already providing egress; the missing piece is TLS trust, so add the required PEM certificate(s) under `./pki/` and restart the bootstrap run. This was observed in practice on a corporate TLS-inspection network. If your environment uses tooling such as zScaler, a temporary workaround is to disable that inspection layer briefly for the one-time registration or activation step; the more durable fix is to add the corporate CA to `./pki/`.

## ⚡️ Quick start and usage

The core of the Self-hosted AI Starter Kit is a Docker Compose file, pre-configured with network and storage settings, minimizing the need for additional installations.
After completing the installation steps above, simply follow the steps below to get started.

1. Open <http://localhost:5678/> in your browser to set up n8n. You’ll only
   have to do this once.
2. Open the included workflow:
   <http://localhost:5678/workflow/srOnR8PAY3u4RSwb>
3. Click the **Chat** button at the bottom of the canvas, to start running the workflow.
4. Stage the `llama3.2` model into Ollama before using the demo workflow. The
   hardened default no longer auto-downloads models. If you intentionally want
   a setup-time internet download, use the temporary online bootstrap flow
   documented in [LOCAL_ONLY_HARDENING.md](LOCAL_ONLY_HARDENING.md).
5. Any Docling-backed workflow requires a running Docling service. Start either
   the `docling` profile or the full `cpu` profile before using those flows.

To open n8n at any time, visit <http://localhost:5678/> in your browser.

When the corresponding services are running, these localhost URLs should work:

- n8n: <http://localhost:5678/>
- Docling UI: <http://localhost:5001/ui>
- Docling docs: <http://localhost:5001/docs>
- Admo health: <http://localhost:8000/api/v1/health>
- Qdrant dashboard: <http://localhost:6333/dashboard>
- Static files: <http://localhost:8080/>
- Docker-hosted Ollama: <http://localhost:11434/>

With your n8n instance, you’ll have access to over 400 integrations and a
suite of basic and advanced AI nodes such as
[AI Agent](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/),
[Text classifier](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.text-classifier/),
and [Information Extractor](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.information-extractor/)
nodes. n8n is still an automation platform with many nodes capable of outbound
HTTP or API access. In the hardened default, container egress is blocked at the
Docker network layer, but imported workflows and future config changes can
reintroduce outbound paths. Review every workflow before using sensitive data.

> [!NOTE]
> This starter kit is designed to help you get started with self-hosted AI
> workflows. While it’s not fully optimized for production environments, it
> combines robust components that work well together for proof-of-concept
> projects. You can customize it to meet your specific needs

## Optional Admo integration

This fork can also run [Admo](https://github.com/nsolz7/Admo) as a sibling local service for ServiceNow XML.

> [!WARNING]
> The `admo` profile builds code from a separate sibling repository outside this
> repo. This audit only covers the integration points in this repository. Treat
> Admo as a separate codebase that must be audited independently before handling
> sensitive data.

Keep the responsibilities split clean:

- `n8n` orchestrates workflow routing
- `Docling` handles non-ServiceNow documents such as PDFs, DOCX, PPTX, and images
- `Admo` handles ServiceNow XML exports only
- `Qdrant` remains the shared retrieval backend

### Setup

Clone Admo next to this repo and keep its Knowledge Project root outside both repositories:

```bash
git clone https://github.com/nsolz7/Admo.git ../Admo
cp .env.example .env
```

Then confirm these variables in `.env`:

```bash
ADMO_REPO_PATH=../Admo
ADMO_KNOWLEDGE_ROOT=../Admo-Knowledge-Project
ADMO_PORT=8000
ADMO_QDRANT_COLLECTION=admo_servicenow_xml
```

Start the stack with the `admo` profile in addition to your normal Docling/Ollama profile:

```bash
docker compose --profile cpu --profile admo up
```

Inside the backend Compose network, n8n can call Admo at `http://admo:8000`.

### Routing model

- Route ServiceNow XML exports to `POST /api/v1/imports/xml` on Admo.
- Route other local document types to Docling.
- Let Admo generate normalized XML-derived artifacts and staged Qdrant-ready payload files under its external Knowledge Project root.
- Let the larger n8n workflow handle any later embedding and Qdrant upsert steps.

Admo is intentionally not mounted into the starter kit `./shared` folder by default. Use the Admo API as the service boundary and keep Admo’s Knowledge Project root separate from the starter kit’s generic shared workspace.

## Upgrading

> [!WARNING]
> Every `docker compose pull` is a setup-time external network operation that
> fetches new container code. For sensitive deployments, pin image digests and
> mirror the images you trust instead of pulling floating tags directly.

* ### For Nvidia GPU setups:

```bash
docker compose --profile gpu-nvidia pull
docker compose create && docker compose --profile gpu-nvidia up
```

* ### For Mac / Apple Silicon users running Ollama on the host without Docling

```bash
docker compose pull
docker compose create && docker compose up
```

* ### For Mac / Apple Silicon users running Ollama on the host with Docker Docling

```bash
docker compose --profile docling pull
docker compose create && docker compose --profile docling up
```

* ### For Non-GPU setups:

```bash
docker compose --profile cpu pull
docker compose create && docker compose --profile cpu up
```

## 👓 Recommended reading

n8n is full of useful content for getting started quickly with its AI concepts
and nodes. If you run into an issue, go to [support](#support).

- [AI agents for developers: from theory to practice with n8n](https://blog.n8n.io/ai-agents/)
- [Tutorial: Build an AI workflow in n8n](https://docs.n8n.io/advanced-ai/intro-tutorial/)
- [Langchain Concepts in n8n](https://docs.n8n.io/advanced-ai/langchain/langchain-n8n/)
- [Demonstration of key differences between agents and chains](https://docs.n8n.io/advanced-ai/examples/agent-chain-comparison/)
- [What are vector databases?](https://docs.n8n.io/advanced-ai/examples/understand-vector-databases/)

## 🎥 Video walkthrough

- [Installing and using Local AI for n8n](https://www.youtube.com/watch?v=xz_X2N-hPg0)

## 🛍️ More AI templates

> [!WARNING]
> The n8n template gallery and the linked workflows are online resources and are
> not audited here for local-only behavior. Many templates call hosted APIs,
> websites, or cloud model providers. Do not import them into a sensitive-data
> environment without reviewing every node, credential, and outbound call.

For more AI workflow ideas, visit the [**official n8n AI template
gallery**](https://n8n.io/workflows/categories/ai/). From each workflow,
select the **Use workflow** button to automatically import the workflow into
your local n8n instance.

### Learn AI key concepts

- [AI Agent Chat](https://n8n.io/workflows/1954-ai-agent-chat/)
- [AI chat with any data source (using the n8n workflow too)](https://n8n.io/workflows/2026-ai-chat-with-any-data-source-using-the-n8n-workflow-tool/)
- [Chat with OpenAI Assistant (by adding a memory)](https://n8n.io/workflows/2098-chat-with-openai-assistant-by-adding-a-memory/)
- [Use an open-source LLM (via Hugging Face)](https://n8n.io/workflows/1980-use-an-open-source-llm-via-huggingface/)
- [Chat with PDF docs using AI (quoting sources)](https://n8n.io/workflows/2165-chat-with-pdf-docs-using-ai-quoting-sources/)
- [AI agent that can scrape webpages](https://n8n.io/workflows/2006-ai-agent-that-can-scrape-webpages/)

### Local AI templates

- [Tax Code Assistant](https://n8n.io/workflows/2341-build-a-tax-code-assistant-with-qdrant-mistralai-and-openai/)
- [Breakdown Documents into Study Notes with MistralAI and Qdrant](https://n8n.io/workflows/2339-breakdown-documents-into-study-notes-using-templating-mistralai-and-qdrant/)
- [Financial Documents Assistant using Qdrant and](https://n8n.io/workflows/2335-build-a-financial-documents-assistant-using-qdrant-and-mistralai/) [Mistral.ai](http://mistral.ai/)
- [Recipe Recommendations with Qdrant and Mistral](https://n8n.io/workflows/2333-recipe-recommendations-with-qdrant-and-mistral/)

## Tips & tricks

### Accessing local files

The self-hosted AI starter kit will create a shared folder (by default,
located in the same directory) which is mounted to the n8n container and
allows n8n to access files on disk. This folder within the n8n container is
located at `/data/shared` -- this is the path you’ll need to use in nodes that
interact with the local filesystem.

This repository intentionally enables **Local File Trigger** for self-hosted
local ingestion while still keeping **Execute Command** blocked through
`NODES_EXCLUDE=["n8n-nodes-base.executeCommand"]`.

Use this dedicated watch path for Local File Trigger workflows:

- Host path: `./shared/watched-files/`
- Container path: `/data/shared/watched-files/`

Because `./shared` is bind-mounted into the n8n container, files dropped into
`./shared/watched-files/` on the host are visible to Local File Trigger inside
the container at `/data/shared/watched-files/`.

Local File Trigger is a self-hosted capability. It only works when the watched
folder is bind-mounted into the n8n container, which this stack already does
through `./shared:/data/shared`.

On Docker Desktop for macOS, bind-mounted file events can be unreliable. If the
node doesn’t react to file changes consistently, use the Local File Trigger
node’s **Polling Mode** as the recommended fallback.

Use this intentionally:

- watch only trusted local folders
- keep the watched path inside `/data/shared`
- leave `Execute Command` blocked unless you perform a separate security review

**Nodes available for local filesystem access**

- [Local File Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.localfiletrigger/)
- [Read/Write Files from Disk](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.filesreadwrite/)

## 📜 License

This project is licensed under the Apache License 2.0 - see the
[LICENSE](LICENSE) file for details.

## 💬 Support

Join the conversation in the [n8n Forum](https://community.n8n.io/), where you
can:

- **Share Your Work**: Show off what you’ve built with n8n and inspire others
  in the community.
- **Ask Questions**: Whether you’re just getting started or you’re a seasoned
  pro, the community and our team are ready to support with any challenges.
- **Propose Ideas**: Have an idea for a feature or improvement? Let us know!
  We’re always eager to hear what you’d like to see next.
