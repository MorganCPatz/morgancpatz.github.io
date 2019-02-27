---
layout: post
section-type: post
title: Separate Accounts for Admin Tasks
category: Security
---

When my interest in security started to take off, suddenly everything look good and I wanted to dabble in so many different areas.  In the beginning its often hard to decide where to start but even harder to stay focused once you started when there are so many changes waiting to be made.  This is where you should take a step back, survey the situation, and develop a plan.  The plan if you're not already there, should be about building a solid foundation so you can stack the layers.  Keep it simple and start small.  

For me, this would be looking at IT accounts and asking:  
-Are there shared admin credentials or does everyone (in IT) have their own account?
-Are there separate accounts for admin tasks or are day to day accounts used for everything?
-Does everyone need the same permissions/privileges?  Does everyone need access to everything?

This post will focus on answering these questions by discussing the creation of Separate Accounts for Admin Tasks.  To begin, hopefully there is not a single admin account that is used for everything or is being shared.  Everyone in IT who needs an Admin Account should have their own separate account - separate from each other and separate from their day-to-day account.  We want to create that separation of not only between individuals but between accounts as well.  Meaning, you should have at least two accounts: a Standard User and an Administrator account.  Typically not everyone will need the same permissions and privileges, nor will they need access to everything.  Environments vary for multiple reasons such as:  department size, individual roles, culture, etc.  Creating that separation where possible and where it makes sense will help strengthen your security overall.  Does the Help Desk Team need the same access as the Server Team?  I'd certainly hope not.  So let's talk about account separation.

I mentioned you should have at least two accounts - standard user account for day to day activities and an administrator account for admin related tasks.  This can be further divided up and additional accounts can be created for specific tasks, permissions, privileges, etc. but two is the absolute bare minimum.  Depending on your environment, this could result in each person in IT having 3, 4, or even 5+ accounts.  This largely depends on the roles within IT and the access needed.  Personally I like the idea of 4x different types of accounts:  Standard User, Administrator, Workstation Administrator, and Enterprise/Domain Admin.

<b>Standard User Account</b>:<br><br>
-Account used for day to day activities, such as checking email, internet access, etc.<br><br>
-This is a non-privileged account

<b>Administrator Account</b>:<br><br>
-Main Administrator Account - This varies but can be anything from Active Directory/Group Policy to Server Administrator<br><br>
-This account has more permissions/privileges than standard user account<br><br>
-Should not be used to log into or administer workstations

<b>Workstation Administrator</b>:<br><br>
-Account specifically used for administering workstations and has Local\Admin privileges on workstations<br><br>
-Should not be used to log into workstations and should not be used as a day to day account

<b>Enterprise/Domain Admin Account</b>:<br><br>
-Account used solely for Domain Administration - IE on Domain Controllers only<br><br>
-Should not be used on workstations or servers, just on Domain Controllers

Who gets what account depends largely on your environment, not everyone will need all 4x accounts.  Some will only need the standard user account and nothing else.  The point is, create separate accounts for separate tasks.  This helps to define roles, permissions, and privileges, and most importantly eliminates that single all powerful account.  If one of these accounts is compromised, then yes, bad things can still happen.  Regardless, care and effort should be taken to help secure these accounts. 

For example, this can be achieved by doing things such as:<br>
-Creating and Setting Fine Grained Password Policies<br>
-Setting User Rights Assignments > Deny Assignments to prevent certain accounts from certain actions<br>
-Utilizing the accounts only for their intended purposes-Enabling UAC on Workstations and Servers<br>
-Ensuring IT staff only have the accounts they need - Help Desk does not need an EA/DA Account.

There are many things that can be done to help secure these accounts; the above is just a general idea of some steps that can be taken.  Separation of accounts and creating separate accounts for admin tasks is about using the right tools - the correct purpose built account, for the right situation.  EA/DA accounts should never touch the workstation, likewise a day to day to account should not have local admin privileges.  I've been in environments where a single Domain Admin account was used everywhere by multiple individuals - it should be obvious why this is bad for multiple reasons.  Likewise, I've seen environments with account separation but using only two types of accounts and everyone had an EA/DA account...not much better.  The days of a single account or the same permissions/privileges for everyone are over and should be long gone by now. Unfortunately, that isn't the case for some environments but your environment doesn't have to be like that.  This is all but a small yet important piece to helping secure an environment.  No single idea will protect the environment but multiple ideas or layers will help.  Build the Foundation - Stack the Layers.  Defense in Depth.  


