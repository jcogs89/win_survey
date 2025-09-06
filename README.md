# Windows Threat‑Hunting Survey (`win_survey.ps1`)

A single‑file, **PowerShell 5.1–compatible** collection script for quick Windows triage and threat‑hunting.  
It writes a streaming **TXT log** (no persistent file lock) _and_ a structured **JSON report** that tooling can parse later.

---

## Table of contents
- [What it collects](#what-it-collects)
- [Why this script](#why-this-script)
- [Requirements](#requirements)
- [Usage](#usage)
- [Outputs](#outputs)
- [Fields in the JSON](#fields-in-the-json)
- [Optional Sysinternals integration](#optional-sysinternals-integration)
- [Operational notes](#operational-notes)
- [Troubleshooting](#troubleshooting)
- [Security considerations](#security-considerations)
- [License](#license)

---

## What it collects

High‑signal host/network and persistence data with safe fallbacks and short timeouts:

- **Timestamp & Host**: OS caption/version/build/arch (CIM), logged‑in user, hostname, current time.
- **Sessions / Shares / Open Files**: SMB shares/sessions/open files (skips if LanmanServer is stopped; falls back to `net share`).
- **Network configuration**: IP config, routes, neighbor/ARP table.
- **Active connections**: `Get-NetTCPConnection` with owning PID → process name; fallback to `netstat -ano` if needed.
- **ARP / DNS cache**: `Get-NetNeighbor` (IPv4) and `Get-DnsClientCache`.
- **Services**: `Get-Service` (status/name/display) with fallback to `sc query`.
- **Processes**: `Get-Process` (Id, Name, Path, StartTime, CPU, PM, WS).
- **Loaded drivers (kernel)**: `Get-CimInstance Win32_SystemDriver` (Name, State, PathName, StartMode).
- **Autoruns (Registry)**: Common Run/RunOnce keys and `Svchost` service grouping.
- **Startup folders**: Per‑user and All Users startup folder contents.
- **Scheduled Tasks**: Task path/name, state, author, next/last run, result, and action command lines (non‑Microsoft first).
- **Console History**: `Get-History` (if present in the current session).
- **Sysinternals (optional)**: `logonsessions.exe`, `psfile.exe`, `psloggedon.exe`, `psservice.exe`, `pslist.exe`, `listdlls.exe`, `handle.exe` (auto‑skips if missing).

> Script primitives include short **timeouts** for slow cmdlets and **fallbacks to native CLI** to avoid hanging on misconfigured hosts.

## Why this script

- **No persistent file locks**: the TXT log is appended line‑by‑line; the JSON is written **once at the end**.
- **PS 5.1‑only environments**: works on stock Windows 10/11 without requiring RSAT or external modules.
- **Graceful degradation**: if a cmdlet is unavailable or times out, the script uses a CLI fallback or emits “(no data)”.

## Requirements

- **Windows PowerShell 5.1** (preinstalled on Windows 10/11).  
- **Standard user** works for most sections; **Administrator** is recommended to maximize coverage (e.g., services, drivers, SMB).  
- Optional: **Sysinternals** tools in `PATH` **or** in the **same folder** as the script.

## Usage

Open **Windows PowerShell** (preferably *Run as Administrator*) and execute one of the following:

```powershell
# From the script’s folder
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\\win_survey.ps1

# Or, dot-call from an interactive session
Set-Location <folder-containing-script>
.\\win_survey.ps1
