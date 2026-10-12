# USB Data Theft Investigation

Digital forensics mini-project demonstrating three Kali Linux tools (**Strings**, **Foremost** and **ExifTool**) through a controlled, synthetic USB forensic case study.

## Team

| Member | Presentation role |
| --- | --- |
| Aaryan Sharma | Case introduction, Foremost, findings and conclusion |
| Aditey Kshirsagar | Forensic procedure, Strings |
| Aditya Baheti | ExifTool, cross-tool correlation, GitHub walkthrough |

## Tools

| Tool | Package | Version recorded | Role |
| --- | --- | --- | --- |
| GNU Strings | `binutils` | GNU strings (GNU Binutils for Debian) 2.47 | Searches printable strings across the raw image and reports offsets |
| Foremost | `foremost` | 1.5.7 | Signature-based carving of PDF and PNG artifacts |
| ExifTool | `libimage-exiftool-perl` | 13.55 | Inspects metadata and file properties |

## Case summary

This is a **synthetic laboratory exercise**, not a real employee or seized-device investigation. A 50 MiB FAT32 filesystem image named `usb_evidence.raw` was populated with a note, a tiny PNG and a synthetic PDF with metadata. The PDF was then deleted from the mounted image to simulate a deletion scenario.

| Tool | Observed result |
| --- | --- |
| Strings | Found nine relevant printable strings, including the synthetic report title, author and Project Aurora keywords, at offsets 825111–826492 |
| Foremost | Recovered a 616-byte PDF fragment at offset 824832 and a 69-byte PNG at offset 824320 |
| ExifTool | Identified the recovered PDF as a one-page PDF 1.4 file but did not report the target descriptive metadata in that 616-byte fragment |
| Correlation | The metadata strings occur after the first PDF end marker and outside the 616-byte carve, consistent with an incomplete carve of a PDF using an incremental metadata update |

The image SHA-256 recorded for the completed run is:

`74f073eb8c47789fd0adde67fd64d49da643bb6115968fe082fc8bccb2611afc`

### Conclusion and limits

The observed artifacts demonstrate raw-string analysis, file carving, metadata inspection and hash verification. They do **not** prove a real unauthorized transfer, establish real-world attribution, or identify who copied a file or when. The optional manual recovery of the complete 4,569-byte PDF is documented in `COMMANDS.md` section 9; treat it as pending unless the commands have been run on the case image and the output has been verified.

## Repository contents

| Path | Contents |
| --- | --- |
| `REPORT.md` | Investigation report, findings, interpretation and limitations |
| `COMMANDS.md` | Preparation commands, completed analysis commands and separately marked follow-up |
| `tools/strings.md` | Strings options, results and interpretation |
| `tools/foremost.md` | Foremost output, truncation interpretation and follow-up carve |
| `tools/exiftool.md` | ExifTool observations and limitations |
| `CHAIN_OF_CUSTODY.md` | Evidence intake and artifact records for the simulation |
| `SCREENSHOT_CHECKLIST.md` | Screenshot names and mapping to commands |
| `VIDEO_SCRIPT.md` | Demo script and presenter assignments |
| `evidence/` | Hash records, tool versions, Foremost audit and metadata reports |
| `output/` | Strings dump and carved artifacts |
| `screenshots/` | Demonstration screenshots |

## Reproducing the lab

See `COMMANDS.md` for the complete procedure. The initial setup commands create and format the image; **do not rerun them against an existing case image**, because they would replace the evidence. Install required packages on a fresh lab only:

```bash
sudo apt update
sudo apt install -y foremost libimage-exiftool-perl binutils dosfstools xxd
```

All evidence in this repository was created for this educational exercise. Do not add real confidential data or personal information.
