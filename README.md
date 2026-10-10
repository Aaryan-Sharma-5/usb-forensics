# USB Data Theft Investigation

Digital forensics mini-project demonstrating three Kali Linux tools (**Strings**,
**Foremost** and **ExifTool**) through a simulated USB data theft case.

## Team

| Member | Presentation role |
| --- | --- |
| Aaryan Sharma | Case introduction, Foremost, findings and verdict |
| Aditey Kshirsagar | Forensic procedure, Strings |
| Aditya Baheti | ExifTool, cross-tool correlation, GitHub walkthrough |

## Tools

| Tool | Package | Version | Role in the investigation |
| --- | --- | --- | --- |
| Strings | `binutils` | GNU strings 2.47 | Raw keyword search across the whole image, including deleted space |
| Foremost | `foremost` | 1.5.7 | File carving: recovers deleted files by header/footer signatures |
| ExifTool | `libimage-exiftool-perl` | 13.55 | Metadata extraction from recovered files |

## Case Summary

An employee is suspected of copying confidential company documents to a USB
drive. The drive is seized and imaged as `usb_evidence.raw` (50 MiB, FAT32).
Its visible contents are two harmless files, but a confidential audit report
had been written to it and then deleted.

| Tool | Result |
| --- | --- |
| Strings | Found "CONFIDENTIAL INTERNAL AUDIT REPORT 2026", "Corporate Finance Directorate" and "Project_Aurora, InternalOnly" at offsets 825111–826492 |
| Foremost | Recovered the deleted PDF (offset 824832) and the decoy PNG (offset 824320) |
| ExifTool | No metadata in the recovered PDF; correlation showed Foremost recovered only the first revision, and the metadata update that followed it on disk is where Strings found the hits |

**Verdict:** the evidence supports the allegation that confidential data was
stored on the USB drive and deleted before seizure. Image SHA-256
`74f073eb8c47789fd0adde67fd64d49da643bb6115968fe082fc8bccb2611afc` was unchanged
after analysis.

## Repository Contents

| Path | Contents |
| --- | --- |
| `REPORT.md` | Full investigation report with findings, timeline and conclusion |
| `COMMANDS.md` | Every command run on Kali, in order, with output |
| `tools/strings.md` | Strings options, results and interpretation |
| `tools/foremost.md` | Foremost options, results, truncation analysis and full-recovery follow-up |
| `tools/exiftool.md` | ExifTool results and interpretation |
| `CHAIN_OF_CUSTODY.md` | Evidence intake form, custody log, examination log, artefact register |
| `SCREENSHOT_CHECKLIST.md` | Required screenshots mapped to commands and report sections |
| `VIDEO_SCRIPT.md` | 10-minute presentation script with speaker assignments |
| `case/case-details.txt` | Case summary record |
| `environment.txt` | Platform and tool versions |
| `evidence/` | Hash records, tool versions, Foremost audit and ExifTool reports |
| `output/` | Strings dump and Foremost recovered files |
| `screenshots/` | Screenshots listed in the checklist |

## Reproducing

All commands are in `COMMANDS.md`. Run them from `~/forensic_case_study` on
Kali Linux after installing the tools:

```bash
sudo apt install -y foremost libimage-exiftool-perl binutils dosfstools xxd
```

## Note

All evidence is synthetic and was created for this lab. Do not upload real
confidential data, passwords or personal information.
