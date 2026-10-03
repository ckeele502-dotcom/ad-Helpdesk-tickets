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

![Locked out message](ticket01-symptom.png)
![Unlocking the account in AD](ticket01-resolution.png)
![Successful login after unlock](ticket01-resolution2.png)

---

## Ticket #2 — Password Reset

Emily Johnson (Sales) forgot her password, so I went into Active Directory and reset it 
for her. Set a temporary password and checked the box so she'd have to change it herself 
on next login — didn't want to just hand her a permanent password.

Logged in as her on the client to test it out, got the "you must change your password" 
prompt like expected, set a new one, and confirmed I could log in fine afterward.

![Password reset in AD](ticket02-symptom.png)
![Password reset applied](ticket02-resolution1.png)
![Password change confirmation](ticket02-resolution2.png)
![Successful login with new password](ticket02-resolution3.png)

---

## Ticket #3 — Access Denied on Shared Drive

Michael Williams (Sales) reported he couldn't get into the shared Sales folder anymore. 
Checked his group memberships in Active Directory and found he'd somehow been removed 
from SG-Sales — he was only showing up under the default Domain Users group.

Added him back to SG-Sales and had him try the folder again. It opened right up after 
that, so the fix worked.

![Access denied error](ticket03-symptom.png)
![Diagnosis and fix in PowerShell](ticket03-resolution.png)
![Successful access after fix](ticket03-resolution2.png)

---

## Ticket #4 — New Hire Onboarding

Got a request to set up a new employee, Sarah Lee, starting in Finance. Created her 
account in Active Directory with PowerShell, dropped her straight into the Finance OU, 
and added her to SG-Finance so she'd have access to the shared drive from day one.

Logged in as her on a client to test everything end to end — had to change the temp 
password like expected, then confirmed she could get into the Finance shared folder 
right away with no extra steps needed.

![Account created and added to group](ticket04-setup.png)
![Successful access to Finance folder](ticket04-resolution.png)

---

## Ticket #5 — Employee Offboarding

James Rodriguez left the company, so I needed to shut down his access without deleting 
the account outright (better to keep it around for records/audit purposes instead of 
wiping it). Disabled his account, pulled him out of SG-Sales so he'd lose folder access, 
and moved him into the Disabled Users OU to keep things organized.

Double checked afterward that everything took — account shows disabled and sitting in 
the right OU now.

![Offboarding steps and confirmation](ticket05-resolution.png)

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

![Broken security filtering](ticket06-broken-filtering.png)
![gpresult showing it filtered out](ticket06-diagnosis.png)
![Corrected filtering with Read/Apply split](ticket06-fix.png)
![gpresult confirming it applied](ticket06-resolution.png)

---

## Ticket #7 — Client Cannot Access Network Shares (DNS Resolution Failure)

John Smith called in saying he could no longer open the shared department drives, and a 
couple of internal sites wouldn't load either. Said everything was fine the day before 
and he hadn't changed anything. Had him ping by IP first to rule out a basic connectivity 
issue — that came back fine, which pointed at DNS specifically rather than the network 
itself.

Remoted in and reproduced the error in File Explorer, then checked his adapter's DNS 
settings in PowerShell. Found it was pointed at `192.168.1.250` instead of the actual 
domain controller — nslookup against the domain came back with no response, confirming 
it couldn't resolve anything.

Repointed the adapter to the real DC address and re-ran nslookup, which resolved 
correctly. Went back into File Explorer and all the department shares loaded normally 
again.

![File Explorer error reproducing the issue](ticket07-symptom.png)
![DNS pointed at the wrong address, nslookup failing](ticket07-diagnosis1.png)
![DNS corrected, nslookup resolving successfully](ticket07-resolution1.png)
![Department shares loading normally again](ticket07-resolution2.png)

---

## Ticket #8 — Client Has No Network/Internet Access (DHCP Failure)

John Smith's machine suddenly showed "No internet access" and couldn't reach any shared 
drives. He mentioned a coworker on the same network was fine, which at first pointed at 
something specific to his machine — that turned out not to be the whole story.

Confirmed DHCP was healthy to start, then stopped the DHCP Server service on the domain 
controller to simulate the failure. Back on the client, tried to renew the lease and it 
fell back to an APIPA address (169.254.x.x) with no default gateway — a clear sign the 
machine couldn't reach a DHCP server at all. That lines up with "no internet, no shares" 
even though basic hardware/networking was fine.

Restarted the DHCP Server service on the DC, confirmed it came back up, then had the 
client renew again — pulled a normal address immediately and everything worked.

![DHCP Server running normally (baseline)](ticket08-baseline.png)
![DHCP Server service stopped to simulate the failure](ticket08-diagnosis2.png)
![Client falling back to APIPA with no gateway](ticket08-diagnosis3.png)
![DHCP Server service restarted](ticket08-resolution1.png)
![Client renewing successfully, valid IP restored](ticket08-resolution2.png)

---

## Ticket #9 — Mapped Network Drive Inaccessible (Sales Share)

John Smith reported his mapped S: drive was throwing an error and he couldn't get to any 
of his files, even though it had been working fine the day before. Confirmed the drive 
was still mapped correctly on his end, but opening it gave a network error.

Checked the share directly on the file server with `Get-SmbShare` and got nothing back — 
the Sales share had been removed entirely, even though the actual folder and data were 
still sitting there untouched. Most likely someone unshared it by accident while cleaning 
up a neighboring folder.

Re-shared the folder from the server, confirmed it showed up again with `Get-SmbShare`, 
then ran `Test-Path` on the client — came back True. Opened the S: drive again and it 
loaded normally, no error.

![S: drive working normally (baseline)](ticket09-baseline.png)
![Client-side network error](ticket09-symptom.png)
![Share restored on the server](ticket09-resolution1.png)
![Test-Path returning True, drive opening normally](ticket09-resolution2.png)

---

## Ticket #10 — Computer Running Extremely Slow

John Smith called in saying his computer had gotten extremely slow over the last hour — 
apps taking forever to open, everything laggy. He hadn't installed anything new or 
changed any settings.

Opened Task Manager to get actual numbers instead of going off the description alone. 
Overall CPU was sitting at 57%, and sorting the Processes tab by CPU showed a Windows 
PowerShell process eating 44.5% on its own. Checked the Performance tab to make sure it 
wasn't just a brief spike — it was sustained around 48%, and since the machine only has 
two logical processors, that one process was basically pinning an entire core by itself.

No errors tied to it, it was just stuck in a loop and never giving control back. Ended 
the task directly from Task Manager, and CPU usage dropped from 57% down to 3% 
immediately. Had the user open a few apps again afterward — everything responded 
normally.

![Task Manager showing the runaway PowerShell process](ticket10-diagnosis1.png)
![Performance tab confirming sustained CPU load](ticket10-diagnosis2.png)
![CPU usage back to normal after ending the process](ticket10-resolution.png)
