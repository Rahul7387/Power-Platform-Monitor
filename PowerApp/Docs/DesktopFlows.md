# Listing Desktop Flows in Power Apps

Desktop flows (Power Automate Desktop / RPA flows) are stored
in Dataverse as Workflow records with category = 6.
The Power Automate Management connector cannot list them.

---

## STEP 1 — Add the Dataverse connector

Data → Add data → search "Microsoft Dataverse"
(may also appear as "Common Data Service")
→ Add

---

## STEP 2 — Set your Desktop Flows gallery Items

Set the gallery Items property to:

    Filter(
        Processes,
        category = 6
    )

"Processes" is the Dataverse table name for workflows/flows.
category = 6 means Desktop Flow specifically.

---

## Category values for reference
    0 = Classic workflow
    1 = Dialog
    2 = Business Rule
    3 = Action
    4 = Business Process Flow
    5 = Modern Flow (Cloud Flow)
    6 = Desktop Flow  ← this is what you want

---

## STEP 3 — Gallery template labels for desktop flows

    Name:           ThisItem.name
    Status:         If(ThisItem.statecode = 1, "● Active", "● Inactive")
    Status Color:   If(ThisItem.statecode = 1, RGBA(16,124,16,1), RGBA(196,43,28,1))
    Description:    ThisItem.description
    Modified:       Text(ThisItem.modifiedon, "dd mmm yyyy")
    Created:        Text(ThisItem.createdon, "dd mmm yyyy")
    Owner:          ThisItem.'Owning User'.'Full Name'
    Flow ID:        ThisItem.workflowid

Status values:
    statecode = 1  → Active
    statecode = 0  → Draft / Inactive

---

## STEP 4 — Search filter for desktop flows

TextInput (DF_Search) OnChange:

    Set(gDesktopFlows,
        Filter(
            Processes,
            category = 6 &&
            (IsBlank(DF_Search.Text) ||
             DF_Search.Text in name)
        )
    )

---

## STEP 5 — Load desktop flows on screen visible

DesktopFlowsScreen OnVisible:

    ClearCollect(gDesktopFlows,
        Filter(Processes, category = 6));
    Set(gFilteredDF, gDesktopFlows)

---

## STEP 6 — Desktop Flows stat card on Dashboard

    Text(CountRows(Filter(Processes, category = 6))) & " Desktop Flows"

Active desktop flows:

    Text(CountRows(Filter(Processes, category = 6 && statecode = 1))) & " Active"

---

## STEP 7 — Details for selected desktop flow

When user taps a desktop flow row:

    Set(gSelectedDesktopFlow, ThisItem);
    Set(gDetailType, "desktop");
    Navigate(DetailScreen)

On DetailScreen, show:

    Name:        gSelectedDesktopFlow.name
    Status:      If(gSelectedDesktopFlow.statecode=1,"Active","Inactive")
    Owner:       gSelectedDesktopFlow.'Owning User'.'Full Name'
    Description: gSelectedDesktopFlow.description
    Modified:    Text(gSelectedDesktopFlow.modifiedon,"dd mmm yyyy hh:mm")
    Flow ID:     gSelectedDesktopFlow.workflowid

---

## NOTE — Permissions needed for Dataverse

You need at least "Basic User" role in your Dataverse environment
to read the Processes table.

If you get a permissions error:
  admin.powerplatform.microsoft.com
  → Environments → GS Healthcare LLC (default)
  → Settings → Users → assign "Basic User" role to yourself

Most users already have this role — it's the minimum default role.

