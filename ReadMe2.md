# Helpdesk Ticket Log

This is a running log of simulated helpdesk tickets from my Active Directory homelab. 
Each one follows the same basic pattern: something breaks (usually on purpose, to create 
a realistic scenario), I diagnose it, fix it, and confirm the fix worked.

---

## Ticket #1 — Account Lockout

Wanted to test what happens when someone gets locked out, so I picked John Smith and just 
typed the wrong password on purpose a bunch of times until it locked. Once it did, I went 
into Active Directory and checked his account — sure enough, it showed as locked on the 
Account tab.

To fix it, I checked the "Unlock account" box and hit apply. Went back and logged in as 
him again with the right password just to make sure it actually worked, and it did.

![Locked out message](<screenshots/ticket01-symptom.png>)
![Locked account confirmed in AD](<screenshots/ticket01-diagnosis.png>)
![Successful login after unlock](<screenshots/ticket01-resolution.png>)

---

## Ticket #2 — Password Reset

Emily Johnson (Sales) forgot her password, so I went into Active Directory and reset it 
for her. Set a temporary password and checked the box so she'd have to change it herself 
on next login — didn't want to just hand her a permanent password.

Logged in as her on the client to test it out, got the "you must change your password" 
prompt like expected, set a new one, and confirmed I could log in fine afterward.

![Password reset in AD](<screenshots/ticket02-symptom.png>)
![Successful login with new password](<screenshots/ticket02-resolution.png>)

---

## Ticket #3 — Access Denied on Shared Drive

Michael Williams (Sales) reported he couldn't get into the shared Sales folder anymore. 
Checked his group memberships in Active Directory and found he'd somehow been removed 
from SG-Sales — he was only showing up under the default Domain Users group.

Added him back to SG-Sales and had him try the folder again. It opened right up after 
that, so the fix worked.

![Access denied error](<screenshots/ticket03-symptom.png>)
![Diagnosis and fix in PowerShell](<screenshots/ticket03-diagnosis-fix.png>)
![Successful access after fix](<screenshots/ticket03-resolution.png>)

---

## Ticket #4 — New Hire Onboarding

Got a request to set up a new employee, Sarah Lee, starting in Finance. Created her 
account in Active Directory with PowerShell, dropped her straight into the Finance OU, 
and added her to SG-Finance so she'd have access to the shared drive from day one.

Logged in as her on a client to test everything end to end — had to change the temp 
password like expected, then confirmed she could get into the Finance shared folder 
right away with no extra steps needed.

![Account created and added to group](<screenshots/ticket04-setup.png>)
![Successful access to Finance folder](<screenshots/ticket04-resolution.png>)

---

## Ticket #5 — Employee Offboarding

James Rodriguez left the company, so I needed to shut down his access without deleting 
the account outright (better to keep it around for records/audit purposes instead of 
wiping it). Disabled his account, pulled him out of SG-Sales so he'd lose folder access, 
and moved him into the Disabled Users OU to keep things organized.

Double checked afterward that everything took — account shows disabled and sitting in 
the right OU now.

![Offboarding steps and confirmation](<screenshots/ticket05-offboarding.png>)

---

## Ticket #6 — Group Policy Not Applying

Emily Johnson (Sales) said her mapped network drive wasn't showing up when she logged in. 
I checked the GPO responsible for mapping the Sales shared drive and found the Security 
Filtering had somehow gotten scoped to SG-IT instead of Sales — meaning it was only ever 
going to apply to IT staff, not her.

Fixed it by removing SG-IT and adding SG-Sales instead. Tested it with gpresult /r as her 
and found it was still being filtered out even with the right group added — turned out 
Authenticated Users needs Read access on the GPO for the computer to even retrieve it in 
the first place, separate from who it actually applies to. Added Authenticated Users back 
with Read-only permission (no Apply), kept SG-Sales as the only group with Apply Group 
Policy, and after that it worked — gpresult confirmed "Sales Mapped Drive" as applied, and 
the S: drive showed up correctly.

![Broken security filtering](<screenshots/ticket06-broken-filtering.png>)
![gpresult showing it filtered out](<screenshots/ticket06-diagnosis.png>)
![Corrected filtering with Read/Apply split](<screenshots/ticket06-fix.png>)
![gpresult confirming it applied](<screenshots/ticket06-resolution.png>)

---

*More tickets added as they're completed.*
