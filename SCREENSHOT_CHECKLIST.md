# Screenshot Checklist

Save screenshots in `screenshots/` using these names. Step numbers refer to
`COMMANDS.md`.

| # | File name | What it must show | Step | Report section |
| --- | --- | --- | --- | --- |
| 01 | `01_tool_versions.png` | `evidence/tool_versions.txt`: Foremost 1.5.7, ExifTool 13.55, GNU strings 2.47 | 8 | Appendix A |
| 02 | `02_files_before_deletion.png` | `ls -lh /mnt/usb_sim` with the PDF (4.5K), PNG and notes | 1d | 6 |
| 03 | `03_deletion_and_unmount.png` | `rm`, listing with only the 2 decoys, "Synthetic filesystem unmounted" | 2 | 6 |
| 04 | `04_image_hash.png` | SHA-256 `74f073eb…` and image size 52428800 bytes | 3 | 7 |
| 05 | `05_strings_extraction.png` | `strings -a -t d usb_evidence.raw > output/all_strings.txt` | 4 | 8 |
| 06 | `06_strings_keyword_hits.png` | The 9 keyword hits with offsets 825111–826492 | 4 | 8 |
| 07 | `07_foremost_execution.png` | `foremost -v -t pdf,png …` with "2 FILES EXTRACTED" | 5 | 9 |
| 08 | `08_foremost_audit.png` | `cat output/foremost_recovered/audit.txt` | 5 | 9 |
| 09 | `09_foremost_recovered_files.png` | `find` / `ls -lh` of the `pdf/` and `png/` folders | 5 | 9 |
| 10 | `10_png_signature.png` | `xxd` / `od` showing `89 50 4E 47 0D 0A 1A 0A` | 5 | 9 |
| 11 | `11_exiftool_recovered_pdf.png` | Full ExifTool output for `00001611.pdf` | 6 | 10 |
| 12 | `12_exiftool_targeted_query_empty.png` | `exiftool -Title -Author …` returning nothing | 6 | 10 |
| 13 | `13_exiftool_recovered_png.png` | ExifTool output for `00001610.png` (1×1) | 6 | 10 |
| 14 | `14_original_vs_recovered_hash.png` | `strings` of recovered PDF and the two different SHA-256 values | 7 | 11 |
| 15 | `15_final_hash_verification.png` | `final_hashes.sha256` and `sha256sum -c` showing OK | 8 | 7 |
| 16 | `16_full_pdf_recovery.png` | `grep -abo '%%EOF'`, `dd` carve, hash `cec3f686…`, ExifTool showing full metadata | 9 (pending) | 11 |
| 17 | `17_github_push.png` | Repository files and `git push` confirmation | 10 (pending) | Appendix B |

## Tips

- Keep the prompt `aaryan㉿Aaryan)-[~/forensic_case_study]` visible so the
  working directory is clear.
- Crop to the command and its output; make sure offsets and hashes are readable.
- Under each screenshot in the report, add one line saying what it proves.
