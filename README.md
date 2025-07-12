
## 📦 Installer untuk Windows (`install.ps1`)

```powershell
<#
.SYNOPSIS
    ZCode Windows Installer
#>
$ErrorActionPreference = "Stop"

Write-Host "=== ZCode Installer for Windows ===" -ForegroundColor Cyan

# Check admin rights
if (-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    Write-Host "Harap jalankan sebagai Administrator" -ForegroundColor Red
    Exit 1
}

# Download binary
$versions = Invoke-RestMethod -Uri "https://api.github.com/repos/zcode-lang/zcode/releases/latest"
$asset = $versions.assets | Where-Object { $_.name -match "zcode-windows-x64\.zip" }

$tempDir = "$env:TEMP\zcode-install"
New-Item -ItemType Directory -Path $tempDir -Force

Write-Host "Mengunduh ZCode v$($versions.tag_name)..." -ForegroundColor Yellow
Invoke-WebRequest -Uri $asset.browser_download_url -OutFile "$tempDir\zcode.zip"

# Install
$installDir = "$env:ProgramFiles\ZCode"
Expand-Archive -Path "$tempDir\zcode.zip" -DestinationPath $installDir

# Add to PATH
$path = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($path -notlike "*$installDir*") {
    [Environment]::SetEnvironmentVariable("PATH", "$path;$installDir", "Machine")
}

# Install board support
Write-Host "Menginstal dukungan board..." -ForegroundColor Yellow
& "$installDir\zcode.exe" board install arduino
& "$installDir\zcode.exe" board install esp32

# Cleanup
Remove-Item -Recurse -Force $tempDir

Write-Host "`nInstalasi berhasil!`n" -ForegroundColor Green
Write-Host "Coba jalankan:`n  zcode new project`n  cd project`n  zcode build`n" -ForegroundColor Yellow
