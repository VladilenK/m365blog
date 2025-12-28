+++
date = '2025-12-25T11:17:57-06:00'
draft = false
title = 'SharePoint Custom Scripts'
+++

Microsoft is proceeding with the staged retirement of custom scripts across SharePoint Online and OneDrive. Here is what an admin needs to know.

## Quick Introduction to SharePoint “custom scripts”
In SharePoint, “custom scripts” broadly refers to client‑side code (Javascript) added to pages. Custom scripts were actively used for sites customizations, and still present in "classic" features (like "script editor web part"). Today Microsoft consider custom scripting in SharePoint Online as a **Security risk**. Custom scripts can execute with the current user’s permissions, making it hard to govern what code runs, who added it, and where it loads resources from—opening doors to script injection, XSS, and compliance/performance issues. See more [here](https://learn.microsoft.com/en-us/sharepoint/security-considerations-of-allowing-custom-script).

When custom script is blocked, SharePoint disables numerous features (Script/Content Editor, “Save as template,” sandbox solution upload, certain file types, SharePoint Designer operations, etc.), while previously created items continue to function.  

Historically, this was common in classic SharePoint experiences and early SharePoint Online, but Microsoft has steadily constrained it to reduce risk. Modern team/communication sites, OneDrive, and even the root site block custom scripts by default. Microsoft’s guidance encourages migrating  to the SharePoint Framework (SPFx). 
Microsoft tightened policy further in 2024 by removing the easy tenant toggles and expiring the DelayDenyAddAndCustomizePagesEnforcement workaround, making persistent enablement impractical. 

In 2025, classic publishing site templates were set to have custom script disabled by default and new classic publishing sites/feature activation were blocked, with temporary opt‑outs only until March 15, 2026.

In 2025, Microsoft also introduced a way to update site property bag values without turning on custom script—further reducing reasons to flip that switch

Looking ahead, SharePoint Online is rolling out Content Security Policy (CSP): tenant admins can now set trusted script sources for modern pages, and enforcement moves from report‑only to full blocking on March 1, 2026, which will impact any customization that loads scripts from untrusted CDNs.   

So let me give you a deep technical overview Of the depreciation of custom scripting in SharePoint online along with the most recent developments 

# Today’s status and the Future of custom scripting in SharePoint Online

You can still see the following setting in the SharePoint online modern admin center 
custom scripting blocked or allowed 
the scripting setting in the classic SharePoint online admin center has been removed 

**Summary of key changes**
Microsoft is proceeding with the staged retirement of custom scripting across SharePoint online and OneDrive 

**Key updates include**
- removal of the custom scripting setting from the classic SharePoint admin center (March 2024) 
- introduction of the "DelayDenyAddAndAustomizePagesEnforcement" PowerShell parameter allowing temporary postponement of enforcement until mid November 2024 
- After enforcement administrators may temporarily enable custom script on specific sites for 24 hours using Set-SPOSite -DenyAddAndCus... . This scope is limited and requires repeated enabling

## Upcoming major enforcement milestones new lane 

### Mid-January 2026 App catalog governance enforcement 
- custom scripting will be disabled by default for the SharePoint online app catalog 
- existing scripts will continue functioning but new script development modification require a 24 hour temporary enablement window 
- app operations such as uploading updating deploying SPFx and office apps remain unaffected 

### September 15 2025 (classic publishing and template enforcement)
Per MC 1117115 updates found in SharePoint custom script deprecation and web search results.
Custom scripting is disabled by default for classic publishing templates 
- blankInternetContainer
- CMSpublishing 
- BlankInternet 
- CSPcontainer 
Creation of new classic publishing sites is blocked (UI/API) 
publishing feature activation is prevented 
existing classic publishing sites remain fully functional but operate under no script restrictions 

### New technical capabilities introduced 
 
- Property back updates without allowing custom scripts
- Microsoft introduced a new configuration option AllowWeb property back update when deny add and customize age is enabled 
that allows property bag manipulation without disabling no script mode. This applies at both site and tenant level.

This is a major governance improvement enabling modern customization without reopening scripting vulnerabilities 

## Improved administrative controls

Microsoft recommends transitioning customization to supported models
- Sharepoint framework is SPFX
- Power platform (power apps, power automate, power BI)
- Power pages
- PnP templates, APIs
- Site designs and site scripts

These alternatives offer secure supportable and scalable approaches to customization



### Impact on your organization carriage return 
**Administrators**
- Cannot permanently enable custom scripts post enforcement
- Temporary enabling 24 hours requires continuous reauthorization
- Must prepare for loss of classic features including 
  - Script editor and content editor web parts
  - Custom master pages, page layouts
  - Sandbox solution updates
  - Sharepoint designer modifications (severely restricted) 
  
**Users and site owners**
- No ability to upload certain file types, such as .aspx .ascx .swf .xap
- Save site as template, solution gallery, theme gallery, and other legacy features become unavailable
- Existing scripts continue to operate but cannot be modified without temporary scripting enablement

### Recommended customer actions 
- Inventory existing scripts and identify areas using classic web parts master pages or designer customizations
- Modernize customizations by moving to SPFx, powerapps, site scripts or PnP solutions
- Plan for 24 hour enablement window if future modifications are needed 
- Educate site owners on feature removals and modern alternatives
- Review governance policies to ensure alignment with these enforcement timelines

```

$pnpTenantSite = Get-PnPTenantSite -Connection $connectionAdmin -Identity $siteUrl -Detailed
$pnpTenantSite | Select-Object Url, Template, DenyAddAndCustomizePages, SensitivityLabel, Owner | fl

Set-PnPTenantSite -Identity $siteUrl -Connection $connectionAdmin -DenyAddAndCustomizePages:$false
Set-PnPTenantSite -Identity $siteUrl -Connection $connectionAdmin -DenyAddAndCustomizePages:$true

$tenant = Get-PnPTenant -Connection $connectionAdmin
$tenant.AllowWebPropertyBagUpdateWhenDenyAddAndCustomizePagesIsEnabled



$connectionSite = Connect-PnPOnline -ReturnConnection -Url $siteUrl -ClientId $clientid -Tenant $tenantId -Thumbprint $thumbprint
$connectionSite.Url
# Get-PnPPropertyBag -Connection $connectionSite -Key "SiteSearchClassification" # RefinableString08 aka SiteSearchClassification
Get-PnPPropertyBag -Connection $connectionSite -Key "SiteCustomSubject" # RefinableString09 aka SiteCustomSubject
Get-PnPPropertyBag -Connection $connectionSite -Key "SiteCustomProperty" # RefinableString09 aka SiteCustomSubject

Set-PnPAdaptiveScopeProperty -Key "SiteCustomProperty" -Value "TestSite" -Connection $connectionSite
Request-PnPReIndexWeb -Connection $connectionSite 




PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPPropertyBagValue -Key "SiteCustomSubject" -Value "Test" -Indexed:$true -Connection $connectionSite

Confirm
The current site is a no-script site. Do you want to temporarily enable scripting on it to allow setting property bag value?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y


# user

PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Get-PnPTenantSite -Identity $siteUrl
Get-PnPTenantSite: Unable to connect to the SharePoint Online Admin Center at 'https://s5dz3-admin.sharepoint.com' to run this cmdlet. If this URL is incorrect for your tenant, you can pass in the correct Admin Center URL using Connect-PnPOnline -TenantAdminUrl. If you are using Privileged Identity Management (PIM) on your tenant, please ensure you have activated at least the SharePoint Administrator role and allowed some time for it to activate. Error message: Attempted to perform an unauthorized operation.


PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPAdaptiveScopeProperty -Key "SiteCustomSubject" -Value "Test2"
Set-PnPAdaptiveScopeProperty: Unable to connect to the SharePoint Online Admin Center at 'https://s5dz3-admin.sharepoint.com' to run this cmdlet. If this URL is incorrect for your tenant, you can pass in the correct Admin Center URL using Connect-PnPOnline -TenantAdminUrl. If you are using Privileged Identity Management (PIM) on your tenant, please ensure you have activated at least the SharePoint Administrator role and allowed some time for it to activate. Error message: Attempted to perform an unauthorized operation.


Set-PnPTenantSite -Connection $connectionAdmin -Identity $siteUrl -DenyAddAndCustomizePages:$false

Get-PnPPropertyBag 
Set-PnPAdaptiveScopeProperty -Key "SiteCustomSubject" -Value "Test2"
Set-PnPPropertyBagValue      -Key "SiteCustomSubject" -Value "Test3" -Indexed:$true 

Set-PnPTenantSite -Connection $connectionAdmin -Identity $siteUrl -DenyAddAndCustomizePages:$true

PS C:\Users\Vlad\code\m365-PowerShell>
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPAdaptiveScopeProperty -Key "SiteCustomSubject" -Value "Test4"
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPPropertyBagValue      -Key "SiteCustomSubject" -Value "Test5" -Indexed:$true
Set-PnPPropertyBagValue: Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED))


PS C:\Users\Vlad\code\m365-PowerShell> DisConnect-PnPOnline
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Connect-PnPOnline -Url $siteUrl -ClientId $ClientId -Tenant $tenantId -Interactive
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPPropertyBagValue      -Key "SiteCustomSubject" -Value "Test6" -Indexed:$true
Set-PnPPropertyBagValue: Unable to connect to the SharePoint Online Admin Center at 'https://s5dz3-admin.sharepoint.com' to run this cmdlet. If this URL is incorrect for your tenant, you can pass in the correct Admin Center URL using Connect-PnPOnline -TenantAdminUrl. If you are using Privileged Identity Management (PIM) on your tenant, please ensure you have activated at least the SharePoint Administrator role and allowed some time for it to activate. Error message: Attempted to perform an unauthorized operation.   
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPAdaptiveScopeProperty -Key "SiteCustomSubject" -Value "Test7"
Set-PnPAdaptiveScopeProperty: Unable to connect to the SharePoint Online Admin Center at 'https://s5dz3-admin.sharepoint.com' to run this cmdlet. If this URL is incorrect for your tenant, you can pass in the correct Admin Center URL using Connect-PnPOnline -TenantAdminUrl. If you are using Privileged Identity Management (PIM) on your tenant, please ensure you have activated at least the SharePoint Administrator role and allowed some time for it to activate. Error message: Attempted to perform an unauthorized operation.


PS C:\Users\Vlad\code\m365-PowerShell> Get-PnPTenant -Connection $connectionAdmin | Select-Object AllowWebPropertyBagUpdateWhenDenyAddAndCustomizePagesIsEnabled

AllowWebPropertyBagUpdateWhenDenyAddAndCustomizePagesIsEnabled
--------------------------------------------------------------
                                                         False


Set-PnPTenant -Connection $connectionAdmin -AllowWebPropertyBagUpdateWhenDenyAddAndCustomizePagesIsEnabled $true


PS C:\Users\Vlad\code\m365-PowerShell> Get-PnPTenant -Connection $connectionAdmin | Select-Object AllowWebPropertyBagUpdateWhenDenyAddAndCustomizePagesIsEnabled

AllowWebPropertyBagUpdateWhenDenyAddAndCustomizePagesIsEnabled
--------------------------------------------------------------
                                                          True



PS C:\Users\Vlad\code\m365-PowerShell>
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPAdaptiveScopeProperty -Key "SiteCustomSubject" -Value "Test7"
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPPropertyBagValue      -Key "SiteCustomSubject" -Value "Test6" -Indexed:$true
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Get-PnPPropertyBag

Key                              Value
---                              -----
SiteCustomSubject                Test6




PS C:\Users\Vlad\code\m365-PowerShell>
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPPropertyBagValue      -Key "SiteCustomSubject" -Value "Test9" -Indexed:$true

Confirm
The current site is a no-script site. Do you want to temporarily enable scripting on it to allow setting property bag value?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Get-PnPPropertyBag

Key                              Value
---                              -----
vti_associategroups              5;4;3
FollowLinkEnabled                TRUE
vti_associateownergroup          3
vti_indexedpropertykeys          VABoAGUAbQBlAFAAcgBpAG0AYQByAHkA|UwBpAHQAZQBDAHUAcwB0AG8AbQBQAHIAbwBwAGUAcgB0AHkA|UwBpAHQAZQBDAHUAcwB0AG8AbQBTAHUAYgBqAGUAYwB0AA==|
disabledhelpcollections
vti_approvallevels               Approved Rejected Pending\ Review
vti_defaultlanguage              en-us
taxonomyhiddenlist               4cc93225-d917-4b8e-9139-a92412a1d1cb
enabledhelpcollections           VGSEndUser
vti_sitemasterid                 d2c702b5-65a4-4311-929e-f5dd2daf612b
_VTI_ACCESSREQUESTSLISTID        2ce14573-1f0f-4ade-ab04-ceafed60c01c,Access Requests
_VTI_PENDINGREQUESTSVIEWID       /sites/tst/Access Requests/pendingreq.aspx
contenttypessynctimestampversion 1
ThemePrimary                     #A4262C
UseFastCloneFromSiteMaster       FALSE
vti_associatevisitorgroup        4
vti_extenderversion              16.0.0.25110
vti_associatemembergroup         5
contenttypesusagebackfillversion 3
_VTI_SHARINGLINKSLISTID          280ba1fc-73cb-4d51-9e83-f71971918afc,Sharing Links
profileschemaversion             1
vti_categories                   Travel Expense\ Report Business Competition Goals/Objectives Ideas Miscellaneous Waiting VIP In\ Process Planning Schedule
vti_createdassociategroups       3;4;5
SiteCustomProperty               TestSite
__AllowExternalEmbedding         1
SiteCustomSubject                Test9


PS C:\Users\Vlad\code\m365-PowerShell> 
PS C:\Users\Vlad\code\m365-PowerShell> Set-PnPPropertyBagValue      -Key "SiteCustomSubject" -Value "Test9" -Indexed:$true
Set-PnPPropertyBagValue: Unable to connect to the SharePoint Online Admin Center at 'https://s5dz3-admin.sharepoint.com' to run this cmdlet. If this URL is incorrect for your tenant, you can pass in the correct Admin Center URL using Connect-PnPOnline -TenantAdminUrl. If you are using Privileged Identity Management (PIM) on your tenant, please ensure you have activated at least the SharePoint Administrator role and allowed some time for it to activate. Error message: Attempted to perform an unauthorized operation.   

PS C:\Users\Vlad\code\m365-PowerShell> 

