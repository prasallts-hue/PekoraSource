# Preliminary security audit — PekoraSource

**Scope:** static review of the public `main` snapshot (base commit `54c25c704a3c2cbecf5d3c79b823a43e492b30bf`).  
**Method:** repository metadata, tracked-file inventory, selected scripts/configuration, and source inspection. No executable, installer, container image, or build step was run.

## Verdict

**Not cleared for deployment yet.** The inspected C# inventory endpoint does not contain the reported active reverse-shell code in this snapshot, but the repository includes unverified executable binaries and still has deployment/configuration hazards. This is a preliminary static review, not a full malware certification.

## Findings

### 1. Historical Pekora backdoor report — relevant source lineage

A public report in the related Korone source repository describes a backdoor in an older 30 August 2025 Pekora source: a hard-coded asset ID triggered a TCP connection to `51.15.158.185:9001` and exposed a Windows command shell. The report points to `Roblox/Roblox.Website/Controllers/v2/Inventory.cs` in `Anontux/pekora-latest-src`.

In this repository, the corresponding file is:

`Roblox/Roblox.Website/Controllers/v2/Inventory.cs`

The current method contains a comment warning about the old RAT, but the reported asset-ID condition, TCP client connection, and process/shell-launch block are not present in the inspected method. The file still has unused `System.Net.Sockets` and `System.Diagnostics` imports, which look like leftovers and should be cleaned up after the complete source comparison.

**Assessment:** the specific reported code is not active in the inspected current file. This does not prove every other file or binary is clean.

### 2. Unverified executable with random name — high priority

`RCCService/RCCService2018/kOxI4fIqlll4IIU4.exe`

- Size: 26,888,660 bytes.
- Name is unrelated to the other RCC files.
- It was not referenced by the startup scripts inspected.
- Its contents, publisher/signature, and antivirus reputation have not been verified.

**Action:** do not execute or deploy it. Keep it quarantined until its origin and hash are independently verified. Do not delete it blindly before checking whether a legitimate component depends on it.

### 3. Other binary files are not cleared

The repository tree contains 42 files with executable/library/archive extensions (including RCCService executables, DLLs, and archives). They were not dynamically executed or fully scanned in this review. A source-only review cannot rule out a payload embedded in one of these files.

### 4. External plain-HTTP RCC endpoint

These files currently point RCC at `http://www.silrev.biz` or `http://silrev.biz`:

- `RCCService/AppSettings.xml`
- `RCCService/RCCService2017/AppSettings.xml`
- `RCCService/RCCService2018/AppSettings.xml`
- `RCCService/RCCService2020/AppSettings.xml`

Replace this with the revival's own HTTPS base URL before running RCC. Do not send user/session data to an endpoint you do not control.

### 5. Startup scripts are not portable

- Root `start.bat` references `2016-roblox-main`, which is not present in the inspected tree.
- `Roblox/Roblox.Website/run.bat` contains a hard-coded path under `C:\goober\prjx-frontend-patches\Project-X-V2\...`.
- Root `run.bat` invokes that hard-coded website script.

These scripts need to be rewritten to use paths relative to the repository and the actual folder layout.

### 6. Required local configuration is absent from the public tree

The following files were not present at the inspected paths:

- `frontend/config.json`
- `game-server/config.json`
- `game-server/cookies.txt`
- `Roblox/Roblox.Website/appsettings.json`
- `game-servers.json`

The source expects local configuration and secrets. Create documented example templates with placeholders; never commit real database passwords, bot tokens, cookies, authorization keys, or OAuth secrets.

### 7. Missing game-arbiter

`Notes.md` explicitly says game-arbiter is not included and that game servers will not work without it. This is a missing implementation dependency, not evidence of a RAT.

### 8. Privileged process management

`game-server/src/controllers/index.ts` launches RCC processes and, in Docker mode, invokes scripts through `sudo`. This is expected for this architecture but must run under a dedicated, least-privilege service account with narrowly scoped permissions. Do not expose the game-server HTTP/WebSocket ports publicly.

The thumbnail upload route bypasses the shared HTTP auth middleware but checks an access key in the request body. The WebSocket uses an authorization key in the query string; configure reverse-proxy logging so query strings and secrets are never logged.

## Safe next steps

1. Keep the repository on a separate preparation branch; do not deploy from `main` yet.
2. Quarantine and independently verify all executable/library/archive files, especially the randomly named EXE.
3. Replace every RCC base URL with the CANDY HTTPS domain.
4. Add example configuration files containing placeholders only.
5. Fix startup scripts and document the required services.
6. Implement or replace the missing game-arbiter.
7. Integrate the CANDY frontend only after the API routes, authentication format, cookies, and CORS policy are defined.
8. Run builds and tests in an isolated disposable environment after the binary review.

## Limitations

This review did not execute code, run a full antivirus/YARA scan, inspect every one of the 20,230 tracked paths manually, verify binary signatures, or prove that all dependencies are benign. The repository must not be described as malware-free based only on this report.
