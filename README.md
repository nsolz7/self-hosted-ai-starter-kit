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
- The shared Docker network is marked `internal`, which blocks container egress
- n8n diagnostics, version checks, template fetching, and community package installs are disabled
- Qdrant usage statistics are disabled
- Docling remote services are disabled
- Automatic Ollama model pulls have been removed
- Compose image references are pinned to immutable digests where registry access allowed reliable verification

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
to the Docker instance, unfortunately. There are two options in this case:

1. Run the starter kit fully on CPU, like in the section "For everyone else"
   below
2. Run Ollama on your Mac for faster inference, and connect to that from the
   n8n instance

If you want to run Ollama on your mac, check the
[Ollama homepage](https://ollama.com/)
for installation instructions, and run the starter kit as follows:

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile cpu up
```

> [!NOTE]
> The `--profile cpu` flag is required to start Docling. Without it, only the core services (n8n, PostgreSQL, Qdrant) will start.

##### For Mac users running OLLAMA locally

If you're running Ollama locally on your Mac (not in Docker), you need to modify the `OLLAMA_HOST` environment variable. This path is not compatible with the hardened no-egress default unless you intentionally relax the Docker network isolation for setup or development.

1. Set OLLAMA_HOST to `host.docker.internal:11434` in your .env file. 
2. Additionally, after you see "Editor is now accessible via: <http://localhost:5678/>":

    1. Head to <http://localhost:5678/home/credentials>
    2. Click on "Local Ollama service"
    3. Change the base URL to "http://host.docker.internal:11434/"

#### For everyone else

```bash
git clone https://github.com/nsolz7/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env # replace all placeholder secrets before first run
docker compose --profile cpu up
```

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

To open n8n at any time, visit <http://localhost:5678/> in your browser.

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

Inside the compose network, n8n can call Admo at `http://admo:8000`.

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

* ### For Mac / Apple Silicon users

```bash
docker compose --profile cpu pull
docker compose create && docker compose --profile cpu up
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

By default, this Compose file keeps n8n’s safer node exclusions in place, so
`Local File Trigger` and `Execute Command` are not loaded. If you re-enable
them, treat that as a separate security review.

**Node that remains available for local filesystem access**

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
