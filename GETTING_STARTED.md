# 🚀 Getting Started - CAPture Time

Welcome! This guide will help you get the Time Tracking App up and running quickly.

---

## 🎯 Choose Your Development Environment

### ⚡ Option 1: GitHub Codespaces (Recommended for quick start)

**Zero-Config Development in the Cloud - Perfect for:**
- New contributors without local setup
- Quick prototyping & testing
- Teams with heterogeneous development environments
- Remote work without powerful hardware

**Get started in 3 steps:**

1. **Click the badge:**

   [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=main&repo=nimble-123/cap-fiori-timetracking)

2. **Wait for setup** (~3-5 minutes on first start)
   - Container is automatically built
   - All tools are installed (Node, Java, SAP Tools)
   - Dependencies are installed
   - TypeScript types are generated

3. **Start Development Server:**
   ```bash
   npm run watch
   ```

4. **Access the app:**
   - VS Code automatically shows port forward notification
   - Click "Open in Browser" or navigate to the URL
   - Format: `https://[codespace-name]-4004.app.github.dev`

**Features:**
- ✅ All tools pre-installed (Node 22, Java 17, cds-dk, mbt, cf CLI)
- ✅ VS Code extensions automatically activated
- ✅ Port forwarding with HTTPS URLs
- ✅ 60 hours/month free (2-core machine)
- ✅ Secrets management for CF credentials

**More info:**
- 📖 [Devcontainer README](.devcontainer/README.md)
- 📋 [ADR-0021: Devcontainer & Codespaces](docs/ADR/0021-devcontainer-github-codespaces.md)

---

### 💻 Option 2: VS Code Dev Containers (Local with Docker)

**Container-based development on your machine - Perfect for:**
- Developers with Docker Desktop
- Offline development
- Full control over resources
- No Codespaces limit

**Prerequisites:**
- Docker Desktop installed and running
- VS Code with "Dev Containers" extension

**Get started in 4 steps:**

1. **Clone repository:**
   ```bash
   git clone https://github.com/nimble-123/cap-fiori-timetracking.git
   cd cap-fiori-timetracking
   ```

2. **Open in Container:**
   - Open project in VS Code
   - `F1` → "Dev Containers: Reopen in Container"
   - Wait for setup (~3-5 minutes)

3. **Start Development:**
   ```bash
   npm run watch
   ```

4. **Access App:** `http://localhost:4004`

---

### 🛠️ Option 3: Manual Local Installation (Classic)

**Traditional local development - Perfect for:**
- Maximum flexibility
- No Docker dependency
- Developers with existing tool landscape

---

## 📋 Prerequisites (for Option 3: Manual Installation)

Make sure the following software is installed:

### Required

| Tool           | Version                                 | Download                            | Purpose                  |
| -------------- | --------------------------------------- | ----------------------------------- | ------------------------ |
| **Node.js**    | ≥22.x (per `.nvmrc` 22.20.0)           | [nodejs.org](https://nodejs.org/)   | Runtime for CAP & UI5    |
| **npm**        | ≥10.x                                   | (comes with Node.js)                | Package Manager          |
| **Java (JDK)** | ≥17 (Temurin recommended)              | [Adoptium](https://adoptium.net/)   | Build of `@sap/ams-dev`  |
| **TypeScript** | ≥5.0                                    | `npm install -g typescript`         | Compiler                 |
| **Git**        | Latest                                  | [git-scm.com](https://git-scm.com/) | Version Control          |

> Tip: If you use `nvm`, you can use `nvm use` to automatically activate the Node version defined in `.nvmrc` (22.20.0). If needed, `nvm install` will install the version once. For the Java requirement, Temurin 17 (Adoptium) is recommended; GitHub Actions sets up the same version via `actions/setup-java`.

#### Additional Tools for SAP BTP Deployments

| Tool                         | Version | Installation Step                                                             | Purpose                              |
| ---------------------------- | ------- | ----------------------------------------------------------------------------- | ------------------------------------ |
| **Cloud Foundry CLI (`cf`)** | ≥8.8    | [docs.cloudfoundry.org](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html) | Deployments, Service Management    |
| **CF MultiApps Plugin**      | Latest  | `cf install-plugin multiapps`                                                 | `cf deploy` for MTA support        |
| **MBT (`mbt`)**              | Latest  | `npm install -g mbt`                                                          | Builds `.mtar` for Multi-Target Apps |

### Recommended

| Tool                         | Version | Download                                                | Purpose             |
| ---------------------------- | ------- | ------------------------------------------------------- | ------------------- |
| **VS Code**                  | Latest  | [code.visualstudio.com](https://code.visualstudio.com/) | IDE                 |
| **SAP CDS Language Support** | Latest  | VS Code Extension                                       | CDS Syntax Highlighting |
| **ESLint**                   | Latest  | VS Code Extension                                       | Linting             |
| **Prettier**                 | Latest  | VS Code Extension                                       | Code Formatting     |

### Verify Installation

```bash
node --version    # should be v22.x.x
npm --version     # should be 10.x.x or higher
tsc --version     # should be version 5.x.x or higher
git --version     # should be installed
cf --version      # should be installed
mbt --version     # should be installed
```

---

## 📦 Installation

### 1. Clone repository

```bash
git clone https://github.com/nimble-123/cap-fiori-timetracking.git
cd cap-fiori-timetracking
```

### 2. Install dependencies

```bash
npm install
```

This installs:

- **CAP Framework** (`@sap/cds`)
- **TypeScript** tooling (`typescript`, `@cap-js/cds-typer`)
- **UI5 Tooling** (`@sap/ux-ui5-tooling`, `cds-plugin-ui5`)
- **Dev Tools** (`prettier`, `eslint`, `@sap/dev-cap-tools`)
- All frontend app dependencies (via workspaces)

**Note:** The project uses npm workspaces – `app/timetable` and `app/timetracking` are automatically linked.

> Thanks to `.npmrc`, installation fails if your Node version doesn't match the defined engines (`engine-strict=true`) – this keeps all environments consistent. `npm audit` is active by default.

### 3. Configure environment

Copy the example and adjust values if needed (default values are sufficient for local development):

```bash
cp .env.example .env
```

**Important variables:**

- `NODE_ENV`, `CDS_LOG_LEVELS_TRACK_SERVICE`, `CDS_LOG_FORMAT` – control logging and runtime behavior.
- `HOLIDAY_API_BASE_URL`, `HOLIDAY_API_TIMEOUT_MS` – configuration for the holiday API.
- `CAP_AUTH_STRATEGY` – sets the local authentication strategy (default: `mocked`).

All variables are optional. Unset values fall back to defaults from `Customizing` or services.

### 4. TypeScript types & entry points

- **Types:** `@cap-js/cds-typer` runs automatically (via `npm run watch`, `npm run build` or `cds-typer --watch`) and updates `@cds-models/*` on every `.cds` change – no manual command required.
- **Optional Entry Points:** If you use `dev-cap-tools` scripts (e.g., for CLI calls), you can optionally run `npm run generate-entry-point` to generate updated entry points. This doesn't affect the generated types.

---

## 🧭 Repository Guidelines & Meta Files

To ensure all contributors follow the same standards, the project includes several meta files:

- `.nvmrc` – defines Node.js 22.20.0. `nvm use` ensures you use exactly this version.
- `.npmrc` – enforces compatible Node versions (`engine-strict=true`) and activates security checks (`audit=true`).
- `.editorconfig` – sets formatting rules (2 spaces, LF, no trailing spaces), consistent with Prettier.
- `.env.example` – example configuration for local environment variables. Copy it to `.env` as described above.
- `@cap-js/console` – activates the CAP Console plugin for monitoring & log level switches within the [SAP CAP Console](https://cap.cloud.sap/docs/tools/console) (desktop app).
- `CODE_OF_CONDUCT.md` & `SECURITY.md` – describe code of conduct rules and security reporting procedures.
- `.github/ISSUE_TEMPLATE/*`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/dependabot.yml`, `.github/CODEOWNERS` – ensure clean issues/PRs, automatic dependency updates, and clearly assigned reviews.
- `release-please-config.json` & `.release-please-manifest.json` – control automated release PR creation; before production runs, a local dry-run is recommended (`npx release-please release-pr --config-file release-please-config.json --manifest-file .release-please-manifest.json --dry-run`, see README for details). UI5 app version numbers are pulled along via `extra-files`.

Please follow these guidelines before creating a PR.

---

## 🤖 AI Prompts & LLM Workflows

The repository contains curated prompts for GitHub Models & CoPilot (`.github/prompts/*.prompt.yml`) to support Product Owners, Developers, and QA with CAP-specific tasks.

### Usage

1. **Select prompt:** Read the YAML file contents (e.g., `product-owner-feature-brief.prompt.yml`) and adapt to your needs (replace `{{...}}` placeholders).
2. **Start LLM:** Load prompt into GitHub Models UI, GitHub CLI (`gh models`), or compatible IDE integrations.
3. **Share context:** Describe relevant artifacts/changes as input (summary, diff, incident, etc.).
4. **Review results:** Review output, clarify open points, and document decisions (README, ARCHITECTURE, ADRs).

### Typical Workflows

- **Discovery (Product Owner):** `product-owner-feature-brief` for requirements gathering → `product-owner-story-outline` for story bundles.
- **Code Review:** `review-coach` for findings, test impact, and documentation hints.
- **Architecture Enablement:** `architecture-deep-dive` explains modules/flows; `adr-drafting-assistant` supports new decisions.
- **Support & Releases:** `bug-triage-investigator` structures bug reports, `release-notes-curator` creates stakeholder updates.
- **Quality Assurance:** `test-strategy-designer` defines test packages (unit, CAP integration, UI5 E2E, performance).

### MCP-Server & Wissensquellen

- `.vscode/mcp.json` konfiguriert vier Server, die in kompatiblen IDEs sofort nutzbar sind:
  - `sap-docs` → Aggregierte SAP-Dokumentation (ABAP, CAP, UI5, Community) via HTTP-Server.
  - `cds-mcp` → SAP CAP Referenzen & Best Practices.
  - `@sap-ux/fiori-mcp-server` → Fiori Elements Patterns, UX Guidelines und Annotation-Hilfen.
  - `@ui5/mcp-server` → UI5 Control API, MVC, Routing.
- **Installation:** Für `cds-mcp` muss die CLI einmal global installiert werden, z. B. `npm install -g @cap-js/mcp-server`. Prüfe die Installation mit `cds-mcp --help`. Die anderen drei Server (inkl. `sap-docs`) werden bei Bedarf automatisch verbunden bzw. über `npx …` gestartet.
- In Kombination mit den Prompts können MCP-Server als „Knowledge Provider" dienen, um technische Details während der Anforderungs- oder Review-Phase abzufragen.

> Tipp: Ergänze bei Bedarf projektspezifische Details (z. B. betroffene Entities, Handler, Commands), damit das LLM zielgerichtet antwortet.

---

## 🏃 Starting the Development Server

### Option 1: Watch Mode (recommended for development)

```bash
npm run watch
```

This command:

- Starts CAP server with **auto-reload**
- Uses `cds-tsx` (TypeScript without compilation)
- Watches for changes in `srv/`, `db/`, `app/`
- Automatically opens browser at `http://localhost:4004`

**Server Output:**

```
[cds] - loaded model from 3 file(s):
  db/data-model.cds
  srv/track-service/track-service.cds
  app/services.cds

[cds] - connect to db > sqlite { database: ':memory:' }
[cds] - serving TrackService { path: '/odata/v4/track' }

[cds] - server listening on { url: 'http://localhost:4004' }
[cds] - launched at 16/10/2025, 10:23:45, version: 9.x.x, in: 1.234s
```

### Option 2: Build & Serve (for production check)

```bash
npm run build
npm start
```

### Swagger UI & OpenAPI (development only)

When starting via `npm run watch` or `cds watch`, the `cds-swagger-ui-express` plugin automatically registers a Swagger UI for the TrackService:

- **Swagger UI:** `http://localhost:4004/$api-docs/odata/v4/track/`
- **OpenAPI JSON:** `http://localhost:4004/$api-docs/odata/v4/track/openapi.json`

The interface is intended for local development only and is not included in the production build.

---

## 🧪 Using Test Users

The app uses **mocked authentication** for local development. Three test users with production role names are pre-configured:

### User 1: Max Mustermann

- **E-Mail:** `max.mustermann@test.de`
- **Password:** `max`
- **Roles:** `TimeTrackingUser`, `TimeTrackingAdmin`

### User 2: Erika Musterfrau

- **Email:** `erika.musterfrau@test.de`
- **Password:** `erika`
- **Role:** `TimeTrackingUser`

### User 3: Frank Genehmiger

- **Email:** `frank.genehmiger@test.de`
- **Password:** `frank`
- **Roles:** `TimeTrackingUser`, `TimeTrackingApprover`

**Login Flow:**

1. Open `http://localhost:4004/`
2. Select one of the apps:
   - `timetable/` - Fiori Elements list report
   - `timetracking/` - Custom UI5 dashboard
3. Log in with one of the test users

**Alternative:** Direct OData access without UI:

```bash
# GET requests (in browser or with curl)
http://localhost:4004/odata/v4/track/TimeEntries
http://localhost:4004/odata/v4/track/Users
http://localhost:4004/odata/v4/track/Projects
```

---

## 📂 Project Structure (Quick Overview)

```
cap-fiori-timetracking/
├── app/                      # 📱 UI5 Frontend Apps
│   ├── timetable/            # Fiori Elements (annotation-based)
│   └── timetracking/         # Custom UI5 (TypeScript)
├── db/                       # 💾 Data Model & CSV Test Data
│   ├── data-model.cds
│   └── data/*.csv
├── srv/                      # ⚙️ CAP Backend (100% TypeScript!)
│   └── track-service/
│       ├── track-service.ts  # Service Orchestrator
│       ├── handler/          # Commands, Validators, Services
│       └── annotations/      # UI Annotations
├── docs/                     # 📚 Documentation
│   ├── ARCHITECTURE.md       # arc42 Architecture Documentation
│   └── ADR/                  # Architecture Decision Records
├── .github/                  # 🤖 Templates, Dependabot, CODEOWNERS
├── @cds-models/              # 🔧 Generated TypeScript Types
├── test/                     # 🧪 Tests (Jest + REST Client)
├── .env.example              # ⚙️ Example Environment Variables
├── .editorconfig             # 🧹 Formatting Rules (2 Spaces, LF)
├── .nvmrc                    # 🟦 Node Version 22.20.0
├── .npmrc                    # 📦 npm Guidelines (engine-strict, audit)
├── CODE_OF_CONDUCT.md        # 🤝 Community Guidelines
├── SECURITY.md               # 🔐 Responsible Disclosure
└── package.json              # npm Scripts & Dependencies
```

---

## 🛠️ Important npm Scripts

| Command                        | Purpose                                | When to Use?                    |
| ------------------------------ | -------------------------------------- | ------------------------------- |
| `npm run watch`                | Dev server with auto-reload            | **Main command for development** |
| `npm run build`                | Compile TypeScript                     | Before commit (syntax check)    |
| `npm run format`               | Prettier formatting                    | **Before every commit (required!)** |
| `npm run generate-entry-point` | Service entry points (dev-cap-tools)   | Optional for new services       |
| `npm test`                     | Run Jest tests                         | After code changes              |
| `npm start`                    | Production-like serve                  | Final check before deployment   |

### Typical Development Workflow

```bash
# 1. Start server
npm run watch

# 2. In another terminal: edit code
# ... edit files in srv/, db/, app/ ...

# 3. Before commit: format code
npm run format

# 4. TypeScript check
npm run build

# 5. Optional: tests
npm test

# 6. Commit & push
git add .
git commit -m "feat: new feature XYZ"
git push
```

---

## First Steps After Installation

### 1. Explore the Backend (CAP Service)

**Service Endpoints:**

- **Index Page:** http://localhost:4004
- **OData Service:** http://localhost:4004/odata/v4/track
- **Service Metadata:** http://localhost:4004/odata/v4/track/$metadata
- **Fiori Preview:** http://localhost:4004/fiori.html

**Tip:** Open `test/track-service.http` in VS Code (requires REST Client extension) for pre-configured HTTP requests.

### 2. Test Frontend Apps

**Fiori Elements Timetable:**

```
http://localhost:4004/io.nimble.timetable/index.html
```

Features: list report, object page, drafts, F4 value helps

**Manage Activity Types (Fiori Elements Basic V4):**

```
http://localhost:4004/io.nimble.manageactivitytypes/index.html
```

Features: master data maintenance of activity types, table filters, inline editing; alternatively accessible via launchpad preview `http://localhost:4004/fiori.html`.

**Custom UI5 Dashboard:**

```
http://localhost:4004/io.nimble.timetracking/index.html
```

Features: dashboard, single planning calendar, charts

### 3. Inspect the Database

**SQLite In-Memory DB:**

The app uses an **in-memory database** by default. Data is lost when the server restarts. Test data is automatically loaded from `db/data/*.csv`.

**Persistent DB (optional):**

```json
// package.json → cds.requires.db
"db": {
  "kind": "sqlite",
  "credentials": {
    "database": "db.sqlite"
  }
}
```

Then:

```bash
cds deploy --to sqlite:db.sqlite
npm run watch
```

### 4. Test Code Changes

**Example: Add a new validation**

1. Open `srv/track-service/handler/validators/TimeEntryValidator.ts`
2. Add new rule
3. Server automatically reloads (watch mode!)
4. Test with `test/track-service.http`

---

### 5. Set Up SAP CAP Console

The [SAP CAP Console](https://cap.cloud.sap/docs/tools/console) is a native desktop app (Windows/macOS), inspired by the CAP Developer Dashboard & OpenLens, that bundles local development, BTP deployments, and monitoring in one interface.

1. **Download & Installation**
   Download the application from [SAP Tools](https://tools.hana.ondemand.com/#cloud-capconsole) and install it. Before starting our project, optionally run `npm run watch` so the console can detect the local CAP backend.

2. **Add Project**
   On startup, the console scans running CAP processes (Java/JS). Our project will appear automatically in the list; from the menu (`…`) you can select "Remember Project" to keep it displayed permanently. Alternatively, choose `Add Project` → select root folder.

3. **Monitoring & Structure**
   The app visualizes the `mta.yaml`, shows status, CPU/RAM, and live logs for each module. Through the `@cap-js/console` plugin installed in the project, log levels can be adjusted without restarting. Without a running service, live metrics are unavailable, but metadata remains visible.

4. **Environments & Deployment**
   Environment configurations are stored as `.cds/*.yaml` in your project (see example `.cds/trial.yaml.example`). Switch between `local`, `dev`, `prod` via the top bar. For Cloud Foundry deployments, a wizard guides you through authentication, entitlement checks, and service creation – either completely "in-app" (with bundled CLIs) or as a command collection for your terminal.

5. **Security & SSH**
   For plugin features in BTP, the console creates an SSH tunnel to the application container when needed. Ensure you and your team understand the implications (see "Security" chapter in the product documentation) and enable SSH only as long as necessary.

> Note: Currently, CAP Console does not support µ-services, MTX, or Kyma deployments. The focus is on CAP projects targeting SAP BTP Cloud Foundry.

---

## ☁️ Configure Attachments on SAP BTP (optional)

By default, the attachments plugin (`@cap-js/attachments`) stores binary data in the connected database. For production scenarios with larger files or compliance requirements, you can connect additional SAP BTP services:

1. **SAP Object Store** – stores files in S3-compatible storage.
2. **SAP Malware Scanning Service** – automatically scans uploads for viruses/malware.

**Sample Steps (Cloud Foundry):**

```bash
# Object store for attachments
cf create-service objectstore standard cap-fiori-timetracking-attachments

# Malware scanner for uploads
cf create-service malwarescanning standard cap-fiori-timetracking-malware-scanner

# Application logging service for centralized logs
cf create-service application-logs standard cap-fiori-timetracking-logging

# Service bindings are handled by mta.yaml during cf deploy

# Connectivity + destination for holiday API
cf create-service connectivity lite cap-fiori-timetracking-connectivity
cf create-service destination lite cap-fiori-timetracking-destination
```

> For detailed configuration information (binding names, destinations, environment variables), see the official plugin documentation: [SAP Object Store](https://github.com/cap-js/attachments#using-sap-object-store) and [Malware Scanning Service](https://github.com/cap-js/attachments#using-sap-malware-scanning-service).

---

## ☁️ Deploy to SAP BTP (Cloud Foundry)

1. **Prerequisites:** Install the Cloud Foundry CLI with MultiApps plugin (`cf install-plugin multiapps`) and the Cloud MTA Build Tool (`npm install -g mbt` or via binary).
2. **Prepare Services:** Create the services from the section above once per subaccount (`hana`, `objectstore`, `malwarescanning`, `application-logs`, `connectivity`, `destination`) **plus** `identity` (IAS) and `identity-authorization` (AMS). Name them according to `mta.yaml` (`cap-fiori-timetracking-ias`, `cap-fiori-timetracking-ams`).
3. **Configure IAS:** Create a service key for `cap-fiori-timetracking-ias`, enable `xsuaa-cross-consumption`, and create role collections that reference the templates from `xs-security.json`.
4. **Prepare AMS:** Assign the `Identity_Provisioner` role to your technical deployment user and ensure the AMS API is reachable (service key for `cap-fiori-timetracking-ams`). Policies are deployed later by the deployer module.
5. _(Optional)_ **Clean Up:** `npm run clean` removes existing build artifacts (`gen/`, `mta_archives/`, UI5 `dist/`) before a fresh build.
6. **Run Build:** `npm run build:mta` (assumes `npm ci` was called beforehand). The generated MTAR is located at `gen/mta.mtar`.
7. **Deploy:** `npm run deploy:cf`
8. **Verify Bindings:** `cf services` should show that `cap-fiori-timetracking-srv` is bound to DB, attachments, malware scanner, connectivity, destination, application logging **as well as** IAS & AMS. Additionally, the task module `cap-fiori-timetracking-ams-policies-deployer` appears, which automatically deploys the DCL files (`ams/dcl`).

> Tip: `npm run build:mta` creates a production build by default. For quick iterations, you can optionally run `npx mbt build -p cf --dev -t gen --mtar mta.mtar` to skip optimizations.

Through the combination of `mta.yaml`, clearly separated build/run phases, and external service bindings, the solution fulfills central [12-factor principles](https://12factor.net/) and can be classified as a cloud-native application.

---

## 🐛 Troubleshooting

### Problem: `Cannot find module '#cds-models/TrackService'`

**Solution:**

> TypeScript types are automatically generated by `@cap-js/cds-typer` on save/build. `npm run generate-entry-point` is only required if tooling (e.g., dev-cap-tools) needs new entry points.

---

### Problem: Port 4004 is already in use

**Solution 1: Use a different port**

```bash
PORT=4005 npm run watch
```

**Solution 2: Kill existing process**

```bash
# Windows
netstat -ano | findstr :4004
taskkill /PID <PID> /F

# macOS/Linux
lsof -ti:4004 | xargs kill -9
```

---

### Problem: TypeScript errors in editor

**Symptoms:**

- Red squiggly lines in `.ts` files
- "Cannot find module" errors

**Solution:**

1. Restart VS Code: `Ctrl+Shift+P` → "Reload Window"
2. Restart TypeScript server: `Ctrl+Shift+P` → "TypeScript: Restart TS Server"
3. Reinstall dependencies:

```bash
rm -rf node_modules package-lock.json
npm install
```

---

### Problem: UI5 apps won't load

**Symptoms:**

- Blank page or 404 at `/timetable/` or `/timetracking/`

**Solution:**

```bash
# Reinstall UI5 workspaces
npm install --workspaces

# Restart server
npm run watch
```

---

### Problem: CDS error "Cannot load model"

**Symptoms:**

```
[cds] - failed to load model from db/data-model.cds
Syntax error: Unexpected token...
```

**Solution:**

1. Check CDS syntax in `db/data-model.cds` or `srv/track-service/track-service.cds`
2. Use CDS Language Support extension for syntax highlighting
3. Test with:

```bash
cds compile db/data-model.cds
```

---

### Problem: Prettier not formatting

**Solution:**

```bash
# Install Prettier globally
npm install -g prettier

# Format manually
npm run format

# VS Code: Enable format on save
# settings.json:
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

---

### Problem: Login loop / 401 Unauthorized

**Symptoms:**

- Endless redirect to login page
- "401 Unauthorized" on OData requests

**Solution:**

Check `package.json` → `cds.requires.auth`:

```json
"auth": {
  "kind": "mocked",  // MUST be "mocked" for local development
  "users": { ... }
}
```

If `"kind": "xsuaa"` or `"jwt"` → change to `"mocked"` and restart server.

---

## 📚 Weiterführende Ressourcen

### Internal Documentation

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Complete arc42 architecture documentation
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Guidelines for contributors
- **[ADR Directory](docs/ADR/)** - All architecture decisions (11 ADRs)
- **[README.md](README.md)** - Executive summary & features

### External Links

- **[SAP CAP Documentation](https://cap.cloud.sap)** - Official CAP framework
- **[SAPUI5 SDK](https://ui5.sap.com)** - UI5 API reference
- **[TypeScript Handbook](https://www.typescriptlang.org/docs/)** - TypeScript documentation
- **[Feiertage API](https://feiertage-api.de)** - Used holiday API

### Video Tutorials (extern)

- [SAP CAP in 100 Seconds](https://www.youtube.com/results?search_query=sap+cap+tutorial)
- [SAPUI5 Crash Course](https://www.youtube.com/results?search_query=sapui5+tutorial)

---

## ✅ Quick Checklist - Is Everything Ready?

Review these items before starting development:

- [ ] Node.js ≥22.x installed (`node --version`)
- [ ] Java ≥17 installed (`java -version`)
- [ ] npm ≥10.x installed (`npm --version`)
- [ ] Repository cloned
- [ ] `npm install` completed successfully
- [ ] Optional: `npm run generate-entry-point` (if dev-cap-tools needs entry points)
- [ ] `npm run watch` runs without errors
- [ ] Browser opens `http://localhost:4004`
- [ ] Login with test user works
- [ ] Fiori apps accessible (`/io.nimble.timetable/`, `/io.nimble.timetracking/`, `/io.nimble.manageactivitytypes/`)
- [ ] VS Code extensions installed (CDS, ESLint, Prettier)

**All set ✅? Perfect! You're ready to go!** 🚀

---

---

## 🔁 Inner Loop Checklist

| Step                           | Goal                                 | Empfehlung / Command                                  |
| --------------------------------- | ------------------------------------ | ----------------------------------------------------- |
| **1. Start watch**              | CAP + UI5 hot reload, local mocks   | `npm run watch` uses `cds watch` (mock auth, SQLite) |
| **2. Coding & docs**            | Feature build, review ADR            | Editor + `docs/ARCHITECTURE.md`/ADRs in view         |
| **3. Run tests**              | Prevent regression                   | `npm test` or `npm run test:watch`                  |
| **4. Lint & Format**              | Ensure style & quality               | `npx eslint …`, `npx prettier --check …`              |
| **5. Manual Check / CAP Console** | UI/Service Smoke-Test, Monitoring    | REST Client (`tests/*.http`), Swagger UI, CAP Console |
| **6. Optional Entry Points**      | Dev-tool generation (dev-cap-tools)  | `npm run generate-entry-point` if needed             |
| **7. Commit & PR**                | Share change                      | Conventional commit, use PR template               |

> Tip: Keep the inner loop small (<15 minutes). Only move to the outer loop (PR, review, CI, deployment) once code & tests are stable locally.

---

## 🆘 Need Help?

If you get stuck:

1. **Check known issues:** [docs/ARCHITECTURE.md → Chapter 11](docs/ARCHITECTURE.md#11-risks-and-technical-debt)
2. **Create a GitHub issue:** [GitHub Issues](https://github.com/nimble-123/cap-fiori-timetracking/issues)
3. **SAP Community:** [SAP Community - CAP Forum](https://community.sap.com)

---

**Happy Coding! 🎉**

_Next step: Read the [Architecture Documentation](docs/ARCHITECTURE.md) for deep dive._
