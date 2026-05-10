# ⚡ Power Platform Monitor

A **Canvas App for Power Apps** that gives admins a single dashboard to monitor all **Cloud Flows**, **Desktop Flows**, and **Power Apps** across a Microsoft 365 tenant — with live status, run history, search/filter, and enable/disable controls.

![Power Platform Monitor Preview](PreviewApp/PowerPlatformMonitor_Preview.html)

---

## 📋 Features

| Screen | What it does |
|--------|-------------|
| 🏠 **Dashboard** | Stat cards: total/active/inactive flows, desktop flows, apps count |
| ☁ **Cloud Flows** | Full list with search, status filter, trigger type, owner, last run status |
| 🖥 **Desktop Flows** | RPA flows via Dataverse — machine, run mode, status, owner |
| 📱 **Power Apps** | Canvas & Model-Driven apps — sharing, type, owner, published date |
| 📋 **Detail View** | Flow/app info, run history, enable/disable/run controls |
| 🔑 **Admin Config** | Enter Tenant ID, Environment ID, Client ID & Secret to see all tenant assets |

---

## 🚀 Quick Start

### Prerequisites
- Power Apps license (per-user or per-app)
- Access to [make.powerapps.com](https://make.powerapps.com)
- Power Platform Environment ID (from your environment URL)
- For all-tenant view: **System Administrator** role in the environment

### Option A — Build in Power Apps Studio (Recommended)

1. Go to [make.powerapps.com](https://make.powerapps.com)
2. **Create → Blank canvas app → Tablet format**
3. Name it `PowerPlatformMonitor`
4. Add data sources (Data panel → Add data):
   - `Power Automate Management`
   - `Power Apps for Makers`
   - `Microsoft Dataverse` *(for desktop flows)*
5. Follow [`PowerApp/Docs/StudioBuildGuide.md`](PowerApp/Docs/StudioBuildGuide.md) — every formula is copy-paste ready

### Option B — Pack with PAC CLI

```bash
# Install PAC CLI
npm install -g @microsoft/powerplatform-cli

# Pack the source files into an importable .msapp
pac canvas pack --sources ./PowerApp --msapp PowerPlatformMonitor.msapp

# Import via make.powerapps.com → Apps → Import canvas app
```

---

## 📁 Repository Structure

```
Power-Platform-Monitor/
│
├── README.md                          # This file
│
├── PowerApp/                          # Canvas App source (PAC CLI format)
│   ├── _CanvasManifest.json           # App manifest
│   ├── _AppCheckerResult.sarif        # App checker (empty)
│   ├── DataSources.json               # Connector references
│   ├── Connections.json               # Connection stubs
│   ├── Themes.json                    # Theme config
│   │
│   ├── Src/                           # Power Fx YAML screens
│   │   ├── App.fx.yaml                # App.OnStart formula
│   │   ├── HomeScreen.fx.yaml         # Dashboard
│   │   ├── CloudFlowsScreen.fx.yaml   # Cloud flows list
│   │   ├── DesktopFlowsScreen.fx.yaml # Desktop/RPA flows
│   │   ├── PowerAppsScreen.fx.yaml    # Power Apps list
│   │   ├── DetailScreen.fx.yaml       # Flow/App detail + run history
│   │   └── AdminScreen.fx.yaml        # Admin credentials screen
│   │
│   └── Docs/                          # Formula & setup guides
│       ├── StudioBuildGuide.md        # Step-by-step Studio build
│       ├── FinalCorrectFormulas.md    # All correct Power Fx formulas
│       ├── DesktopFlows.md            # Desktop flows via Dataverse
│       └── PermissionFix.md          # Fix EnvironmentAccessDenied
│
└── PreviewApp/
    └── PowerPlatformMonitor_Preview.html   # Interactive HTML preview
```

---

## 🔌 Data Sources & Connectors

| Connector | Used for | Required permissions |
|-----------|----------|---------------------|
| **Power Automate Management** | List cloud flows, enable/disable/run | Environment Maker or System Administrator |
| **Power Apps for Makers** | List Power Apps | Basic access |
| **Microsoft Dataverse** | List desktop flows (`category = 6`) | Basic User role |

### Key functions used

```powerfx
// List all cloud flows in environment
PowerAutomateManagement.ListFlowsInEnvironmentV2(environmentId).value

// List only your flows (no special permissions needed)
PowerAutomateManagement.ListMyFlows(environmentId).value

// Enable / Disable a flow
PowerAutomateManagement.StartFlow(environmentId, flowId)
PowerAutomateManagement.StopFlow(environmentId, flowId)

// List Power Apps
PowerAppsforMakers.GetApps().value

// List desktop flows via Dataverse
Filter(Processes, category = 6)
```

---

## ⚙️ Admin Configuration

The app includes a dedicated **Admin Config screen** where you enter:

| Field | Where to find it |
|-------|-----------------|
| **Environment ID** | make.powerapps.com URL: `.../environments/[ID]/home` |
| **Tenant ID** | Azure AD → Overview → Directory (tenant) ID |
| **Client ID** | Azure AD → App Registrations → your app → Application (client) ID |
| **Client Secret** | Azure AD → App Registrations → Certificates & secrets |

> Credentials are stored in **session memory only** — never written to disk or any database.

### Required API Permissions (Application type + Admin Consent)

```
Flow.Read.All
Flow.Manage.All
PowerApps.Read.All
PowerApps.Manage.All
User.Read.All
PowerPlatform.Admin.Read.All
```

### Hide the Admin Config button permanently

Add to `App.OnStart` after initial setup:
```powerfx
Set(gHideAdminForever, true)
```

Then set each admin button's `Visible` property to:
```powerfx
Not(gHideAdminForever)
```

---

## 🔐 Fixing EnvironmentAccessDenied

If you see `EnvironmentAccessDenied` errors:

1. Go to [admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)
2. **Environments** → click your environment
3. **Settings → Users + permissions → Users**
4. Find your account → assign **System Administrator**
5. Wait 1–2 minutes → retry

See [`PowerApp/Docs/PermissionFix.md`](PowerApp/Docs/PermissionFix.md) for full details.

---

## 📊 App Variables Reference

| Variable | Type | Description |
|----------|------|-------------|
| `gAdminConnected` | Boolean | True after admin credentials saved |
| `gCurrentEnv` | Text | Environment ID |
| `gCurrentEnvName` | Text | Environment display name |
| `gTenantId` | Text | Azure AD Tenant ID |
| `gClientId` | Text | Service Principal Client ID |
| `gClientSecret` | Text | Client secret (session only) |
| `gCloudFlows` | Table | All cloud flows |
| `gDesktopFlows` | Table | All desktop flows |
| `gPowerApps` | Table | All Power Apps |
| `gFilteredCF` | Table | Filtered cloud flows |
| `gFilteredDF` | Table | Filtered desktop flows |
| `gFilteredPA` | Table | Filtered apps |
| `gFlowRuns` | Table | Run history for selected flow |
| `gSelectedFlow` | Record | Selected cloud/desktop flow |
| `gSelectedApp` | Record | Selected Power App |
| `gDetailType` | Text | `"cloud"` \| `"desktop"` \| `"app"` |
| `gDetailTab` | Text | `"Info"` \| `"Run History"` |
| `gHideAdminForever` | Boolean | Set true to hide admin button |
| `gLastRefresh` | DateTime | Last data refresh timestamp |

---

## 🖥 Interactive Preview

Open [`PreviewApp/PowerPlatformMonitor_Preview.html`](PreviewApp/PowerPlatformMonitor_Preview.html) in any browser for a **fully interactive demo** of all screens with sample data — no Power Apps licence required.

---

## 🗺 Roadmap

- [ ] Desktop flow run history via Dataverse
- [ ] Flow error rate charts (Power BI embedded)
- [ ] Email alerts for failed flows
- [ ] Bulk enable/disable selected flows
- [ ] Export flow inventory to Excel
- [ ] Solution ZIP for direct import (pending PAC CLI packaging)

---

## 🤝 Contributing

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-feature`
3. Edit the `.fx.yaml` files in `PowerApp/Src/`
4. Pack with PAC CLI: `pac canvas pack --sources ./PowerApp --msapp test.msapp`
5. Test in Power Apps Studio
6. Submit a pull request

---

## 📄 License

MIT License — free to use, modify and distribute.

---

## 🙏 Acknowledgements

Built with:
- [Microsoft Power Apps](https://powerapps.microsoft.com)
- [Power Automate Management Connector](https://learn.microsoft.com/connectors/flowmanagement/)
- [Power Apps for Makers Connector](https://learn.microsoft.com/connectors/powerappsforappmakers/)
- [Microsoft Dataverse](https://learn.microsoft.com/power-apps/maker/data-platform/)
