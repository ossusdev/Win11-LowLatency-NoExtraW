# Win11-LowLatency-NoExtraW

**Safe PowerShell tweaks for Windows 11** that improve input latency, frame pacing, and 1%/0.1% lows **without increasing power consumption**. Fully reversible.

---

## How to Use

1. Open **PowerShell as Administrator**  
2. Copy the entire script from this repo  
3. Paste into PowerShell and press **Enter**  
4. **Reboot** to apply changes  

---

# ==================================================
# Windows 11 Low-Latency + 1% Low Improvement Script
# Safe: Does NOT increase power consumption
# Improves responsiveness, scheduler, and GPU frame pacing
# Tested on modern AMD CPUs/GPUs
# ==================================================

Write-Host "Applying low-latency optimizations (no extra watts)..." -ForegroundColor Cyan

# --------------------------------------------------
# 1. Create System Restore Point
# --------------------------------------------------
Checkpoint-Computer -Description "LowLatency_NoExtraW" -RestorePointType MODIFY_SETTINGS

# --------------------------------------------------
# 2. Set Balanced power plan (efficiency-friendly)
# --------------------------------------------------
powercfg -setactive SCHEME_BALANCED

# Adjust CPU boost ramp without forcing max clocks
powercfg -setacvalueindex SCHEME_BALANCED SUB_PROCESSOR PROCTHROTTLEMIN 20
powercfg -setacvalueindex SCHEME_BALANCED SUB_PROCESSOR PROCTHROTTLEMAX 100
powercfg -setactive SCHEME_BALANCED

# --------------------------------------------------
# 3. Improve foreground scheduling (reduces latency)
# --------------------------------------------------
Set-ItemProperty `
 -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" `
 -Name "Win32PrioritySeparation" `
 -Type DWord `
 -Value 26 `
 -Force

# --------------------------------------------------
# 4. Enable Game Mode (only if the key exists)
# --------------------------------------------------
$gameBarKey = "HKLM:\SOFTWARE\Microsoft\GameBar"
if (Test-Path $gameBarKey) {
    Set-ItemProperty `
        -Path $gameBarKey `
        -Name "AllowAutoGameMode" `
        -Value 1 `
        -Type DWord `
        -Force
    Write-Host "Game Mode enabled via registry." -ForegroundColor Cyan
} else {
    Write-Host "GameBar registry key not found. Skipping Game Mode tweak." -ForegroundColor Yellow
}

# --------------------------------------------------
# 5. Enable Hardware Accelerated GPU Scheduling
# --------------------------------------------------
Set-ItemProperty `
 -Path "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" `
 -Name "HwSchMode" `
 -Type DWord `
 -Value 2 `
 -Force

# --------------------------------------------------
# 6. Reduce UI animations & latency
# --------------------------------------------------
Set-ItemProperty `
 -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" `
 -Name "VisualFXSetting" `
 -Value 2 `
 -Type DWord `
 -Force

Set-ItemProperty `
 -Path "HKCU:\Control Panel\Desktop" `
 -Name "MenuShowDelay" `
 -Value "0"

Set-ItemProperty `
 -Path "HKCU:\Control Panel\Desktop\WindowMetrics" `
 -Name "MinAnimate" `
 -Value "0"

# --------------------------------------------------
# 7. Disable safe background services that cause stutter
# --------------------------------------------------
$services = @(
  "SysMain",     # memory/disk bursts
  "DiagTrack"    # telemetry jitter
)

foreach ($svc in $services) {
    $s = Get-Service -Name $svc -ErrorAction SilentlyContinue
    if ($s) {
        Stop-Service $svc -Force -ErrorAction SilentlyContinue
        Set-Service $svc -StartupType Disabled
    }
}

# --------------------------------------------------
# 8. Prevent background app wakeups
# --------------------------------------------------
Set-ItemProperty `
 -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" `
 -Name "GlobalUserDisabled" `
 -Value 1 `
 -Type DWord `
 -Force

# --------------------------------------------------
# 9. Storage latency safeguard
# --------------------------------------------------
fsutil behavior set DisableDeleteNotify 0 | Out-Null

# --------------------------------------------------
# Done
# --------------------------------------------------
Write-Host "Low-latency optimizations applied." -ForegroundColor Green
Write-Host "No extra power draw added." -ForegroundColor Green
Write-Host "Reboot recommended." -ForegroundColor Yellow

