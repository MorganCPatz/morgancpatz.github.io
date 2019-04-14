---
layout: post
section-type: post
title: Group Policy Delegations - Quick, Easy, Effective.
category: Group Policy
---

When it comes to Group Policy and your environment, it's important to restrict who can create/modify/delete/etc. Group Policy Objects.  Additionally it's important to control and restrict who has permissions to link/un-link/block inheritance/etc. on Organizational Units.  I've seen environments where GPOs are typically maintained by one of two groups: Domain Admins or Group Policy Creator Owners.  

<b>Note:</b> Group Policy Creator Owners - Allows members to create GPOs and edit them.  However, this group must be delegated to allow it to link to OUs.  While this could work, if you need to delegate to more than one team, you will need to use a separate group to keep the teams from being able to edit each other's GPOs.    

Personally, I'd avoid using "Group Policy Creator Owners" for any sort of delegation.  In my opinion, it's better to use purpose-built groups for delegations and avoid using any of the Builtin\Groups where possible.  Domain Admins, of course don't always have to be used - in fact, its use and the membership for this group should be pretty limited.  Thankfully, we have a way to delegate permissions to minimize the use of Domain Admins as well as the means to allow non-DA accounts to work in Group Policy.  This is all relatively straight forward but in the end provides a level of control if you do not have something like [Advanced Group Policy Management](https://docs.microsoft.com/en-us/microsoft-desktop-optimization-pack/agpm/).  

Since not all environments have access to AGPM, I wanted to look into a way to delegate permissions to groups and grant them the ability to manage a subset of GPOs and OUs.  For example, grant the Server Team the ability to manage their own GPOs/OUs as well as grant the Desktop Team the ability to manage theirs - delegate both groups without the use of Domain Admins or Group Policy Creator Owners.  Keep in mind, this post only covers the delegation aspect and by no means is equivalent to everything AGPM can do - just delegation.  With that said, let's get into the delegation aspect.

In my test environment, the default groups that have delegation rights are:  Administrators, Domain Admins, Enterprise Admins, and System.  All my OUs have Inheritance Enabled, so permissions delegated at the top would be inherited.  

![Group-Policy-Management-Domain-Delegation]({{ "/img/2019-04-14-GPO-Delegations/GPM1.JPG" | absolute_url }})<br>
![Group-Policy-Management-Servers-Delegation]({{ "/img/2019-04-14-GPO-Delegations/GPM2.JPG" | absolute_url }})

If you're a small environment or only have one team that manages Group Policy, it may be tempting to delegate at permissions at the Domain Level.  The issue with this is, if Inheritance is Enabled on all your Level One or Sub OUs, they will inherit these permissions.  This is problematic for example, because of the "Domain Controllers" OU or any other OUs that contain high level devices/accounts. If you control the OU (ability to link to/etc.) you control the objects in it - same if you control the GPO that links to said OUs.  Delegating at the top for a lower level administrator account (non DA/EA/SA), would grant them the ability to control devices/accounts above this account.  Please do not do this.  Domain Admins should be the only ones who can manage the root, Domain Controllers, and any other Organizational Units that contain high level objects, such as DA/EA accounts.   

For this post, we're going to delegate the group "MCP_GPO_Admins" to our "Servers" OU with the following permissions:  Generate RSOP - Planning, Generate RSOP - Logging, GP-Link, and GP-Options.  All of this can be done thru the Delegation Wizard in Active Directory Users and Computers but if you have to do this multiple times, it's a bit much.  For this, I'm going to use a function I created, [Set-OUGPODelegations](https://github.com/MorganCPatz/PSFunctions/blob/master/Functions/Set-OUGPODelegations.ps1).

![Set-OUGPODelegations]({{ "/img/2019-04-14-GPO-Delegations/Set-OUGPODelegations.JPG" | absolute_url }})

As seen above, we set the permissions for the group "MCP_GPO_Admins" on the "Servers" OU.  This was verified by running a second PowerShell script that shows the ACLs for a group on an OU.  The permissions are set and verified in PowerShell but we can verify the same using ADUC.  

![Active-Directory-Users-and-Computers]({{ "/img/2019-04-14-GPO-Delegations/ADUC.JPG" | absolute_url }})

Setting the permissions on an OU is only the first step in our Group Policy Delegation process.  In Group Policy Management (running as an account that is a member of "MCP_GPO_Admins"), we see on the Delegation tab for the "Servers" OU, that our delegated group has permissions.  Right-clicking on the "Servers" OU, we see that this account has the ability to Link/Block Inheritance/etc., everything except "Create a GPO in this domain and Link it here."  

![Group-Policy-Management-Servers-RC]({{ "/img/2019-04-14-GPO-Delegations/GPM3.JPG" | absolute_url }})

This group currently does not have any Delegations set on "Group Policy Objects" that would allow it create GPOs.  As seen below in the screenshot, the default Delegations are for:  Domain Admins, Group Policy Creator Owners, and System.  Adding "MCP_GPO_Admins" here would allow it the ability to create Group Policy Objects - this is the second step of our Group Policy Delegation.  

![Group-Policy-Management-GPO]({{ "/img/2019-04-14-GPO-Delegations/GPM4.JPG" | absolute_url }})<br>
![Group-Policy-Management-GPO-New]({{ "/img/2019-04-14-GPO-Delegations/GPM5.JPG" | absolute_url }})

With those two delegations set - On the OU and on "Group Policy Objects" members of the group can now create and link GPOs to the "Servers" OU.  What about existing GPOs?  We'll need to set delegations for those.  We can do that using the PowerShell cmd:<br>
`Get-GPO -All | Where {$_.Displayname -like "Servers*"} | Set-GPPermission -PermissionLevel GPOEditDeleteModifySecurity -TargetName 'MCP_GPO_Admins' -TargetType Group`

This command will set for all GPOs that start with "Servers -" and then add permissions for the group to have the ability to Edit/Delete/Modify Security settings for those GPOs.  If you follow a naming scheme, such as all GPOs affecting servers start with "Servers - ," workstations start with "Workstations - ," etc., this cmd will make things soo much easier.  

![Get-GPO]({{ "/img/2019-04-14-GPO-Delegations/Get-GPO.JPG" | absolute_url }})

We can run a cmd to see all GPOs that "MCP_GPO_Admins" have permissions on:  
`Get-GPO -All | ForEachObject { if ($_ | Get-GPPermissions -Target 'MCPLAB\MCP_GPO_Admins' -TargetType Group -ErrorAction SilentlyContinue) {$_.DisplayName}}`  

Quick and easy way to verify which GPOs a group has permissions to.  Setting permissions on already existing GPOs (if you so desire) is the third and final part of our Group Policy delegation.  As seen below, the account now has the ability to Edit the GPO "Servers - Default Local Group Membership" due to the permissions we just set above.

![Group-Policy-Management-Edit]({{ "/img/2019-04-14-GPO-Delegations/GPM6.JPG" | absolute_url }})

If you do not have access to AGPM, I highly recommend taking the time to delegate permissions for Group Policy.   Whether a small or large environment, this is something in my opinion that should be done from a security perspective.  Its tempting to just use a DA Account or add accounts to the "Group Policy Creator Owners" group but please don't.   Likewise, as previously mentioned, take the time to ensure that when setting up delegations - lower level administrator accounts do not have the ability to affect devices/accounts above it.  This is all but yet another (in my opinion) important step to securing an environment.  Build the Foundation - Stack the Layers.  Defense in Depth. 