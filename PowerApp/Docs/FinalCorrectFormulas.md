# Final Correct Formulas — Based on Your Actual IntelliSense
# Functions confirmed from your screenshots

## Your Environment ID (from URL bar):
## Default-063ab74f-56d1-429a-b96d-a24a572025de

---

## CORRECT FUNCTION NAMES (confirmed from your screenshots)

  PowerAutomateManagement.ListFlowsInEnvironmentV2(environment)
    → Lists ALL flows in an environment ✅ USE THIS

  PowerAutomateManagement.ListMyFlows()
    → Lists only YOUR flows (no environment needed)

  PowerAutomateManagement.ListUserEnvironments()
    → Lists environments you have access to

  PowerAutomateManagement.GetFlow(environment, flowId)
    → Get a single flow

  PowerAutomateManagement.StartFlow(environment, flowId)
    → Enable/start a flow ✅ (NOT EnableFlow)

  PowerAutomateManagement.StopFlow(environment, flowId)
    → Disable/stop a flow ✅ (NOT DisableFlow)

  PowerAutomateManagement.ResubmitFlow(environment, flowId, runId)
    → Resubmit a flow run

  PowerAutomateManagement.CancelFlowRun(environment, flowId, runId)
    → Cancel a running flow run

---

## App.OnStart

Paste this into App → OnStart:

    Set(gAdminConnected, false);
    Set(gCurrentEnv, "Default-063ab74f-56d1-429a-b96d-a24a572025de");
    Set(gCurrentEnvName, "Default Environment");
    Set(gDetailType, "cloud");
    Set(gDetailTab, "Info");
    ClearCollect(gCloudFlows, []);
    ClearCollect(gPowerApps, []);
    ClearCollect(gFlowRuns, []);
    Set(gFilteredCF, gCloudFlows);
    Set(gFilteredPA, gPowerApps);
    Set(gSelectedFlow, Blank())

---

## AdminScreen — Save & Connect button OnSelect

    Set(gCurrentEnv, AD_InEnv.Text);
    Set(gCurrentEnvName, If(IsBlank(AD_InEnvName.Text), AD_InEnv.Text, AD_InEnvName.Text));
    Set(gAdminConnected, true);
    ClearCollect(gCloudFlows,
        PowerAutomateManagement.ListFlowsInEnvironmentV2(AD_InEnv.Text).value);
    ClearCollect(gPowerApps,
        PowerAppsforMakers.GetApps().value);
    Set(gFilteredCF, gCloudFlows);
    Set(gFilteredPA, gPowerApps);
    Notify(
        "Loaded " & Text(CountRows(gCloudFlows)) & " flows ✅",
        NotificationType.Success
    );
    Navigate(HomeScreen)

AdminScreen text inputs needed:
    AD_InEnv      HintText = "Environment ID e.g. Default-063ab74f-..."
    AD_InEnvName  HintText = "Display name e.g. Default Environment"

Pre-fill AD_InEnv Default property with known ID:
    Default = "Default-063ab74f-56d1-429a-b96d-a24a572025de"

---

## CloudFlowsScreen

Screen OnVisible:
    Set(gFilteredCF, gCloudFlows)

Gallery Items:
    gFilteredCF

Search TextInput (name it: CF_Search) OnChange:
    Set(gFilteredCF,
        Filter(gCloudFlows,
            IsBlank(CF_Search.Text) ||
            CF_Search.Text in properties.displayName ||
            CF_Search.Text in properties.creator.userPrincipalName))

Status Dropdown (name it: CF_StatusDd) OnChange:
    Set(gFilteredCF,
        Filter(gCloudFlows,
            CF_StatusDd.Selected.Value = "All" ||
            properties.state = CF_StatusDd.Selected.Value))

Status Dropdown Items:
    ["All", "enabled", "disabled"]

Gallery template labels:
    Flow Name:   ThisItem.properties.displayName
    Status:      If(ThisItem.properties.state="enabled","● Active","● Inactive")
    Status Color: If(ThisItem.properties.state="enabled",RGBA(16,124,16,1),RGBA(196,43,28,1))
    Trigger:     Coalesce(First(ThisItem.properties.definitionSummary.triggers).type,"—")
    Modified:    Text(DateTimeValue(ThisItem.properties.lastModifiedTime),"dd mmm yyyy")
    Owner:       ThisItem.properties.creator.userPrincipalName
    Flow ID:     ThisItem.name

Details button OnSelect:
    Set(gSelectedFlow, ThisItem);
    Set(gDetailType, "cloud");
    Navigate(DetailScreen)

---

## Refresh button (all screens) OnSelect

    ClearCollect(gCloudFlows,
        PowerAutomateManagement.ListFlowsInEnvironmentV2(gCurrentEnv).value);
    ClearCollect(gPowerApps,
        PowerAppsforMakers.GetApps().value);
    Set(gFilteredCF, gCloudFlows);
    Set(gFilteredPA, gPowerApps);
    Set(gLastRefresh, Now());
    Notify("Refreshed ✅", NotificationType.Success)

---

## DetailScreen

Screen OnVisible:
    Set(gDetailTab, "Info");
    ClearCollect(gFlowRuns,
        PowerAutomateManagement.GetFlowRuns(
            gCurrentEnv,
            gSelectedFlow.name
        ).value
    )

Note: If GetFlowRuns does not appear in IntelliSense,
scroll down in the list — it may also be called ListFlowRuns.
Try: PowerAutomateManagement.GetFlowRuns(gCurrentEnv, gSelectedFlow.name).value
  or PowerAutomateManagement.ListFlowRuns(gCurrentEnv, gSelectedFlow.name).value

Info panel labels:
    Name:        gSelectedFlow.properties.displayName
    Status:      If(gSelectedFlow.properties.state="enabled","● Active","● Inactive")
    Status Color: If(gSelectedFlow.properties.state="enabled",RGBA(16,124,16,1),RGBA(196,43,28,1))
    Type:        "Cloud Flow"
    Environment: gCurrentEnvName
    Owner:       gSelectedFlow.properties.creator.userPrincipalName
    Modified:    Text(DateTimeValue(gSelectedFlow.properties.lastModifiedTime),"dd mmm yyyy")
    Flow ID:     gSelectedFlow.name

Enable button OnSelect:
    PowerAutomateManagement.StartFlow(gCurrentEnv, gSelectedFlow.name);
    ClearCollect(gCloudFlows,
        PowerAutomateManagement.ListFlowsInEnvironmentV2(gCurrentEnv).value);
    Set(gSelectedFlow,
        PowerAutomateManagement.GetFlow(gCurrentEnv, gSelectedFlow.name));
    Notify("Flow enabled ✅", NotificationType.Success)

Disable button OnSelect:
    PowerAutomateManagement.StopFlow(gCurrentEnv, gSelectedFlow.name);
    ClearCollect(gCloudFlows,
        PowerAutomateManagement.ListFlowsInEnvironmentV2(gCurrentEnv).value);
    Set(gSelectedFlow,
        PowerAutomateManagement.GetFlow(gCurrentEnv, gSelectedFlow.name));
    Notify("Flow disabled ⏸", NotificationType.Warning)

Run History gallery Items:
    gFlowRuns

Run history gallery template labels:
    Status:    ThisItem.status
    Start:     Text(DateTimeValue(ThisItem.startTime),"dd mmm hh:mm:ss")
    Duration:  Concatenate(
                   Text(Round(
                       DateDiff(
                           DateTimeValue(ThisItem.startTime),
                           If(IsBlank(ThisItem.endTime),
                               Now(),
                               DateTimeValue(ThisItem.endTime)),
                           TimeUnit.Seconds),
                   "0")),
               "s")
    Error:     Coalesce(ThisItem.error.message, "—")

Status color:
    Switch(ThisItem.status,
        "Succeeded", RGBA(16,124,16,1),
        "Failed",    RGBA(196,43,28,1),
        "Running",   RGBA(15,108,189,1),
        RGBA(100,116,139,1))

---

## PowerAppsScreen

Screen OnVisible:
    Set(gFilteredPA, gPowerApps)

Gallery Items:
    gFilteredPA

Search TextInput (PA_Search) OnChange:
    Set(gFilteredPA,
        Filter(gPowerApps,
            IsBlank(PA_Search.Text) ||
            PA_Search.Text in name ||
            PA_Search.Text in properties.owner.userPrincipalName))

Gallery template labels:
    Icon:       If(ThisItem.properties.appType="ModelDriven","⚙","📱")
    Name:       Coalesce(ThisItem.properties.displayName, ThisItem.name)
    Type:       Coalesce(ThisItem.properties.appType, "Canvas")
    Sharing:    If(Not(IsBlank(ThisItem.properties.sharedWithTenantEveryone)),
                    "🌐 Everyone","👥 Specific users")
    Published:  Text(DateTimeValue(ThisItem.properties.lastPublishedTime),"dd mmm yyyy")
    Owner:      ThisItem.properties.owner.userPrincipalName

Details button OnSelect:
    Set(gSelectedApp, ThisItem);
    Set(gDetailType, "app");
    Navigate(DetailScreen)

---

## Dashboard stat cards

Total Cloud Flows:  Text(CountRows(gCloudFlows))
Active Flows:       Text(CountRows(Filter(gCloudFlows, state = "enabled")))
Inactive Flows:     Text(CountRows(Filter(gCloudFlows, state = "disabled")))
Power Apps:         Text(CountRows(gPowerApps))
Last Refreshed:     Text(gLastRefresh, "dd mmm hh:mm:ss")

---

## AdminScreen — To hide the Admin Config button permanently

On each screen, find the Admin Config button.
Set its Visible property to:
    Not(gHideAdminForever)

Then in App.OnStart add:
    Set(gHideAdminForever, true)

This hides the button on all screens without deleting it.

