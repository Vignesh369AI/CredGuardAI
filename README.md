# 🛡️ CredGuard v10.5

## Installation and Local Ollama Configuration Guide

> **Microsoft Edge Manifest V3 Extension** • **Node.js Bridge** • **Local Ollama AI**

CredGuard transforms browser credential-entry activity into privacy-preserving, actionable security signals. This guide installs the complete Windows prototype, starts the local AI services, loads the extension into Microsoft Edge, configures Ollama, and validates the end-to-end flow.

---

## 🚀 Quick Start Flow

```text
Download ZIP
   ↓
Extract CredGuard-v10.5-AI-Consistency
   ↓
Install Node.js 18+ and Ollama
   ↓
Run Ollama on port 11434
   ↓
Download llama3.2:3b
   ↓
Start CredGuard bridge on port 8787
   ↓
Load the folder containing manifest.json in Edge
   ↓
Enable Ollama in the CredGuard Admin Console
   ↓
Validate a controlled credential-entry event
```

> [!IMPORTANT]
> Load **the folder that directly contains `manifest.json`**. Do not load the ZIP file, the outer extraction folder, or the `ollama-bridge` subfolder.

---

## 📋 Guide Contents

1. [Prerequisites](#1-prerequisites)
2. [Download and extract the ZIP](#2-download-and-extract-the-zip)
3. [Verify Node.js and npm](#3-verify-nodejs-and-npm)
4. [Install and start Ollama](#4-install-and-start-ollama)
5. [Download the model](#5-download-the-credguard-model)
6. [Check port 8787](#6-check-whether-port-8787-is-in-use)
7. [Start the Ollama bridge](#7-start-the-credguard-ollama-bridge)
8. [Verify bridge health](#8-verify-bridge-health)
9. [Load the extension in Edge](#9-load-the-unpacked-extension-in-microsoft-edge)
10. [Configure Ollama in CredGuard](#10-configure-ollama-in-credguard)
11. [Validate the complete flow](#11-validate-the-complete-flow)
12. [Stop and restart services](#12-stop-and-restart-services)
13. [Troubleshooting](#13-troubleshooting)
14. [Final checklist](#14-final-validation-checklist)
15. [Command reference](#15-command-quick-reference)

---

# 1. Prerequisites

| Requirement | Validated Requirement |
|---|---|
| **Windows** | Windows 10 22H2 or newer for the current Ollama Windows application |
| **Microsoft Edge** | Developer mode enabled for local sideloading |
| **Node.js** | Version 18 or newer |
| **npm** | Installed with Node.js |
| **Ollama** | Local API available on `127.0.0.1:11434` |
| **CredGuard package** | `CredGuard_v10_5_AI_Consistency.zip` |

> [!CAUTION]
> Keep Ollama and the bridge bound to localhost. Do not expose ports `11434` or `8787` to untrusted networks.

---

# 2. Download and Extract the ZIP

The ZIP contains two parts:

- The **Microsoft Edge extension source**
- The **local Node.js Ollama bridge**

## 2.1 Extract the package

1. Download `CredGuard_v10_5_AI_Consistency.zip`.
2. Right-select the ZIP and choose **Extract All**.
3. Extract it to a permanent location, for example:

```text
C:\CredGuard
```

4. Verify the extracted structure:

```text
C:\CredGuard\CredGuard-v10.5-AI-Consistency\
│
├── manifest.json              ← Edge loads this folder
├── background.js
├── content.js
├── popup.html
├── popup.js
├── admin.html
├── admin.js
├── admin.css
├── README.md
│
└── ollama-bridge\             ← Run npm commands here
    ├── package.json
    ├── server.mjs
    └── .env.example
```

## 2.2 Exact folder to load in Edge

✅ **Select this folder:**

```text
C:\CredGuard\CredGuard-v10.5-AI-Consistency
```

❌ **Do not select:**

```text
CredGuard_v10_5_AI_Consistency.zip
C:\CredGuard
C:\CredGuard\CredGuard-v10.5-AI-Consistency\ollama-bridge
```

> [!TIP]
> Before selecting the folder, open it in File Explorer and confirm that `manifest.json` is directly visible.

---

# 3. Verify Node.js and npm

The CredGuard bridge declares **Node.js 18 or newer**.

Open PowerShell and run:

```powershell
node --version
npm --version
```

## Expected result

```text
v18.x.x or newer
A valid npm version number
```

If either command is not recognized:

1. Install or repair Node.js.
2. Close all open terminal windows.
3. Open a new PowerShell window.
4. Run the verification commands again.

---

# 4. Install and Start Ollama

Ollama provides the local AI model API used by the CredGuard bridge.

## 4.1 Verify the Ollama CLI

```powershell
ollama --version
```

## 4.2 Start the Ollama service

```powershell
ollama serve
```

Keep this PowerShell window open.

> [!NOTE]
> The Ollama Windows application can already be running in the background. If a second `ollama serve` reports that the address is already in use, verify the existing service instead of starting a duplicate process.

## 4.3 Verify the Ollama API

Open a second PowerShell window:

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/tags"
```

### Expected result

A response containing a model collection. The collection can be empty before the model is downloaded.

---

# 5. Download the CredGuard Model

CredGuard v10.5 defaults to `llama3.2:3b` unless `OLLAMA_MODEL` overrides it.

## 5.1 Download the model

```powershell
ollama pull llama3.2:3b
```

## 5.2 Confirm the model is installed

```powershell
ollama list
```

### Expected result

```text
NAME
llama3.2:3b
```

## 5.3 Optional model test

```powershell
ollama run llama3.2:3b
```

Enter a harmless test question. Exit the interactive prompt using the method shown by the terminal.

---

# 6. Check Whether Port 8787 Is in Use

The CredGuard bridge listens on `127.0.0.1:8787` by default.

## 6.1 Check the port

```powershell
netstat -ano | findstr :8787
```

### If no output appears

Port `8787` is free. Continue to [Start the bridge](#7-start-the-credguard-ollama-bridge).

### If a listener appears

Example:

```text
TCP    127.0.0.1:8787    0.0.0.0:0    LISTENING    12345
```

The final value is the process ID, or PID.

## 6.2 Inspect the process

```powershell
tasklist /FI "PID eq 12345"
```

> [!WARNING]
> Terminate the PID only after confirming that it belongs to an old CredGuard or Node.js bridge process. Never terminate an unknown process.

## 6.3 Stop a confirmed stale bridge

```powershell
taskkill /PID 12345 /F
```

## 6.4 Confirm the port is free

```powershell
netstat -ano | findstr :8787
```

**Expected result:** no output.

---

# 7. Start the CredGuard Ollama Bridge

Open a dedicated PowerShell window for the bridge.

## 7.1 Navigate to the bridge folder

```powershell
cd "C:\CredGuard\CredGuard-v10.5-AI-Consistency\ollama-bridge"
```

Confirm the current folder contains:

```text
package.json
server.mjs
.env.example
```

## 7.2 Validate the package

```powershell
npm install
```

The verified v10.5 package does not declare third-party runtime dependencies, but this command validates the package metadata.

## 7.3 Start the bridge

```powershell
npm start
```

The package maps `npm start` to:

```text
node server.mjs
```

## 7.4 Expected startup output

```text
> credguard-ollama-bridge@10.5.0 start
> node server.mjs

CredGuard v10.5 Ollama bridge API 2.0 listening on http://127.0.0.1:8787
```

> [!IMPORTANT]
> Keep this terminal window open. Closing the window stops the bridge. The npm prefix can vary slightly, but the final CredGuard listening message should be present.

---

# 8. Verify Bridge Health

Open another PowerShell window.

## 8.1 Call the health endpoint

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:8787/health" | ConvertTo-Json
```

## 8.2 Expected health response

```json
{
  "ok": true,
  "apiVersion": "2.0",
  "bridgeVersion": "10.5.0",
  "ollamaAvailable": true,
  "model": "llama3.2:3b",
  "queueDepth": 0,
  "active": false,
  "cacheEntries": 0,
  "minScore": 60
}
```

Runtime values can change for `queueDepth`, `active`, and `cacheEntries`.

## 8.3 Confirm port ownership

```powershell
netstat -ano | findstr :8787
```

Copy the PID from the final column, then run:

```powershell
tasklist /FI "PID eq <PID_FROM_NETSTAT>"
```

## If `ollamaAvailable` is `false`

1. Confirm `ollama serve` is running.
2. Confirm the Ollama API responds on port `11434`.
3. Confirm `llama3.2:3b` appears in `ollama list`.
4. Restart the bridge after Ollama is available.

---

# 9. Load the Unpacked Extension in Microsoft Edge

1. Open Microsoft Edge.
2. Enter `edge://extensions` in the address bar.
3. Turn on **Developer mode**.
4. Select **Load unpacked**.
5. Select this exact folder:

```text
C:\CredGuard\CredGuard-v10.5-AI-Consistency
```

6. Select **Folder**.
7. Confirm **CredGuard v10.5** appears and is enabled.

> [!IMPORTANT]
> If Edge reports **Manifest file is missing or unreadable**, the wrong folder level was selected. Select the directory in which `manifest.json` is directly visible.

---

# 10. Configure Ollama in CredGuard

## 10.1 Open the Admin Console

1. On `edge://extensions`, locate **CredGuard v10.5**.
2. Select **Details**.
3. Select **Extension options**.
4. Open the Ollama or AI configuration section.

## 10.2 Apply the recommended initial settings

| Setting | Value |
|---|---|
| **Enable AI enrichment** | Enabled |
| **Bridge URL** | `http://127.0.0.1:8787` |
| **Model** | `llama3.2:3b` |
| **Minimum LLM score** | `60` unless intentionally testing another value |
| **Protection mode** | Detect for initial validation |

5. Save the settings.
6. Return to `edge://extensions`.
7. Select **Reload** for CredGuard if changes are not immediately visible.

> [!CAUTION]
> Use Block mode only in an authorized, controlled test environment after validating the detection behavior.

---

# 11. Validate the Complete Flow

## 11.1 Pre-test checks

- [ ] Ollama API responds on `127.0.0.1:11434`.
- [ ] `llama3.2:3b` appears in `ollama list`.
- [ ] Bridge health returns `ok: true`.
- [ ] Bridge health returns `ollamaAvailable: true`.
- [ ] CredGuard v10.5 is enabled in Edge.
- [ ] Bridge URL is configured as `http://127.0.0.1:8787`.

## 11.2 Run an authorized test

1. Open an authorized controlled test page containing a password field.
2. Trigger a credential-entry or submission test.
3. Open the CredGuard event table.
4. Review the following fields:
   - Action type
   - Remote domain
   - Risk level and score
   - Reason codes
   - Enforcement action
   - AI status
   - AI classification
   - Classification source
   - Latency
   - Privacy marker

## 11.3 Expected AI lifecycle

```text
Pending → Running → Completed
```

## Expected policy behavior

| Result | Meaning |
|---|---|
| **Completed** | The model or policy returned a completed result. |
| **Skipped** | Load policy intentionally avoided the LLM call. |
| **Pending** | The job is waiting in the bridge queue. |
| **Running** | The single worker is processing the job. |
| **Failed** | Bridge, model, request, schema, or processing failure occurred. |

> [!NOTE]
> In v10.5, `CredentialEntryObserved` events and events below `minScore` can be **Skipped**. Trusted-domain matches can complete through policy without calling Ollama. These are expected optimization behaviors.

---

# 12. Stop and Restart Services

## 12.1 Stop the bridge gracefully

In the terminal running `npm start`, press Ctrl+C.

Verify the port is free:

```powershell
netstat -ano | findstr :8787
```

## 12.2 Force-stop a confirmed stale bridge

```powershell
netstat -ano | findstr :8787
```

```powershell
tasklist /FI "PID eq <PID>"
```

```powershell
taskkill /PID <PID> /F
```

## 12.3 Inspect and stop the Ollama model

```powershell
ollama ps
ollama stop llama3.2:3b
```

If `ollama serve` was started manually, press Ctrl+C in that terminal. If the Windows background application started Ollama, use the application controls when a full service stop is required.

## 12.4 Restart order

```text
1. Start Ollama
2. Verify port 11434
3. Start the CredGuard bridge with npm start
4. Verify bridge health on port 8787
5. Reload the Edge extension if required
```

---

# 13. Troubleshooting

## 🔴 Manifest file is missing

**Cause:** The wrong folder was selected in Edge.

**Fix:** Select the folder that directly contains `manifest.json`:

```text
C:\CredGuard\CredGuard-v10.5-AI-Consistency
```

## 🔴 `npm` is not recognized

**Cause:** Node.js or npm is unavailable, or the terminal has an old environment.

**Fix:** Install or repair Node.js 18+, close the terminal, open a new PowerShell window, and run:

```powershell
node --version
npm --version
```

## 🔴 Port 8787 is already in use

**Cause:** Another bridge process is listening.

**Fix:**

```powershell
netstat -ano | findstr :8787
tasklist /FI "PID eq <PID>"
taskkill /PID <PID> /F
```

Terminate the PID only after confirming the process identity.

## 🔴 `ollamaAvailable` is `false`

**Cause:** Ollama is not available on port `11434`.

**Fix:**

```powershell
ollama serve
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/tags"
ollama list
```

## 🔴 Model not found

```powershell
ollama pull llama3.2:3b
ollama list
```

## 🟠 AI status remains `Pending`

Check:

1. The bridge terminal for an error.
2. `/health` for `ollamaAvailable`.
3. `queueDepth` and `active`.
4. `ollama ps` for the running model.
5. Whether the model is still loading.

## 🟡 AI status is `Skipped`

This can be expected when:

- The event is `CredentialEntryObserved`.
- The deterministic score is below `minScore`.
- Trusted-domain policy bypasses the LLM.
- A cached or policy result is used.

## 🔴 Bridge health is connection refused

Start the bridge from the correct folder:

```powershell
cd "C:\CredGuard\CredGuard-v10.5-AI-Consistency\ollama-bridge"
npm start
```

---

# 14. Final Validation Checklist

## Installation

- [ ] ZIP extracted successfully.
- [ ] Extension root contains `manifest.json`.
- [ ] Node.js 18+ is installed.
- [ ] npm is available.
- [ ] Ollama is installed.
- [ ] `llama3.2:3b` is downloaded.

## Services

- [ ] Ollama API responds on port `11434`.
- [ ] No unintended process occupies port `8787`.
- [ ] `npm start` displays the CredGuard listening message.
- [ ] `/health` returns `ok: true`.
- [ ] `/health` returns `bridgeVersion: 10.5.0`.
- [ ] `/health` returns `ollamaAvailable: true`.

## Extension

- [ ] Edge loaded the folder containing `manifest.json`.
- [ ] CredGuard v10.5 is enabled.
- [ ] AI enrichment is enabled.
- [ ] Bridge URL is `http://127.0.0.1:8787`.
- [ ] Model is `llama3.2:3b`.
- [ ] Protection mode is Detect for initial testing.

## End-to-End Test

- [ ] Controlled test event appears in the event table.
- [ ] Deterministic risk evidence is visible.
- [ ] Eligible event reaches Completed, or Skipped is explained by policy.
- [ ] No credential values or page content are recorded.

---

# 15. Command Quick Reference

## Ollama

```powershell
ollama --version
ollama serve
ollama pull llama3.2:3b
ollama list
ollama run llama3.2:3b
ollama ps
ollama stop llama3.2:3b
```

## CredGuard bridge

```powershell
cd "C:\CredGuard\CredGuard-v10.5-AI-Consistency\ollama-bridge"
npm install
npm start
```

## Port and process management

```powershell
netstat -ano | findstr :8787
tasklist /FI "PID eq <PID>"
taskkill /PID <PID> /F
```

## Health checks

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/tags"
Invoke-RestMethod -Uri "http://127.0.0.1:8787/health" | ConvertTo-Json
```

---

# ✅ Successful Installation State

A successful local installation has all of the following:

```text
Ollama API               http://127.0.0.1:11434
Ollama model             llama3.2:3b
CredGuard bridge         http://127.0.0.1:8787
Bridge API version       2.0
Bridge version           10.5.0
Edge extension           CredGuard v10.5 enabled
Initial protection mode  Detect
```

> **CredGuard is ready when the extension can store deterministic credential-entry risk evidence and eligible events can receive a completed or intentionally skipped enrichment result.**
