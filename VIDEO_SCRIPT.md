# Video Presentation Script — USB-2026-001

Target duration: 8–10 minutes. Demonstration order: Strings → Foremost → ExifTool → correlation → integrity verification.

> **Evidence status:** This is a controlled synthetic lab. The standard live demo covers the completed analysis only. The full manual PDF carve in `COMMANDS.md` section 9 is an optional follow-up; describe it as completed only after running the commands and verifying the actual hash and metadata output on this case image.

| Time | Section | Speaker |
| --- | --- | --- |
| 0:00–0:45 | Introduction and scope | Aaryan Sharma |
| 0:45–1:30 | Evidence image and preservation | Aditey Kshirsagar |
| 1:30–3:00 | Strings | Aditey Kshirsagar |
| 3:00–4:45 | Foremost | Aaryan Sharma |
| 4:45–6:00 | ExifTool | Aditya Baheti |
| 6:00–7:15 | Correlation and hashes | Aditya Baheti |
| 7:15–8:15 | Limitations and conclusion | Aaryan Sharma |
| 8:15–9:15 | Repository walkthrough | Aditya Baheti |
| 9:15–10:00 | Questions / closing | All |

## Demo preparation (before recording)

Open Kali Linux and run the non-destructive preflight commands from `~/forensic_case_study`:

```bash
cd ~/forensic_case_study
pwd
ls -lh usb_evidence.raw
sha256sum -c evidence/evidence_integrity.sha256
sha256sum -c evidence/recovered_files.sha256
```

The image-integrity check should report `usb_evidence.raw: OK`; recovered-file checks should report OK. If they fail, stop and investigate. Do not rerun image-creation, formatting, mounting, copy, or deletion commands during the demo.

## 0:00–0:45 · Introduction — Aaryan Sharma

**Show:** Title slide, Case ID USB-2026-001, three tools.

**Say:**

“We are presenting a controlled digital-forensics exercise using a synthetic 50 MiB FAT32 USB filesystem image. Our goal is to demonstrate how raw-string searching, file carving and metadata inspection can reveal artifacts that are no longer visible in a directory listing. We use GNU Strings, Foremost and ExifTool. This is synthetic evidence, not a real employee investigation.”

## 0:45–1:30 · Evidence identification and preservation — Aditey Kshirsagar

**Run:**

```bash
cd ~/forensic_case_study
stat -c 'Image size: %s bytes' usb_evidence.raw
cat evidence/evidence_integrity.sha256
sha256sum -c evidence/evidence_integrity.sha256
```

**Say:**

“The evidence object is `usb_evidence.raw`, a 50 MiB image containing 52,428,800 bytes. Its recorded SHA-256 begins `74f073eb` and ends `2611afc`. We check the baseline before analysis. The image was unmounted before examination, and our analysis writes go to separate output folders. The image and documents were generated specifically for this lab.”

## 1:30–3:00 · Tool 1: Strings — Aditey Kshirsagar

**Run:**

```bash
strings -a -t d usb_evidence.raw > output/all_strings.txt
grep -iE 'confidential|project_aurora|audit|finance|InternalOnly' output/all_strings.txt
```

**Say:**

“GNU Strings extracts readable character sequences from the raw image. The `-a` option scans the entire input, and `-t d` prints decimal byte offsets. We found nine matches, including the synthetic report title, author, subject and Project Aurora keywords. The first relevant page-text hit is at offset 825111; metadata hits appear later, from 825545 through 826492. These are indicators within the raw bytes, so we next try to recover files rather than relying on text matches alone.”

## 3:00–4:45 · Tool 2: Foremost — Aaryan Sharma

**Run** (the output directory already exists, so show the existing result rather than rerunning Foremost into the same directory):

```bash
cat output/foremost_recovered/audit.txt
find output/foremost_recovered -type f -printf '%p (%s bytes)\n'
xxd -l 16 output/foremost_recovered/png/00001610.png
```

**Say:**

“Foremost is a signature-based file-carving tool. Its audit log records two extracted artifacts: a 616-byte PDF beginning at offset 824832, and a 69-byte PNG at offset 824320. Dividing the PDF offset by 512 gives sector 1611, which appears in the recovered filename. The PNG’s leading bytes match the PNG signature. The PDF is not listed among the files remaining after the simulated deletion, so the carving result is important. Notice also that the recovered PDF is much smaller than the 4,569-byte reference document. We will not assume a carve is complete merely because a file was produced.”

## 4:45–6:00 · Tool 3: ExifTool — Aditya Baheti

**Run:**

```bash
exiftool output/foremost_recovered/pdf/00001611.pdf
exiftool -Title -Author -Subject -Keywords -CreateDate output/foremost_recovered/pdf/00001611.pdf
exiftool output/foremost_recovered/png/00001610.png
```

**Say:**

“ExifTool reads embedded document and image metadata. The recovered PDF is recognized as a one-page PDF 1.4 file. The targeted query returns no title, author, subject, keywords or creation date. The PNG is a 1-by-1 RGB image, with no relevant descriptive metadata reported. The empty PDF result does not prove the original document had no metadata; it tells us those fields are not available in the carved 616-byte file. This discrepancy leads to our cross-check.”

## 6:00–7:15 · Correlation — Aditya Baheti

**Run:**

```bash
strings -a output/foremost_recovered/pdf/00001611.pdf | grep -iE 'confidential|project_aurora|audit|finance|InternalOnly'
strings -a output/foremost_recovered/pdf/00001611.pdf | tail -n 12
grep -abo '%%EOF' usb_evidence.raw
sha256sum output/foremost_recovered/pdf/00001611.pdf /tmp/Internal_Financial_Audit_2026.pdf
```

**Say:**

“The recovered PDF contains the synthetic page text, but not the later descriptive metadata. In the raw image, Strings found metadata beyond the first carve boundary. The PDF has multiple end markers because ExifTool saved its metadata as an incremental update. Foremost stopped at the first end marker and recovered the 616-byte first revision. The original reference PDF and that partial carve have different hashes, so they are not byte-for-byte identical. The offsets and structure support the explanation that the carve is incomplete. We will claim a full recovery only if we separately run the manual carve and verify its actual size, hash and ExifTool output.”

### Optional extension only if the full manual carve has been run

If and only if the command block in `COMMANDS.md` section 9 has been executed on this image and the output actually confirms a 4,569-byte file, the expected reference hash, and all five metadata fields, show:

```bash
sha256sum output/manual_carve/audit_report_full.pdf
exiftool -Title -Author -Subject -Keywords -CreateDate output/manual_carve/audit_report_full.pdf
```

Otherwise, label the full recovery as **pending follow-up** and do not show expected output as if it were observed.

## 7:15–8:15 · Findings, integrity and conclusion — Aaryan Sharma

**Run:**

```bash
sha256sum -c evidence/evidence_integrity.sha256
sha256sum -c evidence/recovered_files.sha256
```

**Say:**

“Our results demonstrate three complementary techniques. Strings identified confidential-themed text in the raw image; Foremost recovered the PDF fragment and PNG; ExifTool described the recovered files and helped reveal the effect of incomplete carving. The saved checksums verify the image against its recorded baseline and verify the recovered artifacts against their saved hash list, provided both checks return OK. Because this is synthetic laboratory evidence, the conclusion is limited to demonstrating the workflow. It does not establish an actual data-theft incident, identify a person, or prove a real transfer.”

## 8:15–9:15 · GitHub repository — Aditya Baheti

**Show:** `README.md`, `COMMANDS.md`, `REPORT.md`, `tools/`, `evidence/`, `output/` and `screenshots/`.

**Say:**

“The repository stores the command log, per-tool explanations, reports, hashes and screenshots. The command log distinguishes completed work from the optional full-PDF recovery. This makes the lab reproducible and makes it clear which findings were directly observed.”

## 9:15–10:00 · Limitations and close — All

- **Aditey:** “Strings matches are indicators, so we correlated their offsets with the carved file.”
- **Aditya:** “ExifTool can only report metadata available in the recovered file. Incomplete carving can make metadata appear to be absent.”
- **Aaryan:** “All artifacts are synthetic. A real investigation would also require validated acquisition procedures and corroborating host-side evidence to establish who transferred data and when. Thank you.”

## Final rehearsal notes

- Keep the terminal in `~/forensic_case_study`.
- Do not rerun `dd`, `mkfs.vfat`, `rm`, or the setup/mount sequence against the existing image.
- The documented Foremost output path already exists. For the live demo, show its audit log and recovered files rather than overwriting it.
- Never read expected output as if it were observed. State “pending follow-up” unless a command has actually run successfully.
- Ensure the screenshot and report use the same paths and accurately reflect the completed run.
