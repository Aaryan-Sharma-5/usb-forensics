# Chain of Custody and Investigation Forms

Case ID: **USB-2026-001**. All evidence is synthetic and was created for this lab.

## 1. Evidence Intake Form

| Field | Entry |
| --- | --- |
| Evidence ID | E01 |
| Description | USB storage device, imaged as `usb_evidence.raw` |
| Image format / size | Raw (dd), 52,428,800 bytes (50 MiB), FAT32 |
| Acquired by | Aaryan Sharma |
| Date / time | 2026-10-10, after 18:03 IST |
| Location | Kali Linux (WSL 2) forensic workstation, `~/forensic_case_study` |
| Acquisition hash (SHA-256) | `74f073eb8c47789fd0adde67fd64d49da643bb6115968fe082fc8bccb2611afc` |
| Hash record | `evidence/evidence_integrity.sha256` |
| Write protection | Image unmounted and loop device detached before analysis; only read-only tools used |
| Signature | `<signature>` |

## 2. Chain of Custody Log

| # | Evidence ID | Description | Released by | Received by | Date/Time (IST) | Purpose | SHA-256 | Signature |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | E01 | USB image `usb_evidence.raw` | Seizure / preparation | Aaryan Sharma | 2026-10-10 18:03 | Acquisition and hashing | `74f073eb…2611afc` | `<signature>` |
| 2 | E01 | USB image `usb_evidence.raw` | Aaryan Sharma | Analysis workspace | 2026-10-10 18:03 | Examination with Strings, Foremost, ExifTool | `74f073eb…2611afc` | `<signature>` |
| 3 | R01 | Recovered PDF `00001611.pdf` | Foremost output | Aaryan Sharma | 2026-10-10 18:03:53 | Metadata and content review | `6001609b…9275` | `<signature>` |
| 4 | R02 | Recovered PNG `00001610.png` | Foremost output | Aaryan Sharma | 2026-10-10 18:03:53 | Signature and metadata review | `fdf040aa…dd9c` | `<signature>` |
| 5 | E01, R01, R02 | Hash re-verification | Aaryan Sharma | Case file | 2026-10-10 | Integrity check (`evidence/final_hashes.sha256`) | unchanged | `<signature>` |
| 6 | Case file | Report, logs, hashes | Aaryan Sharma | Aditey Kshirsagar, Aditya Baheti | `<date/time>` | Review and presentation | n/a | `<signature>` |

## 3. Examination Log

| Step | Tool | Input | Command | Output | Result |
| --- | --- | --- | --- | --- | --- |
| 1 | sha256sum | E01 | `sha256sum usb_evidence.raw` | `evidence/evidence_integrity.sha256` | Baseline hash recorded |
| 2 | Strings | E01 | `strings -a -t d usb_evidence.raw` | `output/all_strings.txt` | 9 keyword hits, offsets 825111–826492 |
| 3 | Foremost | E01 | `foremost -v -t pdf,png -i usb_evidence.raw -o output/foremost_recovered` | `output/foremost_recovered/`, `evidence/foremost_audit.txt` | 1 PDF + 1 PNG recovered |
| 4 | xxd / od | R02 | `xxd -l 16 …/00001610.png` | screen | Valid PNG signature |
| 5 | ExifTool | R01 | `exiftool …/00001611.pdf` | `evidence/recovered_pdf_metadata.txt` | PDF 1.4, 1 page, no document metadata |
| 6 | ExifTool | R02 | `exiftool …/00001610.png` | `evidence/recovered_png_metadata.txt` | 1×1 RGB, no metadata |
| 7 | Strings + sha256sum | R01, REF01 | `strings -a`, `sha256sum` | screen | Recovered PDF = first revision of original |
| 8 | sha256sum | E01, R01, R02 | `sha256sum -c evidence/recovered_files.sha256` | `evidence/final_hashes.sha256` | All OK; E01 unchanged |

## 4. Recovered Artefact Register

| ID | File | Source offset | Size | SHA-256 | Notes |
| --- | --- | --- | --- | --- | --- |
| R01 | `output/foremost_recovered/pdf/00001611.pdf` | 824832 | 616 B | `6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275` | Deleted confidential PDF, first revision only |
| R02 | `output/foremost_recovered/png/00001610.png` | 824320 | 69 B | `fdf040aaee9270aef6cf03752d51615cf6025c72081820fe0945f0c9c9cbdd9c` | Visible decoy `beach_trip.png` |
| R03 | `output/manual_carve/audit_report_full.pdf` | 824832 | 4,569 B | expected `cec3f686…454c` | Pending: follow-up carve on Kali |

## Handling Rules

- Hash every item when it is acquired and again after analysis.
- Analyse only the image, never the original device.
- Record every transfer: who, when, why, and the hash at that moment.
- Keep tool outputs in separate `output/` folders; never write into the image.
