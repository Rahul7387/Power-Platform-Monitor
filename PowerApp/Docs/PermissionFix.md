# Fix: EnvironmentAccessDenied
## Your environment: GS Healthcare LLC (default)

---

## WHY THIS HAPPENS

ListFlowsInEnvironmentV2 requires you to have the
"Environment Maker" or "System Administrator" role
in that Power Platform environment.

You currently have neither — so all listing functions fail.

---

## FIX 1 — Assign yourself the role (takes 2 minutes)
## Do this FIRST before anything else

1. Open https://admin.powerplatform.microsoft.com
   (you need Global Admin or Power Platform Admin in Azure AD)

2. Click "Environments" in the left nav

3. Click "GS Healthcare LLC (default)"

4. Click "Settings" (top menu)

5. Click "Users + permissions" → "Users"

6. Find your account → click it

7. Assign role: "System Administrator" or "Environment Maker"

8. Click Save

9. Wait 1-2 minutes, then try again in Power Apps

---

## FIX 2 — If you ARE already a Global Admin in Azure AD

Global Admin in Azure AD does NOT automatically give you
Power Platform environment permissions.

You need to explicitly grant yourself access:

1. Go to https://admin.powerplatform.microsoft.com
2. Click the gear icon (Settings) → top right
3. Enable "Allow Global Admins to administer Power Platform"
   (this grants implicit admin access)

OR run this PowerShell (requires Power Platform admin module):

    Install-Module -Name Microsoft.PowerApps.Administration.PowerShell
    Add-PowerAppsAccount
    Add-PowerAppsEnvironmentUser `
        -EnvironmentName "063ab74f-56d1-429a-b96d-a24a572025de" `
        -RoleName "EnvironmentAdmin" `
        -PrincipalType "User" `
        -PrincipalObjectId "[your Azure AD user object ID]"

---

## FIX 3 — Works RIGHT NOW without any permission changes
## Use ListMyFlows() — shows flows you own or that are shared with you

In your Cloud Flows gallery, set Items to:

    PowerAutomateManagement.ListMyFlows().value

This works immediately — no environment role needed.
It shows all flows owned by or shared with YOUR account.

If you created the flows or are co-owner, they will appear.

---

## FIX 4 — See ALL flows using the Admin connector
## (requires System Administrator role — see Fix 1 first)

After you have System Administrator role:

1. Add data source: "Power Platform for Admins"

2. Check IntelliSense: PowerPlatformForAdmins.
   Look for functions that contain "Flow"

3. The admin connector should show functions like:
   GetEnvironmentFlows  or  GetFlowsAsAdmin  or  ListFlowsAsAdmin

---

## QUICKEST PATH TO A WORKING APP TODAY

Step 1: Do Fix 1 (assign System Administrator role) — 2 minutes
Step 2: Come back to Power Apps Studio
Step 3: Set gallery Items to:
    PowerAutomateManagement.ListFlowsInEnvironmentV2(
        "Default-063ab74f-56d1-429a-b96d-a24a572025de"
    ).value
Step 4: It will work

---

## IF YOU CANNOT GET ADMIN ACCESS

Use ListMyFlows() — it works right now:
    PowerAutomateManagement.ListMyFlows().value

The app will show all flows YOU own.
To see other people's flows you must have environment admin rights.

