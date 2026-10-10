# Digital Forensics Investigation Report

## 1. Case Information

| Field | Details |
| --- | --- |
| Case Title | USB Data Theft Investigation |
| Investigator(s) | Aaryan Sharma, Aditey Kshirsagar, Aditya Baheti |
| Evidence Type | USB storage device / forensic image |
| Tools Demonstrated | Foremost, ExifTool, srch_strings / Strings |
| Platform | Kali Linux |
| Objective | Determine whether sensitive data was copied to or stored on a USB device and recover relevant evidence. |

## 2. Investigation Scenario

An organization suspects that confidential files were copied from an internal system to a removable USB device. The USB device is seized and imaged. The original evidence is preserved, and all analysis is performed on a forensic image copy.

The investigation follows a standard forensic procedure:

1. Identification of evidence.
2. Preservation and hashing.
3. Acquisition of forensic image.
4. Examination with Kali tools.
5. Analysis of recovered artifacts.
6. Reporting and presentation.

## 3. Scope

The scope is limited to analyzing the USB forensic image for:

- Deleted or hidden files.
- Recoverable documents, images, PDFs, or archives.
- File metadata such as timestamps, author names, device/software traces, and GPS or camera data where applicable.
- Plaintext indicators such as usernames, project names, passwords, email addresses, internal filenames, or confidential keywords.

## 4. Evidence Handling

| Item ID | Description | Source | Hash Algorithm | Hash Value | Status |
| --- | --- | --- | --- | --- | --- |
| E01 | USB drive forensic image | Seized USB | SHA256 | `<insert hash>` | Working copy analyzed |

Recommended hashing commands:

```bash
sha256sum evidence/usb_image.dd
md5sum evidence/usb_image.dd
```

The original USB device should remain write-protected. Analysis should be performed only on a verified working copy.

## 5. Tool 1: Foremost

### Purpose

Foremost is used to recover deleted or embedded files from a disk image by scanning for known file headers, footers, and internal file structures.

### Demonstration Commands

```bash
mkdir -p output/foremost
foremost -v -t jpg,png,pdf,doc,xls,zip -i evidence/usb_image.dd -o output/foremost
ls -R output/foremost
cat output/foremost/audit.txt
```

### Expected Screenshots

- Foremost command execution.
- Output directory showing recovered file categories.
- `audit.txt` showing carved file summary.
- Sample recovered file preview.

### Analysis

Foremost helps identify deleted or hidden files that are no longer visible in the normal filesystem. Recovered files should be compared with the case keywords and reviewed for sensitive company data. The `audit.txt` file is important because it records recovered file names, offsets, and extraction results.

## 6. Tool 2: ExifTool

### Purpose

ExifTool extracts metadata from files. In a USB theft case, metadata may reveal author names, software used, creation and modification dates, device information, camera details, or document history.

### Demonstration Commands

```bash
exiftool output/foremost/pdf/*.pdf
exiftool output/foremost/jpg/*.jpg
exiftool -csv output/foremost/*/* > output/exiftool_metadata.csv
```

### Expected Screenshots

- Metadata extraction from one recovered file.
- CSV export command.
- Metadata fields showing timestamps, author/software, or device information.

### Analysis

Metadata can support the timeline of data movement. For example, a recovered document with an internal author field, recent modification timestamp, or software path may link the file to a suspect user or workstation. Image metadata may expose camera model, GPS data, or creation date.

## 7. Tool 3: srch_strings / Strings

### Purpose

`srch_strings` from Sleuth Kit, or GNU `strings`, extracts printable text from binary files and disk images. This helps locate keywords, URLs, usernames, passwords, document titles, and other readable evidence.

### Demonstration Commands

```bash
mkdir -p output/strings
srch_strings -a -t d evidence/usb_image.dd > output/strings/usb_strings.txt
grep -iE "confidential|password|project|invoice|client|secret" output/strings/usb_strings.txt
strings -a evidence/usb_image.dd | grep -iE "confidential|password|project|invoice|client|secret"
```

### Expected Screenshots

- Strings extraction command.
- Keyword search results.
- A text snippet showing relevant evidence.

### Analysis

Printable strings are useful when files are fragmented, deleted, or embedded in unallocated space. Keyword hits should be treated as leads and verified against recovered files, metadata, and filesystem timestamps.

## 8. Timeline Template

| Date/Time | Event | Source | Tool Used | Notes |
| --- | --- | --- | --- | --- |
| `<insert>` | USB image acquired | Evidence log | sha256sum | Hash recorded |
| `<insert>` | Files carved | USB image | Foremost | Recovered PDFs/images/docs |
| `<insert>` | Metadata extracted | Recovered files | ExifTool | Author/timestamps reviewed |
| `<insert>` | Keywords found | Disk image | srch_strings | Sensitive terms identified |

## 9. Findings

| Finding ID | Evidence | Tool | Observation | Significance |
| --- | --- | --- | --- | --- |
| F01 | Recovered files | Foremost | `<insert recovered file count and types>` | Indicates deleted/hidden files may have existed on USB |
| F02 | Metadata | ExifTool | `<insert author/time/device metadata>` | Supports attribution and timeline |
| F03 | Keyword strings | srch_strings | `<insert keyword hits>` | Supports presence of sensitive data |

## 10. Conclusion

The investigation demonstrates that Kali Linux forensic tools can recover deleted data, extract metadata, and identify readable evidence from a USB forensic image. The final conclusion should state whether the USB contained confidential data, whether recovered artifacts match the suspected data theft, and what evidence supports the finding.

## 11. Limitations

- Carved files may be incomplete if overwritten or fragmented.
- Metadata can be modified and should not be used alone for attribution.
- String matches are leads and require validation.
- A full forensic conclusion requires original evidence hashes, screenshots, recovered artifact review, and timeline correlation.
