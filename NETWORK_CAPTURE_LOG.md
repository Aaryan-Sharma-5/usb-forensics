# Network Capture Log

This log documents the additional network traffic capture performed during the USB data theft case study. The commands show a local HTTP server hosting sample files and a `tshark` capture on loopback traffic.

## 1. Local Sample File Server

The sample files were hosted locally from the project directory.

```bash
cd ~/usb-forensics

python3 -m http.server 8000 \
  --bind 127.0.0.1 \
  --directory case/sample-files
```

Observed server output:

```text
Serving HTTP on 127.0.0.1 port 8000 (http://127.0.0.1:8000/) ...
127.0.0.1 - - [10/Oct/2026 16:11:30] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [10/Oct/2026 16:15:17] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [10/Oct/2026 16:15:25] "GET /project-alpha.txt HTTP/1.1" 200 -
127.0.0.1 - - [10/Oct/2026 16:15:31] "GET /employee-records.txt HTTP/1.1" 200 -
```

The requests show that `project-alpha.txt` and `employee-records.txt` were accessed through the local web server. These files can be treated as sample sensitive files for demonstration.

## 2. Initial PCAP Verification Attempt

The following command checked whether the capture file already existed:

```bash
ls -lh evidence/pcap/lab-traffic.pcapng
sudo tshark -r evidence/pcap/lab-traffic.pcapng
```

Observed result:

```text
ls: cannot access 'evidence/pcap/lab-traffic.pcapng': No such file or directory
tshark: The file "evidence/pcap/lab-traffic.pcapng" doesn't exist.
```

Analysis: the PCAP had not been created yet, so `tshark -r` could not read it.

## 3. Packet Capture Attempt to Evidence Folder

The first packet capture attempt used the relative evidence path:

```bash
cd ~/usb-forensics

sudo tshark \
  -i lo \
  -f "tcp port 8000" \
  -w evidence/pcap/lab-traffic.pcapng
```

Observed result:

```text
Capturing on 'Loopback: lo'
tshark: The file to which the capture would be saved ("evidence/pcap/lab-traffic.pcapng") could not be opened: Permission denied.
```

A second attempt used the absolute path:

```bash
sudo tshark \
  -i lo \
  -f "tcp port 8000" \
  -w "$HOME/usb-forensics/evidence/pcap/lab-traffic.pcapng"
```

Observed result:

```text
Capturing on 'Loopback: lo'
tshark: The file to which the capture would be saved ("/home/aaryan/usb-forensics/evidence/pcap/lab-traffic.pcapng") could not be opened: Permission denied.
```

Analysis: the evidence output folder likely had missing or restrictive permissions. This is useful to document because forensic work should record failed commands as well as successful ones.

## 4. Successful Temporary Capture

The working command saved the capture to `/tmp`:

```bash
sudo tshark \
  -i lo \
  -f "tcp port 8000" \
  -w /tmp/lab-traffic.pcapng
```

Observed result:

```text
Capturing on 'Loopback: lo'
36
```

Analysis: `tshark` successfully captured loopback traffic for TCP port 8000 and wrote the PCAP to `/tmp/lab-traffic.pcapng`.

## 5. Recommended Cleanup and Evidence Copy

After capture, copy the file into the evidence directory and calculate hashes:

```bash
cd ~/usb-forensics
mkdir -p evidence/pcap
sudo cp /tmp/lab-traffic.pcapng evidence/pcap/lab-traffic.pcapng
sudo chown "$USER:$USER" evidence/pcap/lab-traffic.pcapng
sha256sum evidence/pcap/lab-traffic.pcapng
md5sum evidence/pcap/lab-traffic.pcapng
```

Then verify with `tshark`:

```bash
tshark -r evidence/pcap/lab-traffic.pcapng
```

## 6. Report Analysis

The PCAP supports the case study by showing sample data access over HTTP. It can demonstrate that files named `project-alpha.txt` and `employee-records.txt` were requested from the local sample-file server. In the final report, include screenshots of:

- The Python HTTP server logs.
- The failed `ls` and `tshark -r` read before PCAP creation.
- The permission-denied capture attempts.
- The successful `/tmp/lab-traffic.pcapng` capture.
- The final hash of the copied PCAP file.
- `tshark -r evidence/pcap/lab-traffic.pcapng` output.
