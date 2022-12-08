# Step 1. Enable WSL on Windows

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

## Step 2. Install Linux Distro

```
curl.exe -L -o ubuntu-2004.appx https://aka.ms/wslubuntu2004
Rename-Item ubuntu-2004.appx ubuntu-2004.zip
Expand-Archive ubuntu-2004.zip ubuntu2004
```

## Step 3. Set the Path Environment Variable

```
$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")[System.Environment]::SetEnvironmentVariable("PATH", $userenv + "C:\Users\Administrator\ubuntu2004", "User")
```

## Step 4. Enable SSH on Windows

```
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service -Name ssh-agent -StartupType 'Automatic'
Set-Service -Name sshd -StartupType 'Automatic'
Get-Service ssh* | Start-Service
```

## Step 5. Set default SSH Shell

```
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\WINDOWS\System32\bash.exe" -PropertyType String -Force
```
