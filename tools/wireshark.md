# Wireshark/TShark Network Forensics

## Objective
Capture and analyze controlled HTTP traffic in a local test environment.

## Environment
- Platform: Kali Linux Rolling on WSL 2
- TShark: 4.6.6
- Interface: lo (loopback)
- Server: Python HTTP server bound to 127.0.0.1:8000
- Capture: evidence/pcap/lab-traffic.pcapng

## Traffic Generation
HTTP requests were generated using curl against the local test server.

Requests:
- GET /
- GET /project-alpha.txt
- GET /employee-records.txt

## Findings
The saved packet capture contained 36 packets, including TCP connection
establishment, HTTP requests, HTTP responses, acknowledgements, and connection
termination.

The HTTP display filter identified all three GET requests and their responses.
The test files contain fictional educational data.

## Evidence Integrity
SHA-256:
c8f7ca75933d0e5aa241d0ea4647010544a2f55bf877ee97eb1b5dc672b982c6

The recorded hash was verified against the saved project PCAP.

## Limitations
This capture demonstrates local HTTP traffic analysis. It does not establish
that a physical USB device was connected or that actual data theft occurred.
