Use Case 1 (Pentest): Run Basic Checks Only

🎯 Objective: I want to know if there is any obvious way I can escalate my privileges.
```powershell
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
```
Use Case 2 (Research): Run Extended Checks + Write Human-Readable Reports

🎯 Objective: There is no obvious vulnerability, but I want to dig a little further (and potentially find an 0-day in some third-party software for instance).
```powershell
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML"
```                                                                                     
Use Case 3 (Audit): Run All Checks + Write All Reports

🎯 Objective: I want to further scan the machine in case there are configuration issues that are not covered by common security standards, and optionally feed the results into an automated reporting tool.
```powershell
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended -Audit -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML,CSV,XML"
```
