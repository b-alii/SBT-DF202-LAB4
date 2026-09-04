# Lab 4 — Steganography and Hidden Data Detection

<table>
  <tr>
    <th>Student Name</th>
    <th>Basiru Aliyu</th>
  </tr>
  <tr>
    <td>Registration Number</td>
    <td>FWSD25/11424</td>
  </tr>
  <tr>
    <td>Course / Module</td>
    <td>SBT-DF202 — Computer and Digital Forensics</td>
  </tr>
  <tr>
    <td>Date Performed</td>
    <td>04/09/2026</td>
  </tr>
</table>

## 1. Introduction and Objectives

This report documents a steganography exercise using Steghide to embed and extract a hidden text file within a BMP carrier image, followed by comparative analysis of the original and stego images, and a controlled dictionary-based password recovery test using StegSeek. The exercise demonstrates both the offensive perspective (how data can be concealed inside an innocuous-looking image) and the forensic perspective (how such concealment can be detected, compared, and  where a weak password is used recovered).

Steganography differs from encryption in that it hides the existence of data rather than only its meaning: a stego image is designed to be visually and often statistically indistinguishable from an unmodified original, which is precisely what makes it a forensically interesting technique. This lab demonstrates that indistinguishability at the pixel/visual level does not extend to the file's raw byte content or cryptographic hash — two images can look identical to the eye while being provably different files.

## 2. Part A: Preparing and Preserving the Evidence

The carrier image must be preserved unmodified throughout the exercise so that a clean 'before' state is always available for comparison against the embedded ('stego') version produced in Part C.

### 2.1 Working Folder and Download

A dedicated folder isolates this exercise's files; downloading the exact lab-specified carrier image (rather than a substitute) ensures the embedding/extraction steps match the source lab's expected structure.

```bash
mkdir -p ~/stego-lab
```

![Evidence screenshot](images/image1.png)

### 2.2 Confirming File Type

```bash
file inspects the actual byte signature of the downloaded image, confirming it is genuinely a BMP (not, for example, an HTML error page saved with a .bmp extension due to a failed download).
```

```bash
file _tower_original_image_for_lab.bmp
```

![Evidence screenshot](images/image2.png)

Explanation of the command / output:

- Reports the detected format, dimensions, bit depth, and colour-plane details read directly from the BMP header.

### 2.3 Hashing the Original Carrier Image

Establishes the baseline SHA-256 value of the untouched original — this is the reference value Part E will compare the stego image against, and the value that proves the original was never overwritten.

```bash
sha256sum _tower_original_image_for_lab.bmp
```

![Evidence screenshot](images/image3.png)

Explanation of the command / output:

```bash
6508bebb94bd8ec0833b9f85580cda4261d89632ddc89e7f9f7093b5e653c4db
```

- A single SHA-256 hash is recorded and kept for comparison at every later stage of the exercise.

## 3. Part B: Creating the Secret Evidence File

### 3.1 Creating secret.txt

This known-content file is the payload that will be hidden inside the carrier image; because its exact original content and hash are recorded now, any later extraction can be verified as bit-for-bit successful rather than merely 'file exists'.

```bash
cat > secret.txt << 'EOF'
Basiru Aliyu
FWSD25/11424
04/09/2026
This is a controlled evidence message prepared for the ICDFA steganography lab.
This file was created for the ICDFA lab and is authorised training evidence only.
EOF
cat secret.txt
```

![Evidence screenshot](images/image4.png)

Explanation of the command / output:

- The heredoc (<< 'EOF') writes multiple lines to the file without needing an interactive text editor.

### 3.2 Hashing secret.txt

Recording the hash of the original secret file before embedding is what makes later verification (Part D) meaningful — a byte-for-byte match after extraction proves Steghide's embedding and extraction process is lossless.

```bash
sha256sum secret.txt
```

![Evidence screenshot](images/image5.png)

Explanation of the command / output:

```bash
5a4e6d0dabd5b2e19693710ea98d9749379aacf1ade951ec097e0a62e39a1e62
```

- Produces the reference hash that the extracted copy in Part D must match exactly.

## 4. Part C: Embedding the Hidden File with Steghide

### 4.1 Embedding secret.txt into the Carrier

Steghide uses a password-derived keystream to select which pixel data bits to modify (least-significant-bit style embedding for BMP/other supported formats), writing the result to a new file so the original carrier remains untouched.

```bash
steghide embed -ef secret.txt -cf _tower_original_image_for_lab.bmp -sf tower_stego_lab4.bmp -p 1234
```

![Evidence screenshot](images/image6.png)

Explanation of the command / output:

-ef (embed file): the payload to hide — secret.txt.

-cf (cover file): the original, unmodified carrier image — read only, never overwritten.

-sf (stego file): the new output file that will contain the carrier's visual content plus the hidden payload.

-p: the password used to derive the embedding keystream; the same password is required to extract the payload later.

- The lab specifies password 1234 deliberately: it is weak enough to be recoverable by StegSeek's dictionary attack in Part F, demonstrating why weak steganography passwords are a forensic weakness.

### 4.2 Hashing the Resulting Stego Image

Records the stego image's own baseline hash, which Part E will compare directly against the original carrier's hash from Section 2.3.

```bash
sha256sum tower_stego_lab4.bmp
```

![Evidence screenshot](images/image7.png)

Explanation of the command / output:

```bash
3d7ab5dd11150a1ea37c12053e42d03441d2c471df088a6d99a998125e22828a
```

- Even though the embedded payload is small relative to the image, changing any bit anywhere in the file produces a completely different SHA-256 hash (the avalanche effect) there is no scenario where embedding succeeds but the hash stays the same.

## 5. Part D:  Extracting and Verifying the Hidden Evidence

### 5.1 Extracting with the Correct Password

Steghide reverses the embedding process using the same password to regenerate the correct keystream, reading the hidden bits back out of the stego image's pixel data.

```bash
mkdir -p extracted
steghide extract -sf tower_stego_lab4.bmp -xf extracted/extracted_secret.txt -p 1234
```

![Evidence screenshot](images/image8.png)

Explanation of the command / output:

-sf: the stego image to extract from.

-xf: the output path for the recovered payload file.

-p: must match the password used during embedding exactly, or extraction fails.

### 5.2 Verifying Extraction Integrity

Comparing the extracted file against the original secret.txt — both byte-for-byte (diff) and by cryptographic hash — proves the round trip (embed then extract) preserved the payload exactly, with no corruption or truncation.

```bash
diff secret.txt extracted/extracted_secret.txt
```

```bash
sha256sum secret.txt extracted/extracted_secret.txt
```

![Evidence screenshot](images/image9.png)

![Evidence screenshot](images/image10.png)

Explanation of the command / output:

- diff with no output means the two files are byte-for-byte identical.

- Matching SHA-256 hashes independently confirm the same result cryptographically.

### 5.3 Extraction Verification Summary

<table>
  <tr>
    <th>File</th>
    <th>SHA-256</th>
    <th>Matches Original?</th>
  </tr>
  <tr>
    <td>secret.txt (original)</td>
    <td>5a4e6d0dabd5b2e19693710ea98d9749379aacf1ade951ec097e0a62e39a1e62</td>
    <td>—</td>
  </tr>
  <tr>
    <td>extracted_secret.txt</td>
    <td>5a4e6d0dabd5b2e19693710ea98d9749379aacf1ade951ec097e0a62e39a1e62</td>
    <td>Yes</td>
  </tr>
</table>

## 6. Part E:  Comparing Original and Stego Images

This section directly demonstrates the core forensic principle of the exercise: visual similarity and file-level identity are entirely different properties. The original and stego images should be visually indistinguishable when opened, yet provably different at every other level examined below.

### 6.1 File Sizes

ls -l _tower_original_image_for_lab.bmp tower_stego_lab4.bmp

![Evidence screenshot](images/image11.png)

<table>
  <tr>
    <th>Original image size</th>
    <th>9277494</th>
  </tr>
  <tr>
    <td>Stego image size</td>
    <td>9277494</td>
  </tr>
  <tr>
    <td>Difference</td>
    <td>[bytes, or &#x27;identical&#x27; — Steghide typically preserves BMP dimensions/size exactly, embedding within existing pixel data rather than appending]</td>
  </tr>
</table>

### 6.2 SHA-256 Hashes

```bash
sha256sum _tower_original_image_for_lab.bmp tower_stego_lab4.bmp
```

![Evidence screenshot](images/image12.png)

<table>
  <tr>
    <th>File</th>
    <th>SHA-256</th>
  </tr>
  <tr>
    <td>Original carrier</td>
    <td>6508bebb94bd8ec0833b9f85580cda4261d89632ddc89e7f9f7093b5e653c4db</td>
  </tr>
  <tr>
    <td>Stego image</td>
    <td>3d7ab5dd11150a1ea37c12053e42d03441d2c471df088a6d99a998125e22828a</td>
  </tr>
</table>

### 6.3 Initial Bytes via xxd

```bash
xxd _tower_original_image_for_lab.bmp | head -n 10
```

```bash
xxd tower_stego_lab4.bmp | head -n 10
```

![Evidence screenshot](images/image13.png)

![Evidence screenshot](images/image14.png)

The BMP file header (first ~14-54 bytes: signature 'BM', file size, pixel data offset, DIB header) is typically unchanged between the two files, since Steghide modifies pixel data, not header structure. Differences become apparent further into the file, within the pixel data region, once embedding has altered least-significant bits.

### 6.4 Readable Strings

```bash
strings _tower_original_image_for_lab.bmp | wc -l
```

```bash
strings tower_stego_lab4.bmp | wc -l
```

![Evidence screenshot](images/image15.png)

![Evidence screenshot](images/image16.png)

Steghide does not store the embedded filename or content in plaintext within the stego image (unlike some weaker steganography tools) — the payload is bit-scattered through pixel data according to the password-derived keystream, not concatenated in a readable block. The grep above is expected to return no matches even though the file is genuinely present, which is itself a meaningful finding to record.

### 6.5 Explanation: Visual Similarity vs. Digital Identity

[Write 3–5 sentences explaining: the two images render identically or near-identically to a human viewer because Steghide only alters least-significant bits of pixel values — changes too small to be visible. However, every one of those altered bits changes the file's raw byte sequence, and a cryptographic hash function is designed so that even a single-bit difference produces a completely unrelated hash output (the avalanche effect). This is why hashing — not visual inspection — is the appropriate forensic method for detecting whether a file has been modified, regardless of how convincing it looks.]

## 7. Part F — Controlled Password Dictionary Test with StegSeek

StegSeek performs an automated dictionary attack against a Steghide-produced stego file: for each candidate password in a wordlist, it attempts extraction and reports success if a valid embedded payload is found. This demonstrates why password strength (not just the existence of a password) matters for steganographic concealment.

### 7.1 Creating the Lab Wordlist

A minimal wordlist containing the known correct password proves the recovery mechanism works correctly before considering it a meaningful security demonstration.

```bash
cat > lab4_wordlist.txt << 'EOF'
password
letmein
1234
admin123
EOF
```

Explanation of the command / output:

- Including a few decoy entries alongside the real password (1234) makes the test slightly more realistic than a single-entry wordlist, without meaningfully changing the outcome for such a short list.

### 7.2 Running StegSeek

StegSeek is dramatically faster than brute-forcing Steghide manually because it uses an optimised cracking library rather than invoking steghide extract once per candidate password.

```bash
stegseek _tower_stego_lab4.bmp lab4_wordlist.txt
```

![Evidence screenshot](images/image17.png)

Explanation of the command / output:

- On success, StegSeek reports the recovered password and typically writes the extracted payload to a file automatically (e.g., tower_stego_lab4.bmp.out).

![Evidence screenshot](images/image18.png)

### 7.3 Result

<table>
  <tr>
    <th>Password recovered?</th>
    <th>Yes</th>
  </tr>
  <tr>
    <td>Recovered password value</td>
    <td>1234</td>
  </tr>
  <tr>
    <td>Time taken</td>
    <td>40seconds</td>
  </tr>
  <tr>
    <td>Auto-extracted payload matches secret.txt?</td>
    <td>Yes</td>
  </tr>
</table>

## 8. Part G: Independent Student Task

This section repeats the full embed → extract → verify → dictionary-test workflow independently, using a student-chosen filename, message, and password, to demonstrate the process can be reproduced without step-by-step guidance.

### 8.1 Creating the Personal Hidden File

A short original explanation of steganography, written in the student's own words, serves as this task's payload.

```bash
cat > Basiru_Aliyu_hidden_note.txt << 'EOF'
steganography is the practice of concealing the existence of data within another file and why it matters in digital forensics: examiners must know to look beyond a file's apparent content/type, since innocuous-looking images, audio, or documents can carry concealed data that ordinary file review would miss.
EOF
```

![Evidence screenshot](images/image19.png)

Explanation of the command / output:

- Same heredoc technique as Section 3.1, applied to original content.

### 8.2 Chosen Password

<table>
  <tr>
    <td>Password chosen for this task</td>
    <td>112233</td>
  </tr>
</table>

### 8.3 Embedding

Same process as Part C, applied to the personal file and chosen password, into a freshly named stego output so it does not overwrite tower_stego_lab4.bmp from the guided portion.

```bash
steghide embed -ef Basiru_Aliyu_hidden_note.txt -cf image.jpg -sf Basiru_Aliyu_stego.jpg -p 112233
```

![Evidence screenshot](images/image20.png)

Explanation of the command / output:

- Identical mechanism to Section 4.1, using the student's own filenames and password.

### 8.4 Extraction and Integrity Verification

Same process as Part D, confirming the personal payload round-trips correctly.

```bash
steghide extract -sf Basiru_Aliyu_stego.jpg -xf extracted/Basiru_Aliyu_hidden_note_extracted.txt -p 112233
```

```bash
diff Basiru_Aliyu_hidden_note.txt extracted/Basiru_Aliyu_hidden_note_extracted.txt
```

![Evidence screenshot](images/image21.png)

![Evidence screenshot](images/image22.png)

```bash
sha256sum Basiru_Aliyu_hidden_note.txt extracted/Basiru_Aliyu_hidden_note_extracted.txt
```

![Evidence screenshot](images/image23.png)

Explanation of the command / output:

- Identical verification logic to Section 5.2.

### 8.5 Personal Wordlist and Recovery Test

Repeats the StegSeek dictionary attack against the student's own stego image and password choice, demonstrating the same recovery mechanism against different (self-selected) input.

```bash
cat > my_wordlist.txt << 'EOF'
1123
```

11245

112233

974657

645485

96854904

```bash
EOF
stegseek Basiru_Aliyu_stego.jpg my_wordlist.txt
```

![Evidence screenshot](images/image24.png)

Explanation of the command / output:

- Same mechanism as Section 7.2.

## 9. Consolidated Evidence Checklist

<table>
  <tr>
    <th>Required Evidence Item</th>
    <th>Included?</th>
    <th>Section</th>
  </tr>
  <tr>
    <td>Carrier image downloaded, type confirmed, hashed</td>
    <td>[Y]</td>
    <td>2</td>
  </tr>
  <tr>
    <td>secret.txt created and hashed</td>
    <td>[Y]</td>
    <td>3</td>
  </tr>
  <tr>
    <td>Embedding performed (password 1234) and stego image hashed</td>
    <td>[Y]</td>
    <td>4</td>
  </tr>
  <tr>
    <td>Extraction performed and verified against original (diff/hash)</td>
    <td>[Y]</td>
    <td>5</td>
  </tr>
  <tr>
    <td>Original vs stego comparison (size, hash, xxd, strings)</td>
    <td>[Y]</td>
    <td>6</td>
  </tr>
  <tr>
    <td>StegSeek dictionary test documented (or fallback documented)</td>
    <td>[Y]</td>
    <td>7</td>
  </tr>
  <tr>
    <td>Independent task (Part G) completed end-to-end with screenshots</td>
    <td>[Y]</td>
    <td>8</td>
  </tr>
</table>

## 10. Conclusion

This lab demonstrated how steganography can be used to conceal data within an image and how such hidden information can be identified and recovered through forensic analysis. Using Steghide, a secret file was successfully embedded into a carrier image and later extracted with its integrity confirmed through diff and matching SHA-256 hashes. The comparison between the original and stego images also showed that files can appear visually identical while having different digital identities, as demonstrated by their different SHA-256 hashes.
The lab also demonstrated the security weakness of using simple passwords for steganographic protection. StegSeek successfully recovered the password through a controlled dictionary attack, highlighting the importance of using strong passwords. Overall, the exercise provided practical experience in both concealing information and detecting, verifying, and recovering hidden data during digital forensic investigations.
