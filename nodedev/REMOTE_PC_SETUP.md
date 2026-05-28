# PC Setup Guide

Goal: connect node.dev on iPhone to a Windows PC over Tailscale in as few
steps as possible.

> **Note:** the setup script below is generated from the canonical copy in
> `lib/screens/onboarding/onboarding_constants.dart` (`pcSetupScript`). The
> PowerShell block here is kept byte-for-byte identical and is enforced by
> `test/onboarding_script_sync_test.dart`. Edit the constant, not this block.

## 1. Install Tailscale

Install Tailscale on both the Windows PC and iPhone, then sign in to the same
tailnet. Note the PC's MagicDNS name (e.g. `rog-pc`) or Tailscale IP
(`100.x.y.z`) — the setup script also prints this for you in step 2.

**Recommended:** in the Tailscale admin console under Access controls, allow
your phone to reach this PC on ports 22 (SSH) and 9922 (key push). If you use
a custom ACL, both ports must be listed — a restrictive ACL that only covers
port 22 will block the key delivery step in the app.

## 2. Run the PC setup script

In the node.dev app, copy the one-line command and paste it into PowerShell.
It pops a UAC prompt and relaunches itself as Administrator, then:

1. Installs and starts the Windows OpenSSH Server
2. Restricts SSH to Tailscale addresses (IPv4 + IPv6) via Windows Firewall
3. Hardens `sshd_config` (key-only auth, no passwords)
4. Prints the address to type into the app, then waits to receive your phone's
   public key automatically from node.dev
5. Verifies sshd is running

**Keep the PowerShell window open** — the script blocks at step 4 until it
receives the key from your phone (it times out after 10 minutes).

The full script (also available via **Share script** in the app):

```powershell
function Step($n, $msg) { Write-Host "[$n/5] $msg" -ForegroundColor Cyan }

# ── 0  Pre-flight: Tailscale present and signed in ───────────────────────────
$tsCmd = Get-Command tailscale.exe -ErrorAction SilentlyContinue
if (-not $tsCmd) {
  Write-Host 'WARNING: Tailscale not found. Install it and sign in before connecting.' -ForegroundColor Yellow
} else {
  & tailscale status 1>$null 2>$null
  if ($LASTEXITCODE -ne 0) {
    Write-Host 'WARNING: Tailscale is installed but not signed in. Run:  tailscale up' -ForegroundColor Yellow
  }
}

# ── 1  Install and start OpenSSH ─────────────────────────────────────────────
Step 1 'Installing OpenSSH Server (first run can take a few minutes)...'
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0 | Out-Null
ssh-keygen -A
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd

# ── 2  Restrict SSH to Tailscale addresses (IPv4 + IPv6) ─────────────────────
Step 2 'Locking SSH to your Tailscale network...'
$tsRanges = @('100.64.0.0/10', 'fd7a:115c:a1e0::/48')
$ruleName = 'OpenSSH-Server-Tailscale-In'
Get-NetFirewallRule -Direction Inbound -Enabled True -Action Allow -ErrorAction SilentlyContinue | ForEach-Object {
  $rule = $_
  $port = $rule | Get-NetFirewallPortFilter
  if ($rule.Name -ne $ruleName -and
      $port.Protocol -eq 'TCP' -and $port.LocalPort -eq '22') {
    $rule | Disable-NetFirewallRule
  }
}
Remove-NetFirewallRule -Name $ruleName -ErrorAction SilentlyContinue
New-NetFirewallRule `
  -Name $ruleName `
  -DisplayName 'OpenSSH Server (Tailscale only)' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow `
  -LocalPort 22 -RemoteAddress $tsRanges | Out-Null

# ── 3  Harden sshd_config (key-only auth, no passwords) ──────────────────────
Step 3 'Hardening SSH configuration...'
$config = 'C:\ProgramData\ssh\sshd_config'
$desired = @(
  'PubkeyAuthentication yes',
  'PasswordAuthentication no',
  'KbdInteractiveAuthentication no'
)
$raw = Get-Content $config
$lines = New-Object System.Collections.Generic.List[string]
foreach ($l in $raw) {
  if ($l -notmatch '^\s*#?\s*(PubkeyAuthentication|PasswordAuthentication|KbdInteractiveAuthentication)\s+') {
    $lines.Add($l)
  }
}
$mi = -1
for ($i = 0; $i -lt $lines.Count; $i++) {
  if ($lines[$i] -match '^\s*Match\s+') { $mi = $i; break }
}
$new = if ($mi -lt 0) { @($lines) + $desired }
       elseif ($mi -eq 0) { $desired + @($lines) }
       else { @($lines[0..($mi - 1)]) + $desired + @($lines[$mi..($lines.Count - 1)]) }
Set-Content -Path $config -Value $new -Encoding ascii
$sshdExe = "$env:SystemRoot\System32\OpenSSH\sshd.exe"
if (Test-Path $sshdExe) {
  & $sshdExe -t 2>$null
  if ($LASTEXITCODE -ne 0) {
    Write-Host 'WARNING: sshd_config failed validation; review C:\ProgramData\ssh\sshd_config.' -ForegroundColor Yellow
  }
}
Restart-Service sshd

# ── 4  Receive key from node.dev app (keep this window open) ─────────────────
Step 4 'Waiting for your phone key...'
New-NetFirewallRule -Name 'node.dev-keypush' `
  -DisplayName 'node.dev key push (temp)' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow `
  -LocalPort 9922 -RemoteAddress $tsRanges -ErrorAction SilentlyContinue | Out-Null
$isAdmin = ([Security.Principal.WindowsPrincipal] `
  [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole('Administrators')
$keyFile = if ($isAdmin) { 'C:\ProgramData\ssh\administrators_authorized_keys' } `
           else { "$env:USERPROFILE\.ssh\authorized_keys" }
# Create the file only if absent — never truncate existing authorized keys.
if (-not (Test-Path $keyFile)) { New-Item -ItemType File -Force $keyFile | Out-Null }

$wslInstalled = [bool](Get-Command wsl.exe -ErrorAction SilentlyContinue)
$tsIp = ''
if ($tsCmd) { $tsIp = (& tailscale ip -4 2>$null | Select-Object -First 1) }
$dnsName = [System.Net.Dns]::GetHostName()

Write-Host ''
Write-Host '--------------------------------------------------' -ForegroundColor DarkGray
Write-Host '  Ready. In the node.dev app, enter this address:' -ForegroundColor White
if ($tsIp) { Write-Host "    $tsIp" -ForegroundColor Green }
Write-Host "    (or MagicDNS name: $dnsName)" -ForegroundColor Green
Write-Host '  Keep this window open, then tap Push in the app.' -ForegroundColor DarkCyan
Write-Host '--------------------------------------------------' -ForegroundColor DarkGray

$listener = [System.Net.HttpListener]::new()
$listener.Prefixes.Add('http://+:9922/')
$listener.Start()

$key = $null
$deadline = (Get-Date).AddMinutes(10)
while (-not $key -and (Get-Date) -lt $deadline) {
  $task = $listener.GetContextAsync()
  while (-not $task.IsCompleted) {
    if (-not $task.AsyncWaitHandle.WaitOne(60000)) {
      if ((Get-Date) -ge $deadline) { break }
      Write-Host 'Still waiting for your key from the app...' -ForegroundColor DarkCyan
    }
  }
  if (-not $task.IsCompleted) { break }
  $ctx = $task.GetAwaiter().GetResult()
  $body = ([System.IO.StreamReader]::new($ctx.Request.InputStream)).ReadToEnd().Trim()
  if ($body -match '^(ssh-ed25519|ecdsa-sha2-\S+|ssh-rsa) ') {
    $info = @{
      username     = $env:USERNAME
      hostname     = $dnsName
      tailscaleIp  = "$tsIp"
      wslInstalled = $wslInstalled
    } | ConvertTo-Json -Compress
    $out = [Text.Encoding]::UTF8.GetBytes($info)
    $ctx.Response.StatusCode = 200
    $ctx.Response.ContentType = 'application/json'
    $ctx.Response.ContentLength64 = $out.Length
    $ctx.Response.OutputStream.Write($out, 0, $out.Length)
    $ctx.Response.Close()
    $key = $body
  } else {
    $msg = [Text.Encoding]::UTF8.GetBytes('Invalid key payload.')
    $ctx.Response.StatusCode = 400
    $ctx.Response.OutputStream.Write($msg, 0, $msg.Length)
    $ctx.Response.Close()
    Write-Host 'Ignored an invalid request on port 9922.' -ForegroundColor Yellow
  }
}
$listener.Stop()
Remove-NetFirewallRule -Name 'node.dev-keypush' -ErrorAction SilentlyContinue

if (-not $key) {
  Write-Host 'Timed out waiting for the key. Re-run this script and push again from the app.' -ForegroundColor Red
  return
}

# Append the key only if it is not already present (safe to re-run).
$existing = @()
if (Test-Path $keyFile) { $existing = @(Get-Content $keyFile -ErrorAction SilentlyContinue) }
if ($existing -notcontains $key) { Add-Content $keyFile $key }
if ($isAdmin) {
  icacls $keyFile /inheritance:r /grant 'Administrators:F' /grant 'SYSTEM:F' /grant 'NT SERVICE\sshd:R' | Out-Null
}
Write-Host 'Key saved.' -ForegroundColor Green

# ── 5  Verify ────────────────────────────────────────────────────────────────
Step 5 'Verifying...'
$svc = Get-Service sshd -ErrorAction SilentlyContinue
if ($svc -and $svc.Status -eq 'Running') {
  Write-Host 'Setup complete. SSH is running and ready.' -ForegroundColor Green
} else {
  Write-Host 'Setup finished, but sshd is not running. Try:  Start-Service sshd' -ForegroundColor Yellow
}
```

## 3. Pair your key in node.dev

The app generates your device key automatically. Enter the PC's Tailscale
hostname or IP (the address the script printed) and tap **Push key to PC**.
The app sends the public key over Tailscale to the waiting listener, and the
PC answers with its username and hostname so the next step is pre-filled.

Optionally tap **Verify connection** to open a real SSH session and pin the
host fingerprint before you finish.

If the push fails, copy the public key manually and paste it into the
`authorized_keys` file the script created.

## 4. Add your first host and connect

The PC's hostname and Windows username are pre-filled from the pairing step.
Confirm port `22` and tap save. SSH sessions land in PowerShell by default.

## Optional: WSL and tmux

WSL gives you a full Linux shell and tmux keeps sessions alive across
disconnects. Run these after the above steps:

```powershell
# Install WSL with Ubuntu — requires a reboot
wsl --install -d Ubuntu
```

```bash
# After reboot, launch Ubuntu from the Start menu.
# It will prompt you to create a Linux username and password.
# Once that's done, run:
sudo apt update && sudo apt install tmux -y
```

```powershell
# Enable WSL login via SSH — PowerShell (Admin) after WSL is ready
Set-Content -Path 'C:\ProgramData\ssh\sshd_config' -Value @'
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
ForceCommand wsl.exe

Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
'@ -Encoding ascii
Restart-Service sshd
```

After this, new SSH sessions drop directly into WSL.
