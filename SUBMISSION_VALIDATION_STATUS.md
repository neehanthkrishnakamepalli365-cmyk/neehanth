# Authoritative ByteProbe validation status (16 October 2026)

This report supersedes the older 123/128/131 test counts and unverified-Windows statements retained as clearly historical descriptions in earlier handoff logs. Do not interpret those earlier counts as the current build.

| Item | Verified evidence |
|---|---|
| Unit tests on Windows, Python 3.13.7 | 139 passed, 0 errors/failures/skips; `windows_tests_after_fix.log` (if separately retained), with Windows suite log under `verification_results/logs/` |
| Unit tests under Linux/Xvfb | 139 passed, 0 skipped, as separately documented in the audit |
| Headless Linux | 134 passed, 5 GUI tests skipped |
| Windows checks | W1 GUI PASS; W2 ownership PASS; W3 NTFS attributes PASS; W4 ADS PASS; W5 live $MFT PASS; W6 controlled $SI-only timestamp change PASS; W7 export PASS |
| W5 | Read-only `\\.\D:` NTFS parse, 67,424 base records; target record and fixups confirmed |
| W6 | Controlled disposable-file SetFileTime Created backdate; $SI changed and $FN preserved; two anomaly findings |
| Independent public $MFT | 13,064 base records, 100% matching reported fields against independently installed Rust-backed `mft` binding, and 500/500 landed injections for each of six profiles |

**Methodological limits:** This does not show universal NTFS parser correctness; VirusTotal was mocked; real NSRL reference corpus and second image/Autopsy/FTK benchmark were not run. The six controlled attack profiles have meaningful detection failures that must remain reported.

**Privacy:** Computer names/usernames were replaced in submission text evidence. Raw originals must be retained privately. Names in third-party public MFT data are not guaranteed anonymised. Historical failures must not be represented as successful first attempts.
