# Video Presentation Script

Maximum duration: 10 minutes. Tools in investigation order: Strings → Foremost → ExifTool.

| Time | Section | Speaker |
| --- | --- | --- |
| 0:00 – 0:45 | Introduction and case | Aaryan Sharma |
| 0:45 – 1:45 | Forensic procedure and evidence preservation | Aditey Kshirsagar |
| 1:45 – 3:30 | Strings demonstration | Aditey Kshirsagar |
| 3:30 – 5:30 | Foremost demonstration | Aaryan Sharma |
| 5:30 – 7:00 | ExifTool demonstration | Aditya Baheti |
| 7:00 – 8:00 | Correlation: the missing metadata | Aditya Baheti |
| 8:00 – 9:00 | Findings and verdict | Aaryan Sharma |
| 9:00 – 9:40 | GitHub repository | Aditya Baheti |
| 9:40 – 10:00 | Limitations and close | All |

---

## 0:00 – 0:45 · Introduction — Aaryan Sharma

**Show:** Title slide: case ID USB-2026-001, team names, the three tools.

**Say:**
"We investigated a simulated case where an employee is suspected of copying
confidential company documents to a USB drive. The drive was seized and we
examined its image on Kali Linux using three tools: Strings, Foremost and
ExifTool. Our question is simple: does the evidence support the allegation or
not?"

## 0:45 – 1:45 · Forensic Procedure — Aditey Kshirsagar

**Show:** Screenshots 02, 03, 04 (files before deletion, deletion, image hash).

**Say:**
- "We followed identification, acquisition, preservation, examination,
  analysis and documentation."
- "For the lab we built a 50 MB FAT32 USB image. It holds two harmless files, a
  note and a photo, plus a confidential audit report that the suspect deleted."
- "Before any analysis we recorded the SHA-256 of the image, `74f073eb…`. We
  only use read-only tools, and at the end we check this hash again."

## 1:45 – 3:30 · Strings — Aditey Kshirsagar

**Show:** Run live or show screenshots 05, 06.
```bash
strings -a -t d usb_evidence.raw > output/all_strings.txt
grep -iE 'confidential|project_aurora|audit|finance|InternalOnly' output/all_strings.txt
```

**Say:**
- "Strings prints every readable text sequence in a binary file. `-a` scans the
  whole image and `-t d` prints the byte offset of each hit."
- "It ignores the filesystem, so it also reads deleted data."
- "The drive only shows a note and a photo, but we get nine hits: the title
  'CONFIDENTIAL INTERNAL AUDIT REPORT 2026', author 'Corporate Finance
  Directorate', and keywords 'Project_Aurora, InternalOnly'."
- "The offsets, 825111 to 826492, tell us exactly where on the drive this data
  sits. Keep these numbers in mind."

**Key point:** String hits are leads. We now need the actual file.

## 3:30 – 5:30 · Foremost — Aaryan Sharma

**Show:** Screenshots 07, 08, 09, 10.
```bash
foremost -v -t pdf,png -i usb_evidence.raw -o output/foremost_recovered
cat output/foremost_recovered/audit.txt
xxd -l 16 output/foremost_recovered/png/00001610.png
```

**Say:**
- "Foremost recovers files from their header and footer signatures. For PDF
  that is `%PDF` and `%%EOF`, and for PNG it is the 8-byte PNG signature."
- "`-t` selects file types, `-i` is the image, `-o` the output folder, `-v` is
  verbose."
- "It recovered two files. The PNG at offset 824320 is the visible decoy photo;
  `xxd` confirms the PNG signature. The PDF at offset 824832 is not on the
  drive's file list, so it was deleted."
- "Foremost names files by sector: 824832 divided by 512 is sector 1611."
- "One thing stands out: the recovered PDF is 616 bytes, but the original was
  about 4.5 KB."

## 5:30 – 7:00 · ExifTool — Aditya Baheti

**Show:** Screenshots 11, 12, 13.
```bash
exiftool output/foremost_recovered/pdf/00001611.pdf
exiftool -Title -Author -Subject -Keywords -CreateDate output/foremost_recovered/pdf/00001611.pdf
exiftool output/foremost_recovered/png/00001610.png
```

**Say:**
- "ExifTool reads embedded metadata such as title, author, keywords and
  creation date. That helps with attribution and timelines."
- "On the recovered PDF it shows PDF version 1.4 and one page, but when we ask
  for Title, Author, Subject, Keywords and CreateDate, it returns nothing."
- "The file dates it shows, 18:03:53, are when Foremost wrote the file, not
  when the document was created."
- "The PNG is a 1-by-1 image with no metadata."
- "So why does a confidential report have no metadata?"

## 7:00 – 8:00 · Correlation — Aditya Baheti

**Show:** Screenshot 14 (and 16 if the follow-up has been run). A simple
diagram works well here: bytes 824832 → 825448 (revision 1, recovered) |
825448 → 829401 (metadata update, missed).

**Say:**
- "ExifTool saves PDF metadata as an incremental update. It appends a second
  section with its own `%%EOF`."
- "Foremost stops at the first `%%EOF`, so it recovered only the first 616
  bytes. That part contains no metadata."
- "Strings found the metadata at offset 825545 onwards, right after where
  Foremost stopped. So the metadata is on the drive; carving just missed it."
- "Carving from the header to the second `%%EOF` gives a 4,569-byte file whose
  hash matches the company's original document, and ExifTool then shows the
  full title and author." *(Say this only after running COMMANDS.md section 9
  on Kali.)*

**Key point:** No single tool told the whole story. The answer came from
correlating all three.

## 8:00 – 9:00 · Findings and Verdict — Aaryan Sharma

**Show:** Findings table (REPORT.md section 13) and screenshot 15.

**Say:**
- "Strings: confidential title, author and keywords physically on the drive."
- "Foremost: a deleted PDF recovered from a drive that only showed decoys."
- "ExifTool plus hashes: the recovered file is the genuine first revision of
  that confidential report."
- "The image hash is the same before and after analysis, so the evidence was
  not altered."
- "Verdict: the evidence supports the allegation that confidential data was
  copied to the USB and deleted before seizure."

## 9:00 – 9:40 · GitHub Repository — Aditya Baheti

**Show:** Repository page: `REPORT.md`, `COMMANDS.md`, `tools/`, `evidence/`
hash files, `screenshots/`, commit history.

**Say:** "Everything is in our repository: the report, the full command log,
per-tool notes, hash records and screenshots, so the investigation can be
reproduced."

## 9:40 – 10:00 · Limitations and Close — All

- Aditey: "Carving can truncate files, as we saw, so recovered files must be
  verified."
- Aditya: "Metadata can be edited, so it supports attribution but does not prove
  it alone."
- Aaryan: "And this drive shows the data was there, but proving who copied it,
  and when, needs host evidence such as USB connection logs. Thank you."
