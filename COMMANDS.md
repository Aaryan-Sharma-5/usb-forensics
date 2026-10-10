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
tshark -v
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

## Local HTTP Sample File Server

```bash
cd ~/usb-forensics

python3 -m http.server 8000 \
  --bind 127.0.0.1 \
  --directory case/sample-files
```

Expected access log examples:

```text
"GET / HTTP/1.1" 200
"GET /project-alpha.txt HTTP/1.1" 200
"GET /employee-records.txt HTTP/1.1" 200
```

Screenshots:

- `screenshots/09_http_server_started.png`
- `screenshots/10_sample_file_requests.png`

## tshark Packet Capture

First verify whether the PCAP exists:

```bash
ls -lh evidence/pcap/lab-traffic.pcapng
sudo tshark -r evidence/pcap/lab-traffic.pcapng
```

If the file does not exist yet, capture loopback HTTP traffic:

```bash
cd ~/usb-forensics

sudo tshark \
  -i lo \
  -f "tcp port 8000" \
  -w evidence/pcap/lab-traffic.pcapng
```

If the evidence path gives `Permission denied`, capture to `/tmp` and then copy it into the evidence folder:

```bash
sudo tshark \
  -i lo \
  -f "tcp port 8000" \
  -w /tmp/lab-traffic.pcapng

mkdir -p evidence/pcap
sudo cp /tmp/lab-traffic.pcapng evidence/pcap/lab-traffic.pcapng
sudo chown "$USER:$USER" evidence/pcap/lab-traffic.pcapng
sha256sum evidence/pcap/lab-traffic.pcapng
md5sum evidence/pcap/lab-traffic.pcapng
tshark -r evidence/pcap/lab-traffic.pcapng
```

Screenshots:

- `screenshots/11_pcap_missing_before_capture.png`
- `screenshots/12_tshark_permission_denied.png`
- `screenshots/13_tshark_tmp_capture_success.png`
- `screenshots/14_pcap_hash_verification.png`
- `screenshots/15_tshark_read_pcap.png`

## Final Packaging

```bash
tree
git status
git add .
git commit -m "Add forensic investigation report and evidence templates"
git push
```

Screenshot: `screenshots/16_github_push.png`
