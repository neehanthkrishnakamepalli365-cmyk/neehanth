# BYTEPROBE — WINDOWS VALIDATION HANDOFF FOR THE FRIEND
**Project:** Neehanth Kamepalli — Windows File Analysis Tool for Digital Forensics (ByteProbe)  
**Document version:** 16 October 2026  
**Purpose:** generate AUTHENTIC Windows test evidence for a later academic re-evaluation. This guide is not a claim that the Windows checks have passed.

## Start here — what you should return

All output is automatically saved in the **same extracted `ByteProbe` project folder**:

```
ByteProbe\
  RUN_1_WINDOWS_BASIC.cmd        <-- start here
  FRIEND_WINDOWS_STEP_BY_STEP.md
  tools\friend_validation_runner.py
  verification_results\         <-- all logs and machine-generated evidence
    windows\
    parser\
    logs\
    setup\
    MANIFEST_SHA256.txt
  ByteProbe_Verification_Results.zip  <-- send this ZIP back
```

`verification_results` is created by the tools; **do not create fake JSON/CSV files or change failed outcomes to PASS.** After every test stage the result ZIP is updated automatically. Review for personal usernames, directory names and sensitive file paths before sharing it. The primary ByteProbe code, test files, and dissertation are separate from the generated results.

### The tasks

| Task | Who performs it | Expected evidence | Status before running |
|---|---|---|---|
| **A. Basic Windows checks W1–W4/W7** | Friend, ordinary Windows account | Unit-test transcript, `windows_validation.json`, Windows reports | Not tested on this Windows host |
| **B. Live NTFS W5/W6** | Friend, admin, using a **dedicated test VHDX** | Real before/after `$SI`/`$FN` JSON and logs | Not tested |
| **C. Independent MFTECmd comparison** | Friend obtains separate lawful extracted `$MFT` and MFTECmd | Reference CSV, field-level differences, hashes, provenance | Not tested |
| **D. Detector interpretation** | Academic reviewer after A–C | Measured results, missed attacks and benign alerts discussed honestly | Research analysis still limited |

**Safe stopping point:** If there is no dedicated VHDX, complete A only; do NOT run B on the personal/system drive. If there is no second independent `$MFT`, skip C and report 'not tested'. A skipped scientific validation is preferable to risking someone's computer or inventing results.

---

## A. Basic checks — anyone with Windows 10/11, no VM, no Administrator required

### A1. Prerequisites

1. Obtain the ByteProbe ZIP from the project owner; do not execute files from an untrusted sender. If Windows Defender warns about code, stop and have the owner verify it rather than disabling security.
2. Install **Python 3.10 or later** for Windows, including the `py` launcher. Internet is required to install Python and the declared packages. The PC must have permission to run Python software.
3. Right-click ZIP > **Extract All** to a local folder such as `Documents\ByteProbe_Project`. Go into the **inner `ByteProbe` folder** containing `main.py`, `core`, `tests` and `RUN_1_WINDOWS_BASIC.cmd`. Avoid network drives, sync folders and spaces requiring elevated access if possible.
4. Open PowerShell there and verify: `py -3 --version`. Keep the project folder where it is throughout testing.
5. Be aware that reports may include the local Windows test user's account name and disposable file paths. They do not need personal files or browser data.

### A2. Execute the basic runner

Double-click **`RUN_1_WINDOWS_BASIC.cmd`**. Alternatively, in Command Prompt opened in the `ByteProbe` folder:

```bat
RUN_1_WINDOWS_BASIC.cmd
```

The command file creates a private `.venv` Python environment **inside ByteProbe**, installs `requirements.txt` (`pywin32` for Windows and `reportlab` for PDF), runs the full test suite and W1–W4/W7 checks. It retains logs even when a check fails.

**Do not move or rename the project during execution.** Expect a visible completion or failure message. If it fails, copy the message and send the resulting ZIP anyway. Do not delete or edit failures.

### A3. Check basic results in the same folder

Open `ByteProbe\verification_results\` and check:

- `windows\environment.json` — Windows and Python version, package versions
- `logs\windows_full_unit_suite.txt` — actual number of unit/GUI tests and skipped/failed information
- `logs\windows_basic_W1_W4_W7.txt` — Windows-only check transcript
- `windows\basic\windows_validation.json` — W1 (GUI), W2 (ownership), W3 (attributes), W4 (ADS), W7 (reports)
- `windows\basic\W1_gui_tests.txt` — actual five GUI smoke-test results
- `windows\basic\W7_metadata.csv`, `W7_report.html`, `W7_report.pdf` — disposable test reports if generation succeeded
- `windows\windows_validation.json` — conservative consolidated summary, W5/W6 still NOT_TESTED
- `windows\basic_stage.json` — stage status
- `ByteProbe_Verification_Results.zip` — ready-to-share ZIP at ByteProbe project root

**Interpretation:** Previous Linux/Xvfb runs passed 139 tests; the friend must report the **actual Windows count**. A test passed on Linux must not be counted as a Windows pass.

---

## B. Safe W5/W6 live NTFS experiment WITHOUT a virtual machine

**Requires admin privileges. Do not use the Windows system drive (`C:`), evidence disks, external drives with personal files, or existing data partitions.** The recommended solution is a new **disposable VHDX file**, which behaves like an independent disk volume without running a virtual machine.

### B1. Create a disposable VHDX (manual Windows action)

1. Save any important work, back up as appropriate, and obtain the computer owner's express permission.
2. Press `Win + R`, enter `diskmgmt.msc`, press Enter, approve Administrator access.
3. In **Disk Management > Action > Create VHD**, choose an ordinary local location such as `C:\ByteProbe_TestDisk.vhdx` (not in `verification_results`). Choose **VHDX**, **dynamically expanding**, approximately **1–2 GB**. Select OK.
4. Carefully locate the **new virtual disk by its size and VHD label**, not by guessing its disk number. Initialize **only that new virtual disk**, using GPT when prompted. Then choose its **Unallocated** space > **New Simple Volume** > format **NTFS** > select an available drive letter, e.g. `R:`. **Never initialize, repartition or format an existing disk.** If uncertain which disk is the VHDX, stop.
5. Open File Explorer to confirm a *new, empty* NTFS `R:` volume (actual letter may differ). In PowerShell verify `Get-Volume -DriveLetter R` shows `FileSystem: NTFS`.
6. Create a dedicated folder on that new volume:

```powershell
New-Item -ItemType Directory -Force -Path 'R:\ByteProbeLab'
```

Replace `R:` in **every command** with the real letter Windows assigned. The volume must be disposable and have no valuable files. Do not move the virtual disk during the test.

Microsoft's official virtual-disk creation instructions: https://learn.microsoft.com/en-us/windows-server/storage/disk-management/manage-virtual-hard-disks

### B2. Run the lab against ONLY that new VHDX volume

1. Search for **Windows PowerShell** > **Run as administrator**.
2. Change directory to the existing project folder. Example (replace path with the actual one):

```powershell
Set-Location 'C:\Users\FRIEND\Documents\ByteProbe_Project\ByteProbe'
```

3. Run **only the guarded handoff command below**, specifying the disposable volume directory:

```powershell
.\.venv\Scripts\python.exe tools\friend_validation_runner.py lab --test-root 'R:\ByteProbeLab' --acknowledge-disposable-vhd
```

The guarded runner **refuses the Windows system drive**, requires the explicit directory, and calls the supplied W5/W6 test. W5 reads the NTFS volume; W6 changes **only the creation timestamp of one new temporary specimen**, then compares it with its `$FILE_NAME` timestamps and removes the disposable specimen. If the script cannot locate an MFT record or encounters an error, keep the result and report the failure. It is not automatically a pass just because the command finishes.

### B3. Verify and retain lab evidence

The following files appear **inside the same ByteProbe folder**:

```
verification_results\windows\lab\W5_W6_real_windows_results.json
verification_results\windows\lab_stage.json
verification_results\windows\windows_validation.json
verification_results\logs\windows_lab_W5_W6.txt
ByteProbe_Verification_Results.zip
```

For a credible W5/W6 outcome, `W5` must contain real parsed records, a valid NTFS specimen and the actual source; `W6` must contain before/after timestamps, rule findings, same record/sequence, and no unexpected error. **A successful W6 profile-A test does not prove all six profiles or eliminate detector blind spots C/D/E/F.**

### B4. Clean up only after saving and transferring results

From Disk Management identify the **VHDX virtual disk**, choose **Detach VHD** (do not delete its data during detachment). After reviewing that the saved evidence is complete, the friend may delete the disposable `.vhdx` file if approved. Never format, initialize or detach an unrelated physical disk.

---

## C. Independent benchmark against MFTECmd (second NTFS source)

**This is separate from the W5/W6 test.** The benchmark must compare **two independently implemented parsers against the exact same bytes** from an extracted `$MFT`. To demonstrate generalisation, this input must also be a **different, independently sourced dataset** from the original ByteProbe 13,064-record sample. Renaming the original sample does not count.

### C1. Gather the inputs

- One authentic **extracted** `$MFT` from a separate, lawfully obtained NTFS image or disposable lab volume (not a live `\\.\C:` device string and not a VHDX disk container).
- Eric Zimmerman's legitimate **MFTECmd.exe** (record version/source; use the official project: https://github.com/EricZimmerman/MFTECmd). It is not bundled with ByteProbe.
- Written source information: origin/dataset name, acquisition process, date, Windows/NTFS version if known, permission to use, and SHA-256. If any item is unknown, state 'unknown' rather than guessing.

The colleague may need a forensic acquisition/extraction tool or a suitable public dataset to obtain the extracted `$MFT`. **Do not export or share the real PC's personal `$MFT` without the owner's explicit permission; it contains file-system metadata and filenames.** The first public 13 MB `$MFT` is not a substitute for a genuinely independent second sample.

### C2. Run the exact comparison (ordinary PowerShell; usually no admin required for extracted files)

From inside `ByteProbe`, using the Python environment created in A2:

```powershell
.\.venv\Scripts\python.exe tools\friend_validation_runner.py benchmark --mft-file 'C:\ForensicLab\dataset_B_MFT' --mftecmd-exe 'C:\ForensicTools\MFTECmd\MFTECmd.exe'
```

Replace both quoted paths with **real existing files**. The runner first generates `reference.csv` from the independent MFTECmd binary (`--at` ensures `$FILE_NAME` timestamp fields are included). It then executes ByteProbe's comparison against **that exact file** and saves every disagreement, missing key, excluded field and summary.

### C3. Check outputs

```
verification_results\parser\reference.csv
verification_results\parser\summary.json
verification_results\parser\summary.md
verification_results\parser\discrepancies.csv
verification_results\parser\unmatched_record_keys.csv
verification_results\parser\provenance.json
verification_results\parser\benchmark_stage.json
verification_results\logs\mftecmd_help_version.txt
verification_results\logs\mftecmd_reference_export.txt
verification_results\logs\byteprobe_mftecmd_comparison.txt
```

**Important:** `RESULTS_GENERATED_REVIEW_REQUIRED` is not the same as “100% agreement.” The reviewer must inspect common record keys, ambiguous entries, missing fields and differences. If the MFTECmd release changes CSV columns, preserve the failure and send the logs rather than altering data to force agreement.

### C4. Return the actual evidence source (if sharing is permitted)

The benchmark runner preserves **hashes and source path**, but does **not automatically copy** the potentially sensitive `$MFT` into the results ZIP. If the sample is authorised for sharing, copy it into `verification_results\parser\dataset_B_MFT`, then from the ByteProbe folder run:

```powershell
.\.venv\Scripts\python.exe tools\friend_validation_runner.py bundle
```

The fresh ZIP will include the sample. If sharing the raw file is not permitted, provide a source location and keep it on the authorised system; the examiner can review the summary but cannot independently reproduce the benchmark without access to the original bytes.

---

## D. Detector performance — what to report accurately

Your colleague should **not tune thresholds to force 100% detection** on the same dataset. The original research includes six synthetic manipulation profiles. Report independent Windows observations as additional evidence, and distinguish benign operations from maliciously constructed scenarios.

- **Profile A:** Created/Modified backdating — W6 directly tests the Created-time component on one disposable Windows file.
- **Profile B:** Backdating all `$SI` timestamps with sub-second precision — NOT verified by W6; needs separate controlled witnessed operations and an independent parser.
- **Profile C:** Modified-only backdating — may be indistinguishable from legitimate copy/preservation; retain documented miss.
- **Profile D:** Forward-dating — may evade currently implemented backdating rules; retain documented miss.
- **Profile E:** Coordinated `$SI` + `$FN` changes — beyond `$SI`/`$FN` discrepancy alone; **do not run low-level disk editing on the friend's PC**. Record as a fundamental methodological limit unless independent artefacts are studied in a safe forensic lab.
- **Profile F:** Copying other timestamps — confidence handling and benign-pattern alerts must be analysed, not concealed.

The controlled synthetic benchmark already highlights the trade-off: high-confidence detections **180/480 modified cases** and alerts on **40/160 benign controls**. Those are synthetic-corpus statistics, not verified Windows rates. Real Windows measurements should be reported separately, with labelled ground truth and denominators.

---

## E. What to send back to the project owner

**Send:**
1. `ByteProbe_Verification_Results.zip` from the **same ByteProbe project folder**; it includes the individual logs, result JSON files, manifest of SHA-256 values and any benchmark CSV.
2. The source ZIP used for testing, or at minimum its SHA-256, so the examiner can ensure the code version matches the dissertation.
3. One short message: exact Windows version and drive type used; stage(s) attempted; any errors; whether the benchmark `$MFT` was a genuinely new evidence source; and whether the original MFT is available for independent review.
4. If permitted, a screenshot of ByteProbe's NTFS MFT tab on the disposable test volume. Avoid showing unrelated personal filenames.

**Do not send:** private API keys, Windows passwords, personal case evidence or raw `$MFT` files without permission. **Do not discard error logs.**

### Copy-and-send status template

```
ByteProbe Windows testing completed on: [real date/time + timezone]
Windows version: [Windows 10/11; build if known]
Python version: [version]
Test volume: [disposable VHDX, drive letter, NTFS] OR [not created]
Basic W1-W4/W7: [passed / failed / partly done / not attempted]
Full unit tests: [actual tests run, passed, failed, skipped]
Advanced W5: [passed / failed / not attempted]
Advanced W6: [passed / failed / not attempted]
Independent dataset B: [source or not available]
MFTECmd version and comparison: [run, discrepancies / not attempted]
Files attached: ByteProbe_Verification_Results.zip [+ independent sample if permitted]
Important errors: [copy from logs]
```

### Academic hand-in criteria

Results must be reviewed against the dissertation **after** your friend sends them. Never alter the dissertation to say “Windows verified”, “independent MFTECmd benchmark completed”, or “90+ achieved” based on a prepared script or synthetic results. A higher grade remains a target, not an outcome any tool can guarantee.
