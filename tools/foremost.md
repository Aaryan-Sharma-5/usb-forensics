# Foremost — File Carving

## Purpose
Foremost recovers files from a disk image by searching for known file headers,
footers and internal structures. It does not use the FAT directory or the
allocation table, so it recovers deleted files as long as their clusters have
not been overwritten.

## Tool Details
- Package: `foremost` (Kali: Forensics › Forensic Carving Tools)
- Version: 1.5.7
- Configuration file: `/etc/foremost.conf`
- Input: `usb_evidence.raw` (52,428,800 bytes)
- Output: `output/foremost_recovered/`

## Options Used
| Option | Meaning |
| --- | --- |
| `-v` | Verbose: print the audit summary to the terminal |
| `-t pdf,png` | Carve only PDF and PNG signatures |
| `-i` | Input image |
| `-o` | Output directory (must be empty or not exist) |

Note: `docx` is not a valid Foremost type. Office 2007+ files are recovered
with `-t zip`.

## Command
```bash
foremost -v \
  -t pdf,png \
  -i usb_evidence.raw \
  -o output/foremost_recovered
```

## Results (`audit.txt`)
```text
Num      Name (bs=512)         Size      File Offset     Comment
0:      00001611.pdf          616 B          824832
1:      00001610.png           69 B          824320       (1 x 1)

2 FILES EXTRACTED
pdf:= 1
png:= 1
```

Foremost names each file after the 512-byte sector where it starts:
824832 / 512 = sector 1611, and 824320 / 512 = sector 1610.

| Recovered file | Size | Offset | SHA-256 |
| --- | --- | --- | --- |
| `pdf/00001611.pdf` | 616 B | 824832 | `6001609b248c93aab88abc73f02207c18217009157b61162db9bcc543ab29275` |
| `png/00001610.png` | 69 B | 824320 | `fdf040aaee9270aef6cf03752d51615cf6025c72081820fe0945f0c9c9cbdd9c` |

PNG signature check:
```bash
xxd -l 16 output/foremost_recovered/png/00001610.png
# 00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
```

## Interpretation
- The PDF fragment is not visible in the post-deletion directory listing, which is consistent with the deletion simulated during preparation. The PNG is the visible decoy `beach_trip.png`. Foremost can recover data by signatures without relying on directory entries.
- **The recovered PDF is incomplete.** The prepared original was 4,569 bytes
  (SHA-256 `cec3f686…454c`) but Foremost recovered 616 bytes (`6001609b…9275`).
  ExifTool saved the document metadata as a PDF *incremental update*, which
  appends a second section ending in its own `%%EOF`. Foremost stops at the
  first `%%EOF` (offset 825442), so it recovered only the original first
  revision, which has no metadata.
- The 616-byte size, page text, trailer structure and raw-image offsets are consistent with the recovered fragment being the first revision. Run the prefix hash check in `COMMANDS.md` section 9 on this case image before claiming an exact byte-for-byte prefix match.

## Follow-up: Recover the Complete PDF
Carving from the PDF header to the last `%%EOF` recovers the whole file. These
commands were verified on a replica image built with the same preparation
steps. Run them on the Kali image and capture screenshots before submission.

```bash
grep -abo '%%EOF' usb_evidence.raw
# 825442:%%EOF   <- end of first revision (where Foremost stopped)
# 829395:%%EOF   <- end of the ExifTool update

# length = 829395 + 6 ("%%EOF\n") - 824832 = 4569 bytes
mkdir -p output/manual_carve
dd if=usb_evidence.raw of=output/manual_carve/audit_report_full.pdf \
  bs=1 skip=824832 count=4569

sha256sum output/manual_carve/audit_report_full.pdf
# expected: cec3f6868336b0495c8216dc6bfd90e1c48c2052be6f2884c473c8c8e9ad454c
```

A match with the original document's hash is the strongest possible link
between the USB and the confidential file.

## Limitations
- Header/footer carving can truncate files that contain multiple footers
  (as here) or that are fragmented.
- Carving gives no filenames or timestamps. These come from the filesystem or
  from embedded metadata.
