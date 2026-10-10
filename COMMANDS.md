# Command Log

Use this file while recording screenshots. After each command, paste the important output or add a screenshot filename.

## Environment

```bash
date
uname -a
lsb_release -a
foremost -V
exiftool -ver
srch_strings -v
```

## Evidence Preservation

```bash
mkdir -p evidence output screenshots
sha256sum evidence/usb_image.dd
md5sum evidence/usb_image.dd
```

Screenshot: `screenshots/01_hash_verification.png`

## Foremost Demonstration

```bash
mkdir -p output/foremost
foremost -v -t jpg,png,pdf,doc,xls,zip -i evidence/usb_image.dd -o output/foremost
ls -R output/foremost
cat output/foremost/audit.txt
```

Screenshots:

- `screenshots/02_foremost_execution.png`
- `screenshots/03_foremost_output_directory.png`
- `screenshots/04_foremost_audit.png`

## ExifTool Demonstration

```bash
exiftool output/foremost/pdf/*.pdf
exiftool output/foremost/jpg/*.jpg
exiftool -csv output/foremost/*/* > output/exiftool_metadata.csv
head output/exiftool_metadata.csv
```

Screenshots:

- `screenshots/05_exiftool_metadata.png`
- `screenshots/06_exiftool_csv_export.png`

## srch_strings / Strings Demonstration

```bash
mkdir -p output/strings
srch_strings -a -t d evidence/usb_image.dd > output/strings/usb_strings.txt
grep -iE "confidential|password|project|invoice|client|secret" output/strings/usb_strings.txt
strings -a evidence/usb_image.dd | grep -iE "confidential|password|project|invoice|client|secret"
```

Screenshots:

- `screenshots/07_strings_extraction.png`
- `screenshots/08_keyword_hits.png`

## Final Packaging

```bash
tree
git status
git add .
git commit -m "Add forensic investigation report and evidence templates"
git push
```

Screenshot: `screenshots/09_github_push.png`
