---
layout: post
section-type: post
title: Fine-Grained Password Policies - Create, Use, Enjoy!
category: Active Directory
---

Starting with Server 2008, administrators have the ability to create different password and lockout settings for specific users or groups through the use of Fine-Grained Password Policies (FGPP). This is particularly useful when you want to add specific requirements and an extra layer of protection for your privileged IT Accounts.  Privileged IT Accounts in this instance would apply to:  Administrator, Workstation Administrator, and Enterprise/Domain Admin Accounts.  Basically anything other than your standard user account.  In previous and in my current environment, I've found that the configured Password Policy is a bit insufficient for use on said accounts.  FGPP are useful because you can configure multiple policies, each with their own unique set of requirements for different accounts or create just a single policy that is applied to multiple accounts. 

Requirements or settings you can configure are:  password length, age, history, lockout policy, and complexity requirements - all the same options you can configure in the Default Domain Policy plus precedence.  Precedence as you can probably already guess, is used to determine which policy will be applied when a user is either assigned multiple policies directly or is a member of separate groups that each have a policy applied.  The policy with the lower precedence (#) wins and will be applied to the user.  If you're setting FGPP for IT Privileged Accounts - precedence most likely will not apply since these accounts should be in separate groups that either share the same FGPP or have their own unique policy configured.  This is especially true if these groups are used for any delegations and not just FGPP - do not intermix accounts.  Keep accounts and groups separate where it makes sense .  My personal preference is to create a group for each type of IT Privileged Account, for example:

* IT_Admin:  Contains Administrator accounts
* IT_WSA:   Contains Workstation Administrator accounts
* IT_DA:  Contains Domain Admin accounts

The group and names used above are just an example - use a naming convention that fits your environment.  The idea is to create purpose built groups that reflect the type and purpose of the accounts it contains.  From here, we can either create a single policy and apply to all groups or create a unique policy for each group.  Applying a FGPP to each of these groups creates our "base" password settings for these accounts - if you need a more or less restrictive for a specific account then I would recommend the following:  

* Create a new group for the specific account(s)
* Create a new FGPP and configure password settings as desired
* Apply the FGPP to the group and set the applicable Precedence

Now that's FGPP has been briefly explained as well as  the idea and purpose behinds groups, let's create a Fine-Grained Password Policy. There are a couple of ways you can create a FGPP:  Using the GUI or through PowerShell.  I will cover both.


**GUI:**

* Launch Active Directory Administrative Center
* On the left navigate to domain > System > Password Settings Container
* On the right > Click New > Password Settings:  This will bring up the Wizard to create a FGPP
* Configure desired settings:  Length/History/Complexity/Age/Lockout Policy
* Add Groups the FGPP will Directly Apply To:<br>

![Active-Directory-Administration-Center]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/ADAC.jpg" | absolute_url }})<br>
![Create-FGPP]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/Create-FGPP.jpg" | absolute_url }})

**Default Settings when creating a FGPP**
![Default-FGPP]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/Default-FGPP.jpg" | absolute_url }})

Using the GUI to create and configure a FGPP is pretty straight-forward.  Any of the settings or configurations shown above can just as easily be configured in PowerShell.<br>

To create a FGPP using PowerShell - there are a couple of ways you can do this:  On the Domain Controller itself (Run PowerShell as Administrator), Remotely (PS-Session), or through Windows Admin Center.   My personal preference would be to use Windows Admin Center.  If you haven't heard of it or used it yet, I recommend taking a look at [Windows Admin Center](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/overview).

**PowerShell**

As previously mentioned, any settings we can configure using the GUI we can set using PowerShell, it's just a matter of using the correct parameters.<br>
For this, we're going to use the following command:  
`New-ADFineGrainedPasswordPolicy`

For Cmd Help and Syntax see below:
![Get-Help]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/Get-Help.JPG" | absolute_url }})<br>

All the settings and parameters used should be pretty straight forward - the areas we want to to pay attention to and the format used, is for the following parameters:
* LockoutDuration
* LockoutObservation
* MaxPasswordAge
* MinpasswordAge

The format to set these is:  Days:Hours:Minutes:Seconds:Fractions of a second.  
In the screenshot below, I created a FGPP named "FGPP - IT WSA"
![WAC-FGPP]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/WAC-FGPP.JPG" | absolute_url }})<br>

Everything above should make sense - there are a couple of settings we did not configure.  Namely, we didn't specify who the policy Applies To. This is easy enough to set with the following cmd:<br>
`Add-ADFineGrainedPasswordPolicy -Identity 'FGPP - IT WSA' -Subjects 'IT_WSA'`<br><br>
This will add apply the FGPP to our group for Workstation Administrators - "IT_WSA"
![WAC-ADD-FGPP]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/WAC-ADD-FGPP.JPG" | absolute_url }})

Looking at our FGPP in the Active Directory Administrative Center , we can confirm that what we see in PowerShell matches what we see in the GUI.
![GUI-FGPP]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/GUI-FGPP.JPG" | absolute_url }})

One setting that is not set, is "Protect from accidental deletion."  While we did not specify this setting in our PowerShell cmd, it is by default, automatically selected when creating a FGPP via ADAC.  Otherwise, everything else not confiugred with PowerShell, will be set using the default values. When using PowerShell, make sure to specify the parameters for any settings you do not wish to use the default values.  Once created, verify your settings!  

Lastly, we will want to confirm that our Password Policy Object (PSO) is applied to an account. <br>We can check this using the `Get-ADUserResultantPassword Policy`<br>
![Get-UserPP]({{ "/img/2019-03-13-Fine-Grained-Password-Policies/Get-UserPP.JPG" | absolute_url }})

One important thing to note: Once a FGPP is applied to an account, the account does not immediately require a password change if it does not meet the current criteria (MinPasswordLength).  Users will continue to use their current password settings until their password reaches its expiration or is changed.  FGPP are not password filters but simply provide a way to apply different settings (other than those in the Default Domain Policy) to specific users or groups.  Creating a FGPP is a simple and straight forward process - it doesn't matter if you do via the GUI or in PowerShell.  Use Fine-Grained Password Policies are a means to ensure privileged accounts are using appropriate password lengths.  Build the Foundation - Stack the Layers.  Defense in Depth.<br>
