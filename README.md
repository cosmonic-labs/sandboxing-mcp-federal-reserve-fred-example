```markdown
# Federal Reserve MCP Server Setup Guide

## 🚀 The Story in Three Parts

### 🏦 1. Build an MCP Server in 60 Seconds (for the Federal Reserve)
Start by spinning up a fully working **Model Context Protocol (MCP)** server in under a minute.  
No fluff — just clean, fast setup that shows how quickly you can move from idea to running service.

### ⚙️ 2. Configure GitHub Actions Toolkit
Next, wire up **GitHub Actions** for **CI/CD**.  
This is where platform engineers shine — automating builds, managing releases, running tests, and keeping everything repeatable and reliable.

### ☁️ 3. Deploy on Cosmonic Control (on K8s) and Configure in Claude
Finally, take it live.  
Deploy the service to **Cosmonic Control** on Kubernetes, connect it to **Claude**, and configure your workflow for smart, autonomous operations across your platform.



## 🧩 Requirements

## 💻 Software Setup
- **[node](https://nodejs.org/)** – NodeJS runtime  
- **[npm](https://github.com/npm/cli)** – Node Package Manager (NPM) manages packages for the NodeJS ecosystem  
- **[wash](https://github.com/wasmCloud/wash)** – Wasm Shell (for developing and building Wasm components) with the `openapi2mcp` plugin  

### Platform Setup: ☁️ Setup Cosmonic Control on Kubernetes

Follow the installation guide to set up **Cosmonic Control** on your Kubernetes cluster:  
👉 [Install Cosmonic Control](https://docs.cosmonic.com/install-cosmonic-control)


## ⚡️ Build Our Server in 60 Seconds

node - NodeJS runtime
npm - Node Package Manager (NPM) manages packages for the NodeJS ecosystem
wash - Wasm Shell (for developing and building Wasm components) with the openapi2mcp plugin


## ⚡️ Build Our Server in 60 Seconds

```bash
wash new --git https://github.com/cosmonic-labs/mcp-server-template-ts.git "fed"
cd fed-fred-mcp
cp ../fred-schema.yaml ./  # or curl it down
```

## 📥 Download the OpenAPI Spec for the Federal Reserve

```bash
wget https://raw.githubusercontent.com/cosmonic-labs/sandboxing-mcp-federal-reserve-fred-example/refs/heads/main/fred-schema.yaml
```

## 🧠 Generate the Code

```bash
wash openapi2mcp ../fred-schema.yaml
```

## 🧑‍💻 Develop Locally

```bash
wash dev
```

Use the MCP Inspector to query and verify your load 🔍

## ⚙️ Leverage GitHub Actions Toolkit

### 🏗️ Create and Push Repo

```bash
git init
git add .
git commit -m "Initial MCP Server for Federal Reserve"
git remote add origin git@github.com:LiamRandall/fed-fred-mcp.git
git push -u origin main
```

### 🧩 Enable GitHub Actions for Pull Requests

In your GitHub repo:

**Settings → Actions → General → Allow GitHub Actions to create and approve pull requests**

🔗 Example: https://github.com/LiamRandall/fed-fred-mcp/settings/actions

### 🚦 Run Workflow

### 🔀 Merge Pull Request

🎃 You'll end up with a package created for Pumpkin Spice ☕

## 🚀 Deploy a Custom OCI Image with the Cosmonic HTTP Trigger Helm Chart

This shows how to deploy your own WebAssembly component image using the http-trigger Helm chart.

### ✅ Working Commands

#### Option A — Inline with Indexed Keys (Recommended for Bash)

For `bash` use:

```bash
helm install fed-fred-mcp2 oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  --set components[0].name="fed-fred-mcp" \
  --set components[0].image="ghcr.io/liamrandall/fed-fred-mcp:v0.1.0" \
  --set ingress.host="fred.localhost.cosmonic.sh" \
  --set pathNote="/v1/mcp"
```

For `zsh` use:

```bash
helm install fed oci://ghcr.io/cosmonic-labs/charts/http-trigger \
  --version 0.1.2 \
  --set 'components[0].name=fed' \
  --set 'components[0].image=ghcr.io/liamrandall/fed-fred-mcp:v0.1.0' \
  --set ingress.host=fred.localhost.cosmonic.sh \
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
  - name: fed-fred-mcp
    image: ghcr.io/liamrandall/fed-fred-mcp:v0.1.0

ingress:
  host: fred.localhost.cosmonic.sh
  pathNote: "/v1/mcp"
```

**Deploy with:**

```bash
helm install fed-fred-mcp2 oci://ghcr.io/cosmonic-labs/charts/http-trigger \
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
helm get values fed-fred-mcp2 -n default
helm get manifest fed-fred-mcp2 -n default | less
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