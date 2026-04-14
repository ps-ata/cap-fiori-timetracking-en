# ADR 0021: Dev Containers and GitHub Codespaces Integration

## Status

**Accepted** – implemented as a complete development environment for cloud-native development

## Context and Problem Statement

The CAP Fiori Time Tracking application has complex development requirements:

- **Node.js 22.20.0** (specific version from `.nvmrc`)
- **Java 17** (Temurin) for `@sap/ams-dev` builds
- **SAP-specific tools** (`@sap/cds-dk`, `mbt`, Cloud Foundry CLI)
- **TypeScript tooling** with auto-generation of CDS types
- **Multiple UI5 apps** with workspace structure

**Challenges for new developers:**

1. ⏱️ **Time-to-first-commit**: 30-60 minutes setup time for local environment
2. 🔧 **Tool versions**: Different Node/Java versions on developer machines
3. 📦 **Dependency hell**: complex installation of `cf` CLI, `mbt`, MultiApps plugin
4. 🐛 **"Works on my machine"**: inconsistent development environments
5. 🆕 **Onboarding**: high entry barrier for new contributors
6. 🌍 **Remote work**: no cloud IDE option for remote development

**Affected stakeholders:**

- new contributors (highest priority)
- external developers without SAP tooling experience
- teams with heterogeneous development environments
- remote teams without access to powerful local hardware

## Decision Factors

1. **Developer Experience (DX)**
   - onboarding time: < 5 minutes to "hello world"
   - zero-config: no manual installation steps
   - consistency: identical environment for all developers

2. **Tool completeness**
   - all SAP CAP dependencies
   - Cloud Foundry CLI + MultiApps plugin
   - VS Code extensions for optimal IDE experience

3. **Maintainability**
   - declarative configuration (Infrastructure as Code)
   - version control for dev environment
   - easy updates when dependencies change

4. **Performance**
   - fast container builds (< 5 minutes)
   - caching of node modules
   - port forwarding for local testing

5. **Cost efficiency**
   - GitHub Codespaces: 60 hours/month free (2-core)
   - VS Code Dev Containers: completely free (local)

6. **Security & compliance**
   - secrets management via GitHub Codespaces Secrets
   - no credentials in container images
   - isolated development environments

## Considered Options

### Option A – comprehensive documentation (status quo)

**Description**: manual installation of all tools according to `GETTING_STARTED.md`

**Pros**:

- ✅ no additional infrastructure
- ✅ full control over local environment
- ✅ already documented

**Cons**:

- ❌ 30-60 minutes setup time
- ❌ platform-specific issues (Windows/macOS/Linux)
- ❌ version conflicts (Node 20 vs. 22, Java 11 vs. 17)
- ❌ "works on my machine" syndrome
- ❌ high entry barrier for contributors

### Option B – Docker Compose

**Description**: `docker-compose.yml` for complete stack (CAP + DB + tools)

**Pros**:

- ✅ reproducible environment
- ✅ production-like setup
- ✅ multi-container for services

**Cons**:

- ❌ more complex orchestration
- ❌ no IDE integration (VS Code extensions missing)
- ❌ no hot-reload for code changes
- ❌ networking issues with port forwarding
- ❌ overhead for simple development tasks

### Option C – GitHub Codespaces + VS Code Dev Containers (chosen)

**Description**: `.devcontainer/devcontainer.json` with:

- base image: `mcr.microsoft.com/devcontainers/typescript-node:22-bookworm`
- features: Node 22.20.0, Java 17 (Temurin)
- tools: `cds-dk`, `mbt`, `cf` CLI, TypeScript, Prettier
- VS Code extensions: CDS, ESLint, Prettier, REST Client
- automatic setup via `setup.sh` (postCreateCommand)

**Pros**:

- ✅ **zero-config**: 1-click start in Codespaces
- ✅ **consistency**: identical environment for all developers
- ✅ **IDE integration**: native VS Code extensions
- ✅ **performance**: caching + pre-builds possible
- ✅ **dual-use**: local (Dev Containers) + cloud (Codespaces)
- ✅ **hot-reload**: `npm run watch` works out-of-the-box
- ✅ **port forwarding**: automatic HTTPS URLs for testing
- ✅ **secrets**: GitHub Codespaces Secrets for CF credentials

**Cons**:

- ⚠️ Codespaces: 60 hrs/month free, then paid
- ⚠️ requires Docker Desktop for local Dev Containers
- ⚠️ container build: ~3-5 minutes on first start

### Option D – GitPod

**Description**: alternative cloud IDE with `.gitpod.yml`

**Pros**:

- ✅ similar features to Codespaces
- ✅ 50 hours/month free

**Cons**:

- ❌ less GitHub integration
- ❌ separate platform vendor lock-in
- ❌ no native VS Code Dev Containers support

## Decision

**We choose Option C: GitHub Codespaces + VS Code Dev Containers**

### Rationale

1. **developer experience first**: new contributors can be productive within **3-5 minutes**
2. **best of both worlds**: local development (Dev Containers) + cloud IDE (Codespaces)
3. **GitHub-native**: perfect integration with our git workflow
4. **industry standard**: devcontainer spec is open and supported by Microsoft, GitHub, GitLab
5. **maintainability**: declarative configuration in `devcontainer.json` (IaC principle)
6. **cost-effective**: 60 hrs/month Codespaces free sufficient for most contributors

### Implementation

**directory structure**:

```
.devcontainer/
├── devcontainer.json    # main configuration
├── setup.sh             # automatic setup script
└── README.md            # dev container documentation
```

**core components**:

1. **base image**: `typescript-node:22-bookworm` with Node 22.20.0
2. **features**:
   - `java:1` (version 17, Temurin distribution)
   - `node:1` (version 22.20.0)
   - `git:1` (latest)
   - `github-cli:1` (latest)

3. **post-create setup** (`setup.sh`):

   ```bash
   - npm install -g @sap/cds-dk typescript tsx mbt prettier @cap-js/mcp-server
   - cf CLI v8 installation + MultiApps plugin
   - npm ci (project dependencies)
   - .env creation (from .env.example)
   - cds-typer type generation
   ```

4. **port forwarding**:
   - `4004`: CAP server (auto-notify)
   - `8080`: UI testing (silent)

5. **VS Code extensions** (auto-install):
   - `SAPSE.vscode-cds` - SAP CDS language support
   - `dbaeumer.vscode-eslint` - ESLint
   - `esbenp.prettier-vscode` - Prettier
   - `humao.rest-client` - REST client
   - `eamodio.gitlens` - GitLens
   - `ms-azuretools.vscode-docker` - Docker

6. **editor settings**:
   - format on save: enabled
   - default formatter: Prettier
   - ESLint auto-fix: on save
   - line endings: LF (Unix-style)

## Consequences

### Positive effects

1. ✅ **onboarding**: time from 60min → **3-5min** for new contributors
2. ✅ **consistency**: "works on my machine" problems eliminated
3. ✅ **documentation**: setup process is code (IaC)
4. ✅ **remote-first**: teams can develop completely remote
5. ✅ **CI/CD alignment**: container setup similar to GitHub Actions runner
6. ✅ **accessibility**: lower hardware requirements (Codespaces in cloud)

### Negative effects / risks

1. ⚠️ **container overhead**: 3-5min on first start
   - **mitigation**: activate pre-builds in Codespaces (repository settings)

2. ⚠️ **Codespaces limits**: 60 hrs/month free
   - **mitigation**: local Dev Containers as fallback
   - **mitigation**: Codespaces auto-stop after 30min inactivity

3. ⚠️ **maintenance effort**: maintain dev containers for major updates
   - **mitigation**: versioning in `devcontainer.json` (Node, Java)
   - **mitigation**: test setup script on dependency updates

4. ⚠️ **Docker requirement**: local Dev Containers require Docker Desktop
   - **mitigation**: keep full docs in `GETTING_STARTED.md`

5. ⚠️ **network dependencies**: setup requires internet access
   - **mitigation**: caching npm packages (`node_modules` volume)

### follow-up tasks

1. **documentation**:
   - [x] `.devcontainer/README.md` with troubleshooting
   - [x] `README.md` update with Codespaces badge
   - [x] `GETTING_STARTED.md` with Codespaces guide
   - [x] `ARCHITECTURE.md` update (ch. 7: distribution view)

2. **optimization**:
   - [ ] activate pre-builds in repository settings (after merge)
   - [ ] `.dockerignore` for faster builds
   - [ ] caching strategy for npm dependencies

3. **testing**:
   - [ ] create fresh Codespace and test all workflows
   - [ ] test local Dev Containers with Docker Desktop
   - [ ] verify `npm run watch` → `npm test` → `npm run build:mta` flow

4. **governance**:
   - [ ] document dev container updates in CONTRIBUTING.md
   - [ ] add ADR references in relevant docs

## References

### Project files

- `.devcontainer/devcontainer.json` - main configuration
- `.devcontainer/setup.sh` - setup script
- `.devcontainer/README.md` - dev container documentation
- `.nvmrc` - Node version (source for container)
- `package.json` → `engines` - tool version requirements

### External documentation

- [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)
- [GitHub Codespaces Docs](https://docs.github.com/en/codespaces)
- [Dev Container Specification](https://containers.dev/)
- [Dev Container Features](https://containers.dev/features)

### Related ADRs

- [ADR-0004: TypeScript Tooling and Workflow](0004-typescript-tooling-und-workflow.md)
- [ADR-0016: Repository Meta Files and Governance](0016-repository-meta-dateien-und-governance.md)
- [ADR-0018: MTA Deployment Cloud Foundry](0018-mta-deployment-cloud-foundry.md)

### GitHub Issues/PRs

- initial implementation: PR #[TBD]
- pre-builds activation: Issue #[TBD]
