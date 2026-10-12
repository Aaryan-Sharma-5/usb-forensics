# Command Log

Commands executed on Kali Linux Rolling (WSL 2) in `~/forensic_case_study`,
in the order they were run, with the key output. Screenshot file names refer
to `SCREENSHOT_CHECKLIST.md`.

All evidence is synthetic and was created for this lab.

---

## 0. Setup (run before the logged session)

### Tool installation
```bash
sudo apt update
sudo apt install -y foremost libimage-exiftool-perl binutils dosfstools
```

### Workspace and blank USB image
The image size (52,428,800 bytes) and the carving offsets below are consistent
with these preparation steps.

```bash
mkdir -p ~/forensic_case_study/{evidence,output}
cd ~/forensic_case_study

dd if=/dev/zero of=usb_evidence.raw bs=1M count=50
LOOP_DEV=$(sudo losetup -fP --show usb_evidence.raw)
sudo mkfs.vfat -F 32 -n "EVIDENCE" "$LOOP_DEV"
sudo mkdir -p /mnt/usb_sim
sudo mount "$LOOP_DEV" /mnt/usb_sim
```

---

## 1. Plant the Synthetic Evidence

### 1a. Decoy note and decoy image
```bash
echo "Meeting notes: Discussing annual office party and holiday plans." \
  | sudo tee /mnt/usb_sim/personal_notes.txt

python3 - <<'PY'
import struct, zlib

def chunk(kind, data):
    body = kind + data
    return (struct.pack(">I", len(data)) + body +
            struct.pack(">I", zlib.crc32(body) & 0xffffffff))

png = (
    b"\x89PNG\r\n\x1a\n"
    + chunk(b"IHDR", struct.pack(">IIBBBBB", 1, 1, 8, 2, 0, 0, 0))
    + chunk(b"IDAT", zlib.compress(b"\x00\x00\x80\xff"))
    + chunk(b"IEND", b"")
)
with open("/tmp/beach_trip.png", "wb") as f:
    f.write(png)
PY

sudo cp /tmp/beach_trip.png /mnt/usb_sim/beach_trip.png
```

### 1b. Confidential PDF (valid xref table so ExifTool can read and write it)
```bash
python3 - <<'PY'
from pathlib import Path

out = Path("/tmp/Internal_Financial_Audit_2026.pdf")
objects = [
    b"<< /Type /Catalog /Pages 2 0 R >>",
    b"<< /Type /Pages /Kids [3 0 R] /Count 1 >>",
    b"<< /Type /Page /Parent 2 0 R /MediaBox [0 0 300 200] /Contents 4 0 R /Resources << /Font << /F1 5 0 R >> >> >>",
    b"<< /Length 85 >>\nstream\nBT /F1 12 Tf 20 150 Td (Synthetic Confidential Audit Report) Tj ET\nendstream",
    b"<< /Type /Font /Subtype /Type1 /BaseFont /Helvetica >>",
]
data = bytearray(b"%PDF-1.4\n%\xe2\xe3\xcf\xd3\n")
offsets = [0]
for i, obj in enumerate(objects, 1):
    offsets.append(len(data))
    data.extend(f"{i} 0 obj\n".encode())
    data.extend(obj)
    data.extend(b"\nendobj\n")
xref_offset = len(data)
data.extend(f"xref\n0 {len(objects)+1}\n".encode())
data.extend(b"0000000000 65535 f \n")
for offset in offsets[1:]:
    data.extend(f"{offset:010d} 00000 n \n".encode())
data.extend(
    f"trailer\n<< /Size {len(objects)+1} /Root 1 0 R >>\n"
    f"startxref\n{xref_offset}\n%%EOF\n".encode()
)
out.write_bytes(data)
print(f"Created {out} ({len(data)} bytes)")
PY
```
Output: `Created /tmp/Internal_Financial_Audit_2026.pdf (616 bytes)`

### 1c. Add document metadata with ExifTool
```bash
exiftool -overwrite_original \
  -Title="CONFIDENTIAL INTERNAL AUDIT REPORT 2026" \
  -Author="Corporate Finance Directorate" \
  -Subject="Synthetic Project Aurora Audit" \
  -Keywords="Project_Aurora, InternalOnly" \
  -CreateDate="2026:09:15 10:30:00" \
  /tmp/Internal_Financial_Audit_2026.pdf

exiftool -Title -Author -Subject -Keywords -CreateDate \
  /tmp/Internal_Financial_Audit_2026.pdf
```
```text
    1 image files updated
Title                           : CONFIDENTIAL INTERNAL AUDIT REPORT 2026
Author                          : Corporate Finance Directorate
Subject                         : Synthetic Project Aurora Audit
Keywords                        : Project_Aurora, InternalOnly
Create Date                     : 2026:09:15 10:30:00
```
ExifTool appends this metadata as a PDF incremental update, so the file grows
from 616 bytes to 4,569 bytes (shown as 4.5K below).

### 1d. Copy the PDF to the drive
```bash
sudo cp /tmp/Internal_Financial_Audit_2026.pdf \
  /mnt/usb_sim/Internal_Financial_Audit_2026.pdf
sync
sudo ls -lh /mnt/usb_sim
```
```text
-rwxr-xr-x 1 root root   69 Oct 10 18:02 beach_trip.png
-rwxr-xr-x 1 root root 4.5K Oct 10 18:03 Internal_Financial_Audit_2026.pdf
-rwxr-xr-x 1 root root   65 Oct 10 18:02 personal_notes.txt
```
Screenshot: `02_files_before_deletion.png`

---

## 2. Simulate Deletion (Anti-Forensics) and Seal the Image

```bash
sudo rm /mnt/usb_sim/Internal_Financial_Audit_2026.pdf
sync
sudo ls -lh /mnt/usb_sim

sudo umount /mnt/usb_sim
sudo losetup -d "$LOOP_DEV"
mount | grep usb_sim || echo "Synthetic filesystem unmounted"
```
```text
-rwxr-xr-x 1 root root 69 Oct 10 18:02 beach_trip.png
-rwxr-xr-x 1 root root 65 Oct 10 18:02 personal_notes.txt
Synthetic filesystem unmounted
```
Screenshot: `03_deletion_and_unmount.png`

---

## 3. Preserve the Evidence

```bash
sha256sum usb_evidence.raw | tee evidence/evidence_integrity.sha256
stat -c 'Image size: %s bytes' usb_evidence.raw
```
```text
74f073eb8c47789fd0adde67fd64d49da643bb6115968fe082fc8bccb2611afc  usb_evidence.raw
Image size: 52428800 bytes
```
`file usb_evidence.raw` failed because `file` is not installed on minimal
Kali. Install it with `sudo apt install file`.

Screenshot: `04_image_hash.png`

---

## 4. Tool 1: Strings

```bash
strings -a -t d usb_evidence.raw > output/all_strings.txt

grep -iE 'confidential|project_aurora|audit|finance|InternalOnly' \
  output/all_strings.txt
```
```text
 825111 BT /F1 12 Tf 20 150 Td (Synthetic Confidential Audit Report) Tj ET
 825545 /Author (Corporate Finance Directorate)
 825618 /Keywords (Project_Aurora, InternalOnly)
 825659 /Subject (Synthetic Project Aurora Audit)
 825701 /Title (CONFIDENTIAL INTERNAL AUDIT REPORT 2026)
 826121     <rdf:li>Synthetic Project Aurora Audit</rdf:li>
 826229     <rdf:li xml:lang='x-default'>CONFIDENTIAL INTERNAL AUDIT REPORT 2026</rdf:li>
 826435   <pdf:Author>Corporate Finance Directorate</pdf:Author>
 826492   <pdf:Keywords>Project_Aurora, InternalOnly</pdf:Keywords>
```
Screenshots: `05_strings_extraction.png`, `06_strings_keyword_hits.png`

---

## 5. Tool 2: Foremost

```bash
foremost -v \
  -t pdf,png \
  -i usb_evidence.raw \
  -o output/foremost_recovered
```
```text
Foremost version 1.5.7 by Jesse Kornblum, Kris Kendall, and Nick Mikus
Invocation: foremost -v -t pdf,png -i usb_evidence.raw -o output/foremost_recovered
Configuration file: /etc/foremost.conf
Length: 50 MB (52428800 bytes)

Num      Name (bs=512)         Size      File Offset     Comment
0:      00001611.pdf          616 B          824832
1:      00001610.png           69 B          824320       (1 x 1)

2 FILES EXTRACTED
pdf:= 1
png:= 1
```
Screenshot: `07_foremost_execution.png`

```bash
cat output/foremost_recovered/audit.txt
find output/foremost_recovered -type f -printf '%p (%s bytes)\n'
ls -lh output/foremost_recovered/pdf/
ls -lh output/foremost_recovered/png/
```
```text
output/foremost_recovered/png/00001610.png (69 bytes)
output/foremost_recovered/pdf/00001611.pdf (616 bytes)
output/foremost_recovered/audit.txt (821 bytes)
```
Screenshots: `08_foremost_audit.png`, `09_foremost_recovered_files.png`

### PNG signature check
```bash
sudo apt install xxd
xxd -l 16 output/foremost_recovered/png/00001610.png
od -An -tx1 -N16 output/foremost_recovered/png/00001610.png
```
```text
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
 89 50 4e 47 0d 0a 1a 0a 00 00 00 0d 49 48 44 52
```
Screenshot: `10_png_signature.png`

### Hash the recovered files
```bash
sha256sum \
  output/foremost_recovered/pdf/00001611.pdf \
  output/foremost_recovered/png/00001610.png \
  | tee evidence/recovered_files.sha256
```
```text
6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275  output/foremost_recovered/pdf/00001611.pdf
fdf040aaee9270aef6cf03752d51615cf6025c72081820fe0945f0c9c9cbdd9c  output/foremost_recovered/png/00001610.png
```

---

## 6. Tool 3: ExifTool

```bash
exiftool output/foremost_recovered/pdf/00001611.pdf
```
```text
ExifTool Version Number         : 13.55
File Name                       : 00001611.pdf
File Size                       : 616 bytes
File Modification Date/Time     : 2026:10:10 18:03:53+05:30
File Type                       : PDF
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Page Count                      : 1
```
Screenshot: `11_exiftool_recovered_pdf.png`

```bash
exiftool -Title -Author -Subject -Keywords -CreateDate \
  output/foremost_recovered/pdf/00001611.pdf
```
Result: no output. The recovered PDF has no Title, Author, Subject, Keywords
or CreateDate.

Screenshot: `12_exiftool_targeted_query_empty.png`

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
Screenshot: `13_exiftool_recovered_png.png`

---

## 7. Cross-Checking the Recovered PDF

```bash
strings -a output/foremost_recovered/pdf/00001611.pdf \
  | grep -iE 'confidential|project_aurora|audit|finance|InternalOnly'
```
```text
BT /F1 12 Tf 20 150 Td (Synthetic Confidential Audit Report) Tj ET
```

```bash
exiftool -Title -Author -Subject -Keywords -CreateDate \
  /tmp/Internal_Financial_Audit_2026.pdf
strings -a output/foremost_recovered/pdf/00001611.pdf
sha256sum \
  /tmp/Internal_Financial_Audit_2026.pdf \
  output/foremost_recovered/pdf/00001611.pdf
```
```text
cec3f6868336b0495c8216dc6bfd90e1c48c2052be6f2884c473c8c8e9ad454c  /tmp/Internal_Financial_Audit_2026.pdf
6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275  output/foremost_recovered/pdf/00001611.pdf
```
The hashes differ. The recovered PDF's trailer is `<< /Size 6 /Root 1 0 R >>`
with no `/Info` dictionary, so it is the first revision of the document,
before the ExifTool update.

Screenshot: `14_original_vs_recovered_hash.png`

---

## 8. Record Versions, Reports and Final Hashes

```bash
{
  echo "=== Foremost ==="
  foremost -V 2>&1 | head -n 1
  echo
  echo "=== ExifTool ==="
  exiftool -ver
  echo
  echo "=== Strings ==="
  strings -v 2>&1 | head -n 1
} | tee evidence/tool_versions.txt

cp output/foremost_recovered/audit.txt evidence/foremost_audit.txt
exiftool output/foremost_recovered/pdf/00001611.pdf > evidence/recovered_pdf_metadata.txt
exiftool output/foremost_recovered/png/00001610.png > evidence/recovered_png_metadata.txt

sha256sum \
  usb_evidence.raw \
  output/foremost_recovered/pdf/00001611.pdf \
  output/foremost_recovered/png/00001610.png \
  | tee evidence/final_hashes.sha256

sha256sum -c evidence/recovered_files.sha256
sha256sum -c evidence/evidence_integrity.sha256
```
```text
=== Foremost ===
1.5.7

=== ExifTool ===
13.55

=== Strings ===
GNU strings (GNU Binutils for Debian) 2.47
74f073eb8c47789fd0adde67fd64d49da643bb6115968fe082fc8bccb2611afc  usb_evidence.raw
6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275  output/foremost_recovered/pdf/00001611.pdf
fdf040aaee9270aef6cf03752d51615cf6025c72081820fe0945f0c9c9cbdd9c  output/foremost_recovered/png/00001610.png
output/foremost_recovered/pdf/00001611.pdf: OK
output/foremost_recovered/png/00001610.png: OK
```
The image hash after analysis matches the hash recorded in step 3, so the
evidence was not modified.

Screenshots: `01_tool_versions.png`, `15_final_hash_verification.png`

---

## 9. Follow-up: Recover the Complete PDF (not yet run on Kali)

Verified on a replica image built with the same steps. Run on Kali and
screenshot before submission.

```bash
# Prove the recovered file is the original's first revision
head -c 616 /tmp/Internal_Financial_Audit_2026.pdf | sha256sum
# expected: 6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275

# Locate both PDF end markers on the image
grep -abo '%%EOF' usb_evidence.raw
# expected:
# 825442:%%EOF
# 829395:%%EOF

# Carve from the PDF header (824832) to the end of the last %%EOF
mkdir -p output/manual_carve
dd if=usb_evidence.raw of=output/manual_carve/audit_report_full.pdf \
  bs=1 skip=824832 count=4569

sha256sum output/manual_carve/audit_report_full.pdf
# expected: cec3f6868336b0495c8216dc6bfd90e1c48c2052be6f2884c473c8c8e9ad454c

exiftool -Title -Author -Subject -Keywords -CreateDate \
  output/manual_carve/audit_report_full.pdf
# expected: the five metadata values from step 1c
```
Screenshot: `16_full_pdf_recovery.png`

---

## 10. Final Packaging

```bash
git status
git add .
git commit -m "Add USB data theft investigation (Strings, Foremost, ExifTool)"
git push
```
Screenshot: `17_github_push.png`
