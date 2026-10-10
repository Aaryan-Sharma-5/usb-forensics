# Chain of Custody Template

| Case ID | Evidence ID | Description | Collected By | Date/Time | Location | Hash | Transfer Details | Signature |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| USB-001 | E01 | USB drive / forensic image | `<name>` | `<date/time>` | `<location>` | `<SHA256>` | Collected from suspect workstation | `<signature>` |
| USB-001 | E01-WC | Working copy of USB image | `<name>` | `<date/time>` | Lab system | `<SHA256>` | Created for analysis | `<signature>` |

## Evidence Handling Notes

- Original USB should be write-protected.
- Hash must be calculated before and after imaging.
- Working copy must be used for all tools.
- Every transfer must record person, time, purpose, and signature.
