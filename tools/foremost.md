# Foremost File-Carving Investigation

## Objective
Recover recognizable file artifacts from a synthetic FAT forensic image.

## Input
- Image: evidence/original/usb_image.dd
- Image size: 67,108,864 bytes (64 MiB)
- SHA-256: 5d4fbaf50a436c1129bab8a0e666dfab610ea38497feb179581b2a4087602deb

## Command
foremost -t png,zip -i evidence/original/usb_image.dd -o evidence/recovered/foremost-all

## Results
- PNG artifacts recovered: 1
- ZIP artifacts recovered: 1
- Recovered PNG size: 292 bytes
- Recovered PNG offset: 153,600 bytes
- Recovered ZIP size: 673 bytes
- Recovered ZIP offset: 155,648 bytes
- PNG validation: passed, 100 x 100 pixels
- ZIP validation: pending

## Interpretation
Foremost identified two artifacts using file signatures. The PNG is consistent
with the deliberately deleted test image in this synthetic lab. The ZIP was
also recovered, but its contents and integrity must be checked separately.

This is a controlled demonstration. These results do not independently prove
that real-world USB data theft occurred.

## Output
- evidence/recovered/foremost-all/audit.txt
- evidence/recovered/foremost-all/png/00000300.png
- evidence/recovered/foremost-all/zip/00000304.zip

## ZIP Validation
Command: unzip -t evidence/recovered/foremost-all/zip/00000304.zip
Result: No errors detected in compressed data.
Contents:
- case/sample-files/project-alpha.txt (160 bytes)
- case/sample-files/employee-records.txt (114 bytes)

## Targeted PNG Scan
Command:
foremost -t png -i evidence/original/usb_image.dd -o evidence/recovered/foremost-targeted

Result:
- One PNG recovered
- Size: 292 bytes
- Offset: 153,600 bytes
- Image dimensions: 100 x 100
- The targeted scan recovered the same PNG as the full scan.

## Recovered Artifact Hashes
- PNG: 74478068ef7c8eb3b688d3613835f61e64d1a345b0f60a38b1b65a10d0012d96
- ZIP: 2f4a85d97eeccf6cfed5b879805c808b91d1bbb6fa44696e43015bc0ce46896e

Both recovered artifact hashes passed SHA-256 verification.
