# ExifTool — Metadata Analysis

## Purpose
ExifTool reads (and writes) metadata embedded in files: PDF Info dictionaries,
XMP packets, EXIF in images and Office document properties. In a data-theft
case, metadata can show a document's title, author, classification keywords and
creation date, which helps with attribution and timeline building.

## Tool Details
- Package: `libimage-exiftool-perl`
- Version: 13.55
- Input: files recovered by Foremost

## Commands and Results

### 1. Full metadata dump of the recovered PDF
```bash
exiftool output/foremost_recovered/pdf/00001611.pdf
```
```text
File Size                       : 616 bytes
File Modification Date/Time     : 2026:10:10 18:03:53+05:30
File Type                       : PDF
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Page Count                      : 1
```

### 2. Targeted attribution query
```bash
exiftool -Title -Author -Subject -Keywords -CreateDate \
  output/foremost_recovered/pdf/00001611.pdf
```
Result: no output. None of these tags exist in the recovered file.

### 3. Recovered PNG
```bash
exiftool output/foremost_recovered/png/00001610.png
```
```text
File Type                       : PNG
Image Width                     : 1
Image Height                    : 1
Bit Depth                       : 8
Color Type                      : RGB
Image Size                      : 1x1
```
The image has no EXIF, XMP or text metadata.

### 4. Reference: the prepared original document
Before the PDF was copied to the drive, its metadata was validated:
```bash
exiftool -Title -Author -Subject -Keywords -CreateDate \
  /tmp/Internal_Financial_Audit_2026.pdf
```
```text
Title                           : CONFIDENTIAL INTERNAL AUDIT REPORT 2026
Author                          : Corporate Finance Directorate
Subject                         : Synthetic Project Aurora Audit
Keywords                        : Project_Aurora, InternalOnly
Create Date                     : 2026:09:15 10:30:00
```

### 5. Saved reports
```bash
exiftool output/foremost_recovered/pdf/00001611.pdf > evidence/recovered_pdf_metadata.txt
exiftool output/foremost_recovered/png/00001610.png > evidence/recovered_png_metadata.txt
```

## Interpretation
- The empty result does **not** mean the document had no metadata. ExifTool
  wrote the metadata as an incremental update appended after the first
  `%%EOF`, and Foremost stopped carving at that first `%%EOF`. The recovered
  616-byte file is the original revision, whose trailer has no `/Info` entry.
- `strings` found the same Title, Author, Subject and Keywords values on the raw
  image at offsets 825545–826492, just past the end of the carved file. So the
  metadata does exist on the USB.
- The `File Modification Date/Time` (18:03:53) is the moment Foremost wrote the
  recovered file. It is not a timestamp of the original document.

## Follow-up
After the complete PDF is carved (see `tools/foremost.md`), run:
```bash
exiftool -Title -Author -Subject -Keywords -CreateDate \
  output/manual_carve/audit_report_full.pdf
```
Expected (verified on a replica image): the same five values as the reference
document above.

## Limitations
- Metadata can be edited, so it supports attribution but cannot prove it alone.
- ExifTool reads only what survives in the recovered file, so its results
  depend on how complete the carving was.
