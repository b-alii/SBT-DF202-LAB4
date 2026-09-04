# Digital Forensics Lab 4 — Steganography and Hidden Data Detection

## Description

This laboratory exercise focuses on **steganography and hidden data detection** from a digital forensics perspective. The practical simulates a situation where an image file is suspected of containing hidden information.

Students will act as junior digital forensic trainees and work with an authorised lab image and files created within the training environment. The exercise covers the complete process of preparing and preserving a carrier image, creating a controlled evidence file, hiding the evidence inside an image, extracting the concealed data, verifying its integrity, and testing password recovery.

The lab uses **Steghide** for embedding and extraction and **StegSeek** for a controlled dictionary-based password recovery test. Cryptographic hashing, `xxd`, file comparison, and string analysis are also used to compare the original and stego images.

The exercise demonstrates both sides of steganography: how information can be concealed inside an apparently ordinary image and how forensic investigators can detect, analyse, verify, and recover concealed information.

> **Important:** All activities must be performed only on the authorised laboratory image and files created within the designated training environment.

## Objectives

By completing this laboratory, students will demonstrate the ability to:
1. Explain the difference between **steganography and encryption**.
2. Describe **insertion and substitution-based hiding techniques**.
3. Explain the concept of **Least Significant Bit (LSB)** data hiding.
4. Understand how bitmap images can be used as carrier files for concealed information.
5. Prepare and preserve an original carrier image for forensic analysis.
6. Create a controlled secret evidence file and calculate its **SHA-256 hash**.
7. Use **Steghide** to embed a secret file into a carrier image.
8. Extract concealed information from a stego image using the correct password.
9. Compare the original and extracted evidence using `diff` and cryptographic hashes.
10. Compare the original and stego images using:
    - File size
    - SHA-256 hashes
    - `xxd`
    - Readable strings
11. Explain why two images can appear visually identical while having different underlying digital content.
12. Perform a controlled **password dictionary test** using StegSeek.
13. Document password recovery results and any errors encountered during the exercise.
14. Complete an independent steganography task using a student-selected filename, message, and password.
15. Produce a **repeatable forensic report** containing commands, results, screenshots, and observations.
