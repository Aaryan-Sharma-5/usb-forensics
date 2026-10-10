# Strings — Raw Keyword Search

## Purpose
GNU `strings` prints every run of printable characters it finds in a binary
file. Run against a raw disk image it ignores the filesystem completely, so it
also reads deleted files, unallocated clusters and slack space. Each hit is
printed with its byte offset, which tells the investigator exactly where on the
media the text lives.

## Tool Details
- Package: `binutils`
- Version: GNU strings (GNU Binutils for Debian) 2.47
- Input: `usb_evidence.raw` (50 MiB FAT32 image, SHA-256 `74f073eb…2611afc`)
- Output: `output/all_strings.txt`

## Options Used
| Option | Meaning |
| --- | --- |
| `-a` | Scan the whole file, not only loaded data sections |
| `-t d` | Prefix every string with its decimal byte offset |
| default `-n 4` | Report runs of at least 4 printable characters |

## Commands
```bash
strings -a -t d usb_evidence.raw > output/all_strings.txt

grep -iE 'confidential|project_aurora|audit|finance|InternalOnly' \
  output/all_strings.txt
```

## Results
Nine keyword hits were found:

| Offset (decimal) | String | Location |
| --- | --- | --- |
| 825111 | `BT /F1 12 Tf 20 150 Td (Synthetic Confidential Audit Report) Tj ET` | PDF page content (inside the Foremost carve) |
| 825545 | `/Author (Corporate Finance Directorate)` | PDF Info dictionary (after the carve) |
| 825618 | `/Keywords (Project_Aurora, InternalOnly)` | PDF Info dictionary (after the carve) |
| 825659 | `/Subject (Synthetic Project Aurora Audit)` | PDF Info dictionary (after the carve) |
| 825701 | `/Title (CONFIDENTIAL INTERNAL AUDIT REPORT 2026)` | PDF Info dictionary (after the carve) |
| 826121 | `<rdf:li>Synthetic Project Aurora Audit</rdf:li>` | XMP metadata packet |
| 826229 | `<rdf:li xml:lang='x-default'>CONFIDENTIAL INTERNAL AUDIT REPORT 2026</rdf:li>` | XMP metadata packet |
| 826435 | `<pdf:Author>Corporate Finance Directorate</pdf:Author>` | XMP metadata packet |
| 826492 | `<pdf:Keywords>Project_Aurora, InternalOnly</pdf:Keywords>` | XMP metadata packet |

Strings was also run on the recovered file:

```bash
strings -a output/foremost_recovered/pdf/00001611.pdf \
  | grep -iE 'confidential|project_aurora|audit|finance|InternalOnly'
# BT /F1 12 Tf 20 150 Td (Synthetic Confidential Audit Report) Tj ET
```

Only the page text appears in the recovered PDF. A full `strings -a` dump of
it shows its trailer is `<< /Size 6 /Root 1 0 R >>`, with no `/Info`
dictionary.

## Interpretation
- The visible drive contains only `personal_notes.txt` and `beach_trip.png`,
  yet confidential-classification text is physically present on the media.
- Offset 825111 falls inside the PDF that Foremost later carved (bytes
  824832–825447). The other eight hits sit after the carved file ends. They
  belong to the metadata update that ExifTool appended to the PDF, which
  Foremost did not recover (see `tools/foremost.md`).
- Without these offsets, the empty ExifTool result on the recovered PDF would
  look like "no metadata". Strings proves the metadata exists on the drive.

## Limitations
- A string hit is a lead, not proof of a complete file. Hits must be tied to a
  recovered file or an offset range.
- Compressed content (for example DOCX or deflated PDF streams) is not readable
  by `strings`.
