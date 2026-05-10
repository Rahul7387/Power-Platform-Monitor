# Power Platform Monitor — Complete Studio Build Guide
> Build time: ~20 minutes. No downloads needed.

---

## STEP 1 — Create the blank app

1. Go to **make.powerapps.com**
2. Click **Create** → **Blank app** → **Blank canvas app**
3. Name: `PowerPlatformMonitor`
4. Format: **Tablet**
5. Click **Create**

---

## STEP 2 — Add data sources

Left panel → **Data** icon → **Add data**:
- Search and add: **Power Automate Management**
- Search and add: **Power Apps for Makers**

---

## STEP 3 — App.OnStart formula

Click **App** in the left tree view → **OnStart** property → paste:

```
Set(gAdminConnected, false);
Set(gHideAdminForever, false);
Set(gDetailType, "cloud");
Set(gDetailTab, "Info");
Set(gCurrentEnvName, "Default Environment");
Set(gCurrentEnv,
    First(PowerAutomateManagement.ListEnvironments().value).name);
Set(gCurrentEnvName,
    First(PowerAutomateManagement.ListEnvironments().value)
        .properties.displayName);
Set(gLastRefresh, Now());
ClearCollect(gCloudFlows,
    PowerAutomateManagement.ListFlows(gCurrentEnv).value);
ClearCollect(gDesktopFlows,
    PowerAutomateManagement.ListDesktopFlows(gCurrentEnv).value);
ClearCollect(gPowerApps,
    PowerAppsforMakers.GetApps().value);
Set(gFilteredCF, gCloudFlows);
Set(gFilteredDF, gDesktopFlows);
Set(gFilteredPA, gPowerApps)
```

---

## STEP 4 — Create 5 more screens

In the **Screens** panel, click **+** five times and rename them:
- `HomeScreen` (rename the default Screen1)
- `CloudFlowsScreen`
- `DesktopFlowsScreen`
- `PowerAppsScreen`
- `DetailScreen`
- `AdminScreen`

---

## STEP 5 — HomeScreen

Set `HomeScreen.OnVisible`:
```
Set(gLastRefresh, Now());
ClearCollect(gCloudFlows, PowerAutomateManagement.ListFlows(gCurrentEnv).value);
ClearCollect(gDesktopFlows, PowerAutomateManagement.ListDesktopFlows(gCurrentEnv).value);
ClearCollect(gPowerApps, PowerAppsforMakers.GetApps().value)
```

**Insert → Rectangle** (Nav bar):
- X=0, Y=0, W=1366, H=56, Fill=RGBA(15,23,42,1)

**Insert → Text label** (Title):
- X=56, Y=0, W=600, H=56
- Text: `"⚡  Power Platform Monitor"`
- Color=White, Size=18, FontWeight=Semibold, Fill=Transparent

**Insert → Button** (Refresh):
- X=1256, Y=10, W=96, H=36
- Text: `"⟳ Refresh"`
- Fill=RGBA(255,255,255,0.15), Color=White
- OnSelect:
```
Set(gLastRefresh,Now());
ClearCollect(gCloudFlows,PowerAutomateManagement.ListFlows(gCurrentEnv).value);
ClearCollect(gDesktopFlows,PowerAutomateManagement.ListDesktopFlows(gCurrentEnv).value);
ClearCollect(gPowerApps,PowerAppsforMakers.GetApps().value)
```

**Insert → 5 Rectangles** (Stat cards) at Y=100, H=90:
- Card 1: X=28, W=200, Fill=RGBA(15,108,189,1)
- Card 2: X=240, W=200, Fill=RGBA(16,124,16,1)
- Card 3: X=452, W=200, Fill=RGBA(196,43,28,1)
- Card 4: X=664, W=200, Fill=RGBA(107,33,168,1)
- Card 5: X=876, W=200, Fill=RGBA(14,116,144,1)

**Inside each card, Insert → Text labels** for value and label:
| Card | Value Formula | Label Text |
|------|--------------|------------|
| 1 | `Text(CountRows(gCloudFlows))` | "Cloud Flows" |
| 2 | `Text(CountRows(Filter(gCloudFlows, state="enabled")))` | "Active" |
| 3 | `Text(CountRows(Filter(gCloudFlows, state="disabled")))` | "Inactive" |
| 4 | `Text(CountRows(gDesktopFlows))` | "Desktop Flows" |
| 5 | `Text(CountRows(gPowerApps))` | "Power Apps" |

**Insert → 4 Buttons** (Nav tiles) at Y=220, H=160:
| Button | X | Text | OnSelect |
|--------|---|------|----------|
| Cloud | 28 | "☁ Cloud Flows" | `Navigate(CloudFlowsScreen)` |
| Desktop | 320 | "🖥 Desktop Flows" | `Navigate(DesktopFlowsScreen)` |
| Apps | 612 | "📱 Power Apps" | `Navigate(PowerAppsScreen)` |
| Admin | 904 | "🔑 Admin Config" | `Navigate(AdminScreen)` |

---

## STEP 6 — CloudFlowsScreen

Set `CloudFlowsScreen.OnVisible`:
```
ClearCollect(gFilteredCF, gCloudFlows)
```

**Nav bar** (same as HomeScreen — copy/paste the rectangle + label + refresh button)

**Insert → Text input** (Search):
- X=28, Y=80, W=300, H=40
- HintText: `"Search flows..."`
- OnChange:
```
ClearCollect(gFilteredCF,
    Filter(gCloudFlows,
        IsBlank(TextInput1.Text) ||
        TextInput1.Text in properties.displayName ||
        TextInput1.Text in properties.creator.userPrincipalName))
```
*(Replace TextInput1 with the actual control name)*

**Insert → Dropdown** (Status filter):
- X=340, Y=80, W=160, H=40
- Items: `["All", "enabled", "disabled"]`
- OnChange:
```
ClearCollect(gFilteredCF,
    Filter(gCloudFlows,
        Dropdown1.Selected.Value = "All" ||
        properties.state = Dropdown1.Selected.Value))
```

**Insert → Vertical gallery**:
- X=28, Y=135, W=1330, H=620
- Items: `gFilteredCF`
- TemplateSize: 56

**Inside the gallery template, Insert → Labels**:
| Label | Text | X | W |
|-------|------|---|---|
| Name | `ThisItem.properties.displayName` | 8 | 280 |
| Status | `If(ThisItem.properties.state="enabled","● Active","● Inactive")` | 296 | 110 |
| Trigger | `Coalesce(First(ThisItem.properties.definitionSummary.triggers).type,"—")` | 414 | 150 |
| Modified | `Text(DateTimeValue(ThisItem.properties.lastModifiedTime),"dd mmm yyyy")` | 572 | 170 |
| Owner | `ThisItem.properties.creator.userPrincipalName` | 750 | 250 |

Status label Color:
```
If(ThisItem.properties.state="enabled", RGBA(16,124,16,1), RGBA(196,43,28,1))
```

**Inside gallery, Insert → Button** (Details):
- X=1080, Y=8, Text="Details →"
- OnSelect:
```
Set(gSelectedFlow, ThisItem);
Set(gDetailType, "cloud");
Navigate(DetailScreen)
```

---

## STEP 7 — DesktopFlowsScreen

Same structure as CloudFlowsScreen but:

OnVisible: `ClearCollect(gFilteredDF, gDesktopFlows)`

Gallery Items: `gFilteredDF`

Gallery labels:
| Label | Text |
|-------|------|
| Name | `ThisItem.properties.displayName` |
| Status | `If(ThisItem.properties.state="enabled","● Active","● Inactive")` |
| Machine | `Coalesce(ThisItem.properties.machineName,"Unassigned")` |
| Mode | `Coalesce(ThisItem.properties.runMode,"Attended")` |
| Owner | `ThisItem.properties.creator.userPrincipalName` |

Details button OnSelect:
```
Set(gSelectedFlow, ThisItem);
Set(gDetailType, "desktop");
Navigate(DetailScreen)
```

---

## STEP 8 — PowerAppsScreen

OnVisible: `ClearCollect(gFilteredPA, gPowerApps)`

Gallery Items: `gFilteredPA`

Gallery labels:
| Label | Text |
|-------|------|
| Icon | `If(ThisItem.properties.appType="ModelDriven","⚙","📱")` |
| Name | `Coalesce(ThisItem.properties.displayName, ThisItem.name)` |
| Type | `Coalesce(ThisItem.properties.appType,"Canvas")` |
| Sharing | `If(Not(IsBlank(ThisItem.properties.sharedWithTenantEveryone)),"🌐 Everyone","👥 Specific")` |
| Published | `Text(DateTimeValue(ThisItem.properties.lastPublishedTime),"dd mmm yyyy")` |
| Owner | `ThisItem.properties.owner.userPrincipalName` |

Details button OnSelect:
```
Set(gSelectedApp, ThisItem);
Set(gDetailType, "app");
Navigate(DetailScreen)
```

---

## STEP 9 — DetailScreen

OnVisible:
```
Set(gDetailTab, "Info");
If(gDetailType <> "app",
    ClearCollect(gFlowRuns,
        PowerAutomateManagement.ListFlowRuns(gCurrentEnv, gSelectedFlow.name).value))
```

**Back button**: `Back()`

**Labels for info panel** (left side, X=28):
| Label | Text |
|-------|------|
| Name | `If(gDetailType="app", Coalesce(gSelectedApp.properties.displayName,gSelectedApp.name), gSelectedFlow.properties.displayName)` |
| Status | `If(gDetailType="app","Power App",If(gSelectedFlow.properties.state="enabled","● Active","● Inactive"))` |
| Type | `If(gDetailType="cloud","Cloud Flow",If(gDetailType="desktop","Desktop Flow","Power App"))` |
| Environment | `gCurrentEnvName` |
| Owner | `If(gDetailType="app",gSelectedApp.properties.owner.userPrincipalName,gSelectedFlow.properties.creator.userPrincipalName)` |
| Modified | `If(gDetailType="app",Text(DateTimeValue(gSelectedApp.properties.lastPublishedTime),"dd mmm yyyy"),Text(DateTimeValue(gSelectedFlow.properties.lastModifiedTime),"dd mmm yyyy"))` |

**Enable button** OnSelect:
```
If(gDetailType <> "app",
    PowerAutomateManagement.EnableFlow(gCurrentEnv, gSelectedFlow.name);
    Notify("Flow enabled ✅", NotificationType.Success),
    Notify("Manage apps via Admin Centre", NotificationType.Information))
```

**Disable button** OnSelect:
```
If(gDetailType <> "app",
    PowerAutomateManagement.DisableFlow(gCurrentEnv, gSelectedFlow.name);
    Notify("Flow disabled", NotificationType.Warning),
    Notify("Manage apps via Admin Centre", NotificationType.Information))
```

**Run Now button** OnSelect:
```
If(gDetailType = "cloud",
    PowerAutomateManagement.RunFlow(gCurrentEnv, gSelectedFlow.name, {inputs: {}}),
    Notify("Launch triggered", NotificationType.Information))
```

**Run History gallery** (right side):
- Items: `SortByColumns(gFlowRuns, "startTime", SortOrder.Descending)`
- TemplateSize: 50

Labels inside:
| Label | Text |
|-------|------|
| Status | `ThisItem.status` |
| Start | `Text(DateTimeValue(ThisItem.startTime),"dd mmm hh:mm")` |
| Duration | `Concatenate(Text(Round(DateDiff(DateTimeValue(ThisItem.startTime),If(IsBlank(ThisItem.endTime),Now(),DateTimeValue(ThisItem.endTime)),TimeUnit.Seconds),"0")),"s")` |
| Error | `Coalesce(ThisItem.error.message,"—")` |

---

## STEP 10 — AdminScreen

**4 Text inputs**:
- `AD_InTenant` — HintText: "Tenant ID (Azure AD → Overview)"
- `AD_InEnv` — HintText: "Environment ID (blank = Default)"
- `AD_InClient` — HintText: "Client ID (App Registration)"
- `AD_InSecret` — HintText: "Client Secret"

**Save & Connect button** OnSelect:
```
If(IsBlank(AD_InTenant.Text) || IsBlank(AD_InClient.Text) || IsBlank(AD_InSecret.Text),
    Notify("Tenant ID, Client ID and Secret are required", NotificationType.Error),
    Set(gTenantId, AD_InTenant.Text);
    Set(gClientId, AD_InClient.Text);
    Set(gClientSecret, AD_InSecret.Text);
    Set(gEnvId, If(IsBlank(AD_InEnv.Text),"Default",AD_InEnv.Text));
    Set(gCurrentEnvName, If(IsBlank(AD_InEnv.Text),"Default Environment",AD_InEnv.Text));
    Set(gAdminConnected, true);
    ClearCollect(gCloudFlows, PowerAutomateManagement.ListFlows(gTenantId).value);
    ClearCollect(gDesktopFlows, PowerAutomateManagement.ListDesktopFlows(gTenantId).value);
    ClearCollect(gPowerApps, PowerAppsforMakers.GetApps().value);
    Notify("Connected — all tenant data loaded ✅", NotificationType.Success);
    Navigate(HomeScreen))
```

**To permanently hide the Admin button** on all screens:
- Add `Set(gHideAdminForever, true)` to App.OnStart
- Set each admin nav button's `Visible` property to `Not(gHideAdminForever)`

---

## STEP 11 — Publish & Share

1. **File → Save**
2. **File → Publish**  
3. **Share** with users in your org

---

## Required API Permissions (for Admin service principal)

All must be **Application** type with **Admin Consent** granted:
- `Flow.Read.All`
- `Flow.Manage.All`
- `PowerApps.Read.All`
- `PowerApps.Manage.All`
- `User.Read.All`
- `PowerPlatform.Admin.Read.All`
