# Rizin Static Binary Analysis

## Objective
Demonstrate static analysis of a harmless compiled C program.

## Sample
- Source: case/sample-files/sample_binary.c
- Binary: evidence/original/sample_binary
- Architecture: x86-64
- Format: ELF64
- Compiler: GCC 16.2.0
- Analysis tool: Rizin 0.9.1

## Commands
- aaa: Automated analysis completed.
- iI: Displayed binary information.
- afl: Listed recognized functions, including main.
- izz: Located embedded strings.
- ii: Listed imported symbols.

## Observations
The binary contained these strings:
- SIMULATED FILE TRANSFER TEST
- USB forensic lab demonstration

The analysis identified main and standard runtime/library functions.
The command `li` was unsupported in this session and was not required.

## Interpretation and Limitations
This is a controlled test executable created for the lab. It was analyzed
statically and is not a recovered artifact from the synthetic USB image.
The results demonstrate binary inspection, not proof of data theft.
