# Digital Forensics Investigation Report

## 1. Case Information

| Field | Details |
| --- | --- |
| Case ID | USB-2026-001 |
| Case Title | Digital Forensic Investigation of Suspected Data Theft Through a USB Device |
| Investigators | Aaryan Sharma, Aditey Kshirsagar, Aditya Baheti |
| Evidence | `usb_evidence.raw`: 50 MiB FAT32 image of the seized USB drive (synthetic) |
| Tools Demonstrated | Strings, Foremost, ExifTool |
| Platform | Kali Linux Rolling on WSL 2 |
| Examination Date | 10 October 2026 |
| Verdict | Evidence **supports** the allegation that confidential data was stored on the USB device and deleted before seizure |

## 2. Executive Summary

The seized USB drive showed only two harmless files: a personal note and a
holiday photo. Analysis of the raw image found a deleted PDF titled
**CONFIDENTIAL INTERNAL AUDIT REPORT 2026**, authored by the **Corporate
Finance Directorate** and tagged **Project_Aurora, InternalOnly**.

- **Strings** found the confidential title, author and keywords at known byte
  offsets on the drive.
- **Foremost** recovered the deleted PDF and the decoy PNG from the image.
- **ExifTool** found no metadata in the recovered PDF. Cross-checking showed
  why: Foremost recovered only the document's first revision. The metadata
  update that followed it on disk was confirmed by Strings.

Together the three tools show that a confidential organisational document was
present on the USB device and was deleted to hide it.

## 3. Investigation Scenario

A company suspects that an employee copied confidential organisational
documents to a USB storage device without authorisation. The device is seized
and preserved as digital evidence. The investigation must identify relevant
files, examine their properties and contents, recover deleted information, and
correlate the evidence to decide whether unauthorised data transfer occurred.

## 4. Scope and Objectives

- Identify confidential indicators on the device, including in deleted and
  unallocated areas.
- Recover deleted files without relying on the filesystem.
- Extract metadata that shows document origin, classification and dates.
- Correlate results across tools and reach a supported conclusion.

Out of scope: host-side artefacts (registry USB history, LNK files), which
would be needed to prove *who* copied the data and *when*.

## 5. Forensic Procedure

| Phase | Action in this case |
| --- | --- |
| Identification | USB drive identified as evidence item E01 |
| Acquisition | Bit-for-bit image `usb_evidence.raw` (52,428,800 bytes) |
| Preservation | SHA-256 recorded before analysis; image unmounted and loop device detached; only read-only tools used |
| Examination | Strings (raw keyword search), Foremost (carving), ExifTool (metadata) |
| Analysis | Offsets, hashes and metadata correlated across tools |
| Documentation | Command log, hash records, audit file, metadata reports, screenshots, this report |

## 6. Evidence Preparation (Synthetic Case Data)

No real evidence is used. The seized drive was simulated as follows (full
commands in `COMMANDS.md`):

1. A 50 MiB blank image was formatted FAT32 (label `EVIDENCE`) and mounted.
2. Two decoy files were written: `personal_notes.txt` (65 B) and
   `beach_trip.png` (69 B, 1×1 pixel).
3. A confidential PDF `Internal_Financial_Audit_2026.pdf` was generated, and
   ExifTool added its metadata (Title, Author, Subject, Keywords, CreateDate).
   The file grew from 616 B to 4,569 B.
4. The PDF was copied to the drive and then deleted with `rm`, simulating the
   suspect hiding it.
5. The image was unmounted and hashed.

After deletion, a normal directory listing shows only the two decoy files.

## 7. Evidence Handling and Integrity

| Item | Description | SHA-256 | Status |
| --- | --- | --- | --- |
| E01 | `usb_evidence.raw`: seized USB image | `74f073eb8c47789fd0adde67fd64d49da643bb6115968fe082fc8bccb2611afc` | Verified unchanged after analysis |
| R01 | `output/foremost_recovered/pdf/00001611.pdf`: recovered PDF | `6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275` | Verified (`sha256sum -c`: OK) |
| R02 | `output/foremost_recovered/png/00001610.png`: recovered PNG | `fdf040aaee9270aef6cf03752d51615cf6025c72081820fe0945f0c9c9cbdd9c` | Verified (`sha256sum -c`: OK) |
| REF01 | `/tmp/Internal_Financial_Audit_2026.pdf`: company's original document (reference copy) | `cec3f6868336b0495c8216dc6bfd90e1c48c2052be6f2884c473c8c8e9ad454c` | Reference only |

Hash records: `evidence/evidence_integrity.sha256`,
`evidence/recovered_files.sha256`, `evidence/final_hashes.sha256`.

## 8. Tool 1: Strings

**Purpose.** Extracts printable text from any binary file. On a raw image it
reads deleted and unallocated data that the filesystem no longer shows.

**Commands.**
```bash
strings -a -t d usb_evidence.raw > output/all_strings.txt
grep -iE 'confidential|project_aurora|audit|finance|InternalOnly' output/all_strings.txt
```
`-a` scans the whole file; `-t d` prints each string's decimal byte offset.

**Results.** Nine hits, including:

| Offset | String |
| --- | --- |
| 825111 | `(Synthetic Confidential Audit Report)` (page text) |
| 825545 | `/Author (Corporate Finance Directorate)` |
| 825618 | `/Keywords (Project_Aurora, InternalOnly)` |
| 825659 | `/Subject (Synthetic Project Aurora Audit)` |
| 825701 | `/Title (CONFIDENTIAL INTERNAL AUDIT REPORT 2026)` |
| 826121–826492 | Same values repeated in an XMP metadata packet |

**Analysis.** Confidential-classification text is physically present on a
drive whose visible files are harmless. This is the first indicator of hidden
or deleted content. Details: `tools/strings.md`.

**Screenshots.** 05, 06

## 9. Tool 2: Foremost

**Purpose.** Recovers files by their header/footer signatures, independent of
the filesystem, so deleted files can be restored.

**Command.**
```bash
foremost -v -t pdf,png -i usb_evidence.raw -o output/foremost_recovered
```

**Results.**

| File | Size | Offset | Meaning |
| --- | --- | --- | --- |
| `pdf/00001611.pdf` | 616 B | 824832 (sector 1611) | Deleted PDF, not visible on the drive |
| `png/00001610.png` | 69 B | 824320 (sector 1610) | Visible decoy `beach_trip.png` (PNG signature verified with `xxd`) |

**Analysis.** Foremost recovered a PDF that the directory listing does not
show, which proves a document was deleted from the drive. The recovered PDF is
616 bytes, but the original was 4,569 bytes. This is explained in Section 11.
Details: `tools/foremost.md`.

**Screenshots.** 07, 08, 09, 10

## 10. Tool 3: ExifTool

**Purpose.** Reads embedded metadata (PDF Info, XMP, EXIF) for attribution and
timeline evidence.

**Commands.**
```bash
exiftool output/foremost_recovered/pdf/00001611.pdf
exiftool -Title -Author -Subject -Keywords -CreateDate output/foremost_recovered/pdf/00001611.pdf
exiftool output/foremost_recovered/png/00001610.png
```

**Results.**
- Recovered PDF: PDF 1.4, 1 page, not linearised. **No** Title, Author,
  Subject, Keywords or CreateDate.
- Recovered PNG: 1×1, 8-bit RGB, no embedded metadata.
- The `File Modification Date/Time` (2026-10-10 18:03:53 IST) is when Foremost
  wrote the recovered file, not when the document was created.

**Analysis.** An empty metadata result is itself a finding that needs an
explanation. It led to the cross-tool correlation in Section 11. Details:
`tools/exiftool.md`.

**Screenshots.** 11, 12, 13

## 11. Correlation: Why the Recovered PDF Has No Metadata

1. ExifTool saves PDF metadata as an **incremental update**: it appends a new
   Info dictionary, an XMP packet and a new cross-reference table, ending in a
   second `%%EOF`. The original document is therefore two revisions back to
   back:

   | Bytes on disk | Content |
   | --- | --- |
   | 824832 – 825447 | Revision 1: page content, ends with the first `%%EOF` |
   | 825448 – 829400 | Revision 2: ExifTool's Info dictionary + XMP, ends with the second `%%EOF` |

2. Foremost stops a PDF at the **first** `%%EOF`, so it recovered only
   revision 1: 824832 + 616 = 825448.
3. The trailer of the recovered file is `<< /Size 6 /Root 1 0 R >>`. It has no
   `/Info` entry, which is why ExifTool reported no document metadata.
4. Strings found the metadata at offsets **825545–826492**, which is inside
   revision 2, immediately after Foremost's carve. The metadata is on the
   drive; carving missed it.
5. The first 616 bytes of the original document hash to `6001609b…9275`, the
   exact hash of the recovered file. So the recovered PDF is genuine and
   unaltered, just incomplete.

**Follow-up (verified on a replica image, to be run on Kali).** Carving from
the PDF header to the second `%%EOF` (4,569 bytes) recovers a file whose
SHA-256 equals the company's original document (`cec3f686…454c`), and
ExifTool then shows the full Title, Author, Subject, Keywords and CreateDate.
Commands: `COMMANDS.md` section 9.

## 12. Timeline

| Date/Time (IST) | Event | Source |
| --- | --- | --- |
| 2026-09-15 10:30:00 | Confidential audit report created | PDF metadata (CreateDate) of the reference copy, via ExifTool |
| 2026-10-10 18:02 | `personal_notes.txt` and `beach_trip.png` written to the drive | Directory listing during preparation |
| 2026-10-10 18:03 | `Internal_Financial_Audit_2026.pdf` written to the drive, then deleted | Directory listing before and after `rm` |
| 2026-10-10 (after unmount) | Image sealed, SHA-256 `74f073eb…2611afc` recorded | `evidence/evidence_integrity.sha256` |
| 2026-10-10 18:03:53 | Foremost recovers the PDF and PNG | `evidence/foremost_audit.txt` |
| 2026-10-10 (end of session) | Image hash re-verified, unchanged | `evidence/final_hashes.sha256` |

## 13. Findings

| ID | Evidence | Tool | Observation | Significance |
| --- | --- | --- | --- | --- |
| F01 | Raw image | Strings | Title "CONFIDENTIAL INTERNAL AUDIT REPORT 2026", Author "Corporate Finance Directorate", Keywords "Project_Aurora, InternalOnly" at offsets 825545–826492 | Confidential company data is physically present on the drive |
| F02 | Raw image | Foremost | Deleted PDF recovered at offset 824832; PNG decoy at 824320 | A document was deleted from the drive to hide it |
| F03 | Recovered PDF | ExifTool | No document metadata in the 616-byte recovered file | Carving was incomplete, not a sign of a clean file |
| F04 | Offsets + hashes | Strings + Foremost + ExifTool | Metadata lies directly after the carve end; recovered file = first 616 bytes of the original | Links the recovered file and the metadata to the same confidential document |
| F05 | Raw image | sha256sum | Image hash identical before and after analysis | Evidence integrity preserved |

## 14. Conclusion

The visible contents of the USB drive were harmless decoys. The three tools
together show that a confidential document belonging to the Corporate Finance
Directorate (Project Aurora, internal-only) was stored on the device and then
deleted:

- Strings located the confidential title, author and keywords on the media.
- Foremost recovered the deleted document itself.
- ExifTool's empty result, correlated with Strings offsets and hashes, showed
  the recovered file is the genuine first revision of that document.

**Verdict: the evidence supports the allegation that confidential
organisational data was copied to the USB device and deleted before seizure.**

These artefacts alone do not identify *who* copied the file or *when*. That
would require host-side evidence (USB connection history, user logon records).

## 15. Limitations

- All data is synthetic and was created for this lab.
- Signature carving can truncate files that have several end markers (as here)
  or that are fragmented.
- Metadata can be edited and supports attribution only alongside other evidence.
- Strings hits are leads that must be tied to recovered files or offset ranges.
- The full-file recovery in Section 11 is verified on a replica image and must
  be repeated on the Kali image before submission.

## Appendix A: Tool Versions

| Tool | Package | Version |
| --- | --- | --- |
| Strings | binutils | GNU strings (GNU Binutils for Debian) 2.47 |
| Foremost | foremost | 1.5.7 |
| ExifTool | libimage-exiftool-perl | 13.55 |

## Appendix B: Supporting Files

- `COMMANDS.md`: full command log with outputs
- `tools/strings.md`, `tools/foremost.md`, `tools/exiftool.md`: per-tool detail
- `CHAIN_OF_CUSTODY.md`: custody and examination forms
- `SCREENSHOT_CHECKLIST.md`: screenshot index
