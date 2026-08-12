# Agent Instructions

## Project
Need4Admin — cross-platform PowerShell auditing tool that scans privileged users in Microsoft
Entra ID and Azure subscriptions and produces HTML + CSV reports. Fork of Vlad Johansen's
Windows-only original, extended with macOS and Linux support (version 1.0-pesip.1).

Reports cover Entra ID and Azure role assignments (active and PIM-eligible), MFA status and
authentication methods, phishing-resistant method detection, account status (enabled,
cloud/hybrid), sign-in activity, and PIM group-based assignments.

## Running
```bash
pwsh ./Need4Admin_V1.0.ps1                    # interactive; answer N to skip Azure
pwsh ./Need4Admin_V1.0.ps1 -TenantId "..." -ClientId "..." -CertificateThumbprint "..."
```
PowerShell Core 7.0+ on macOS/Linux; Windows also works on PowerShell 5.1. The script offers
to install missing modules on first run.

## Key Rules
- Reports are written **outside the repository** — `$HOME/Need4Admin-Reports/` on macOS/Linux,
  `%USERPROFILE%\Need4Admin-Reports\` on Windows. This is deliberate: the output holds
  privileged UPNs and MFA state and must never reach version control. Keep it that way.
- Platform is detected via `$IsWindows` / `$IsMacOS` / `$IsLinux` plus a version check for
  Windows PowerShell 5.1, and `$env:PSModulePath` is adjusted accordingly
  (`$HOME/.local/share/powershell/Modules` vs `Documents\WindowsPowerShell\Modules`).
- Module loading is deliberately resilient: load with `-MinimumVersion`, never
  `-RequiredVersion`; fall back to any available version; load Az modules before
  Microsoft.Graph to avoid assembly conflicts. Do not tighten this — Windows hosts have real
  version conflicts and the fallback is what keeps the script usable there.
- Opening the HTML report is platform-specific: `Start-Process` / `open` / `xdg-open`
  (Linux needs the `xdg-utils` package).
- The privileged roles being monitored live in the `$PrivRoles` hashtable (31 roles). Add or
  remove roles there, never inline at the call site.
- Read-only by design: every Graph and Azure scope is a read scope and the script modifies no
  configuration. Keep it that way.

## Required Graph scopes
`Directory.Read.All`, `User.Read.All`, `UserAuthenticationMethod.Read.All`,
`RoleManagement.Read.Directory`, `RoleManagement.Read.All`,
`RoleEligibilitySchedule.Read.Directory`, `RoleAssignmentSchedule.Read.Directory`,
`AuditLog.Read.All`, `Reports.Read.All`, `Group.Read.All`

## Testing
No formal test suite — this is a standalone script. Before shipping a change verify: Windows,
macOS and Linux; with and without preinstalled modules; interactive and service-principal
auth; with and without the Azure connection; that the HTML report opens on each platform; and
that insufficient permissions fail gracefully rather than crashing.

## Do Not
- Do not commit reports, tenant IDs, client IDs, certificates, or any scan output
- Do not add write scopes or any operation that modifies a tenant
- Do not pin modules to exact versions
- Do not delete code that appears unused — verify via grep first
- Do not rename functions without updating all references
