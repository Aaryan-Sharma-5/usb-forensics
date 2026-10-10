# USB Data Theft Investigation

Digital forensics mini-project for demonstrating Kali Linux tools through a case study.

## Submission Mapping

| Student | Case Study | Kali Category | Tool 1 | Tool 2 | Tool 3 |
| --- | --- | --- | --- | --- | --- |
| Aaryan Sharma | USB Data Theft Investigation | Forensics / Forensic Carving | Foremost | ExifTool | srch_strings / Strings |
| Aditey Kshirsagar | Suspicious Host Reconnaissance | Reconnaissance | Nmap | WhatWeb | theHarvester |
| Aditya Baheti | Password-Protected Evidence Archive | Credential Access | hashid | fcrackzip | John the Ripper |

## Repository Contents

- `REPORT.md` - main investigation report template and filled USB data theft case study.
- `COMMANDS.md` - command log for screenshots and demonstrations.
- `NETWORK_CAPTURE_LOG.md` - documented HTTP server and `tshark` packet-capture steps.
- `VIDEO_SCRIPT.md` - 10-minute presentation flow with member participation.
- `CHAIN_OF_CUSTODY.md` - sample evidence handling template.
- `SCREENSHOT_CHECKLIST.md` - required screenshot list for the final report.

## Case Study Summary

An employee is suspected of copying confidential company files to a USB drive. A forensic image of the USB device is acquired and analyzed without modifying the original media. The investigation demonstrates file carving, metadata extraction, printable string analysis, and sample network capture review to identify recovered files, timestamps, author/device traces, file access events, and possible sensitive data.

## Tool References

- Foremost: forensic recovery using file headers, footers, and internal structures.
- ExifTool: metadata extraction from documents, images, PDFs, and many other file types.
- srch_strings / Strings: printable string extraction for keyword-based forensic review.
- tshark: packet capture and packet analysis for network evidence.

## Note

Do not upload real confidential evidence, passwords, or private personal data. Use sample evidence files or screenshots prepared for demonstration.
