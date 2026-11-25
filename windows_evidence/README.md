# Windows verification chronology (16 October 2026)

1. `W1_W4_W7/` (07:58), `W1_W4_W7_retry/` (07:59) and `W1_W4_W7_failed_before_fix/` (08:00) are the three runs that **failed W4**; the last of these was formerly named `final`, which was misleading. `W1_W4_W7_fixed/` (08:06) is the first run that passed W4 after the correction below.
2. The real NTFS ADS stream existed independently; ByteProbe's `ctypes` binding omitted 64-bit handle argument declarations for `FindNextStreamW` and `FindClose`, raising OverflowError. That defect was corrected in `core/hidden_data.py`.
3. `W1_W4_W7_after_ads_fix/` is the authoritative later W1–W4/W7 run: all five PASS, with ADS `:byteprobe_test:$DATA` detected.
4. `W5_W6_retry/` records the administrator-authorised one-file lab: W5 and W6 PASS. Earlier run refused without elevation; preserve it if available.
5. Prior files are retained solely to document the genuine failure → change → retest chronology; they are not final results.

Redaction of account names in JSON is purely for privacy; record outcomes remain unchanged. See `SUBMISSION_VALIDATION_STATUS.md`.
