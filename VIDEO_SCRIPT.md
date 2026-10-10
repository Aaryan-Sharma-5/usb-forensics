# Video Presentation Script

Maximum duration: 10 minutes.

## 0:00-0:45 - Introduction

Speaker: Aaryan Sharma

Introduce the case study: suspected USB data theft. Explain that the investigation uses Kali Linux forensic tools to recover deleted files, inspect metadata, and search for sensitive text indicators.

## 0:45-1:30 - Forensic Procedure

Speaker: Aditey Kshirsagar

Explain the investigation steps:

1. Identify and seize USB evidence.
2. Preserve original device with write protection.
3. Create forensic image.
4. Calculate hashes.
5. Analyze only the working copy.
6. Document findings with screenshots.

## 1:30-3:45 - Foremost Demonstration

Speaker: Aaryan Sharma

Show:

- Input forensic image.
- Foremost command.
- Recovered output folders.
- `audit.txt`.

Key point: Foremost recovers files from headers, footers, and internal file structures, which is useful for deleted USB data.

## 3:45-5:45 - ExifTool Demonstration

Speaker: Aditya Baheti

Show:

- Metadata extraction from recovered files.
- CSV metadata export.
- Important metadata fields such as author, creation time, modification time, software, or device.

Key point: Metadata helps build a timeline and can support attribution.

## 5:45-7:30 - srch_strings / Strings Demonstration

Speaker: Aditey Kshirsagar

Show:

- Extracting printable strings from the USB image.
- Searching for case keywords.
- Explaining why strings are useful for deleted or fragmented evidence.

Key point: String hits are leads and must be verified against recovered files.

## 7:30-8:45 - Findings and Analysis

Speaker: Aaryan Sharma

Summarize:

- Number and type of files recovered.
- Important metadata found.
- Relevant keyword matches.
- How findings support or do not support the data theft allegation.

## 8:45-9:30 - GitHub Repository

Speaker: Aditya Baheti

Show:

- Repository structure.
- Report files.
- Screenshot folder.
- Git commit and push.

## 9:30-10:00 - Conclusion

All members

Close with the final conclusion and limitations. Mention that metadata can be changed, recovered files can be incomplete, and all findings require correlation.
