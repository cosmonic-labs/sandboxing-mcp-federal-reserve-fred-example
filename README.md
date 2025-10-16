# Federal Reserve FRED MCP Server Building Guide

## 🚀 The Story in Three Parts

### 🏦 1. Build an MCP Server in 60 Seconds (for the Federal Reserve FRED API)
Instantly build a fully working **Model Context Protocol (MCP)** server in under a minute with Contract-driven generation with OpenAPI Specifications.

We are targeting [FRED](https://fred.stlouisfed.org/), from the Federal Reserve Bank of St. Louis.

### ⚙️ 2. Configure GitHub Actions Toolkit
Next, we'll walk through the included  **GitHub Actions** for **CI/CD**; we'll setup our first release and ship our new WebAssembly Sandboxed MCP Server to the `ghcr` OCI registry 

Automate builds, tests, and releases with GitHub Actions for seamless CI/CD — keeping everything repeatable, reliable, and hands-off.

### ☁️ 3. Deploy on Cosmonic Control (on K8s) and Configure in Claude
Finally, take it live.  
Easily deploy our tiny and fast WebAssembly Component to **Cosmonic Control** on Kubernetes, connect it to **Claude**, and start generating reports.


## 🧩 Requirements

## 💻 Software Setup
- **[node](https://nodejs.org/)** – NodeJS runtime  
- **[npm](https://github.com/npm/cli)** – Node Package Manager (NPM) manages packages for the NodeJS ecosystem  
- **[wash](https://github.com/wasmCloud/wash)** – Wasm Shell (for developing and building Wasm components) with the `openapi2mcp` plugin  

### Platform Setup: ☁️ Setup Cosmonic Control on Kubernetes

Follow the installation guide to set up **Cosmonic Control** on your Kubernetes cluster:  
👉 [Install Cosmonic Control](https://docs.cosmonic.com/install-cosmonic-control)


### ⚡️ Build Our Server in 60 Seconds

Let's start by downloading a pre-configured WebAssembly MCP Server Template: 
```bash
wash new --git https://github.com/cosmonic-labs/mcp-server-template-ts.git "fred-mcp"
cd fred-mcp
```

### 📥 Download the OpenAPI Spec for the Federal Reserve

`openapi2mcp` leverages contract driven development - so let's grab an OpenAPI Specification for the Federal Reserve Fred API:

```bash
wget https://raw.githubusercontent.com/cosmonic-labs/sandboxing-mcp-federal-reserve-fred-example/refs/heads/main/fred-spec.yaml
```

### 🧠 Generate the Code

In our template directory, let's go ahead and generate the API's; there is a 1:1 mapping in between OpenAPI Endpoints and MCP Tools.

```bash
wash openapi2mcp ../fred-schema.yaml
```

### 🧑‍💻 Develop Locally
Then, let's launch the developer loop- `Wash` is smart enough that it looks at the underlying programming language and knows how to do two things:

1. Launch the idiomatic build tools, so you can continue to work on your project in your code editor of choice.
2. Launch `pre` and `post` commit hooks to customize the functionality or experience of the project.  

In this case we are launching an MCP Inspector for us to diagnose our loop locally.

```bash
wash dev
```

### 🔍 Inspect your Secure WebAssembly MCP Server

Use the MCP Inspector to query and verify your load 🔍

### 💡 Optional
You can keep developing locally with `wash dev` — just expose your server to external LLM hosts using **[ngrok](https://ngrok.com/)**:  

```bash
ngrok http http://localhost:8000
```

## ⚙️ Configure Included GitHub Actions Toolkit

The Cosmonic MCP Server Template comes pre-configured with a **GitHub Actions Toolkit** for **CI/CD**; let's push our project to github, build a release, and ship our new WebAssembly Sandboxed MCP Server to the `ghcr` OCI registry  

### 🏗️ Create and Push Repo

Let's start by setting up a new repo for our code - our Github Actions will need the rights to create and approve pull requests. Most enterprise or team accounts will already have that enabled.

In this case, I'll use my personal account (you should use yours); then let's configure:

```bash
git init
git add .
git commit -m "Initial MCP Server for Federal Reserve"
git remote add origin git@github.com:LiamRandall/fred-mcp.git
git push -u origin main
```

### 🧩 Enable GitHub Actions for Pull Requests

In your GitHub repo:

**Settings → Actions → General → Allow GitHub Actions to create and approve pull requests**

🔗 Example: [https://github.com/LiamRandall/fred-mcp/settings/actions](https://github.com/LiamRandall/fred-mcp/settings/actions)

### 🚦 Run Workflow

Once we permissions, let's navigate to Actions - and generate a kick off our workflow: `Generate MCP from OpenAPI`

### 🔀 Merge Pull Request

Refresh your coffee ☕️ and 🤯 - you've got a pull request to approve!  Let's review, approve, and merge it in!

### 📦 Build a Release and Ship it!

Let's head back to the main page where we can create a release - make sure to create a new tag, I like to generate build notes, and ship it!


🎃 You'll end up with a package created for Pumpkin Spice ☕

Now that was fast, but we have a solid foundation; we have now have our MCP Server setup, we can build releases, and in just a few minutes we already have secure MCP Servers ready to deploy.  At Cosmonic we want to optimize for a great developer experience that embraces the secure principles by design.

## 🚀 Deploy a Custom OCI Image with the Cosmonic HTTP Trigger Helm Chart

Once our WebAssembly MCP Server in available in at `ghcr` let's go ahead and deploy it onto k8s with Cosmonic Control.

### ✅ Working Commands

#### Option A — Inline with Indexed Keys (Recommended for Bash)

For `bash` use:

```bash
helm install fred-mcp oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  --set components[0].name="fred-mcp" \
  --set components[0].image="ghcr.io/liamrandall/fred-mcp:v0.1.0" \
  --set ingress.host="fred-mcp.localhost.cosmonic.sh" \
  --set pathNote="/v1/mcp"
```

For `zsh` use:

```bash
helm install fred-mcp oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  --set 'components[0].name=fred-mcp' \
  --set 'components[0].image=ghcr.io/liamrandall/fred-mcp:v0.1.0' \
  --set ingress.host=fred-mcp.localhost.cosmonic.sh \
  --set pathNote=/v1/mcp
  ```

💡 **Tip:** Helm arrays must be indexed (components[0]). Otherwise, Helm treats components as an object and fails validation ❌

#### Option B — Inline with --set-json (Safe for zsh & CI)

```bash
helm install fed oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  --set-json 'components=[{"name":"fed","ghcr.io/liamrandall/fed:v0.1.0"}]' \
  --set ingress.host="fred.localhost.cosmonic.sh" \
  --set pathNote="/v1/mcp"
```



✅ `--set-json` avoids quoting issues in zsh and CI pipelines.

#### Option C — Using a Custom Values File (Most Readable)

**values.fred.yaml:**

```yaml
components:
  - name: fred-mcp
    image: ghcr.io/liamrandall/fred-mcp:v0.1.0

ingress:
  host: fred-mcp.localhost.cosmonic.sh
  pathNote: "/v1/mcp"
```

**Deploy with:**

```bash
helm install fed-fred-mcp oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  -f values.fred.yaml
```

## 🔍 Verify Deployment

### Preview Manifests

```bash
helm template preview oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  -f values.fred.yaml | less
```

### Inspect a Running or Failed Release

```bash
helm get values fred-mcp -n default
helm get manifest fred-mcp -n default | less
```

## ⚙️ Common Fields

| Field | Type | Description |
|-------|------|-------------|
| components | array | List of WebAssembly components to deploy |
| components[].name | string | Logical component name |
| components[].image | string | OCI image for the component |
| ingress.host | string | Hostname for ingress routing |
| pathNote | string | Optional path shown in Helm notes |
| replicas | int | Number of replicas (default 1) |

## 🧩 Troubleshooting

If you see this:

```
Invalid value: "object": spec.template.spec.components in body must be of type array
```

👉 It means `components` wasn't declared as a list. Always use `components[0].field`, `--set-json`, or a YAML file ✅
```
