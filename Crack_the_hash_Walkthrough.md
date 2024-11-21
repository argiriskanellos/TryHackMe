
# Crack the Hash

This document outlines the methodology for cracking various types of hashes using Hashcat. For each hash, we save the hash in a file named `hash.txt`, and then we use Hashcat to find the plaintext.

## Level 1

### 1. MD5 Hash
- **Hash:** `48bb6e862e54f2a795ffc4e541caed4d`
- **Plaintext:** `easy`

### 2. SHA1 Hash
- **Hash:** `CBFDAC6008F9CAB4083784CBD1874F76618D2A97`
- **Plaintext:** `password123`

### 3. SHA256 Hash
- **Hash:** `1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032`
- **Plaintext:** `letmein`

### 4. Bcrypt (Blowfish) Hash
- **Hash:** `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`
- **Plaintext:** `bleh`

To crack the Bcrypt hash, we will need the Hashcat tool along with a wordlist. The `hash.txt` file contains the hash we want to find, and `3200` is the mode for bcrypt and Blowfish.

#### Cracking Steps:
1. Create a filtered wordlist using the command:
   ```bash
   grep -E '^[a-zA-Z]{4}$' /File_Path/1–4_all_letters_a-z.txt > filtered_words.txt
   ```

2. Execute the Hashcat command:
   ```bash
   hashcat -m 3200 hash.txt filtered_words.txt
   ```

After several minutes, the correct answer will be revealed:
```
Session..........: hashcat                                           
Status...........: Cracked                                           
Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))               
Hash.Target......: $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX...8wsRom                                                           
Time.Started.....: Thu Oct  3 17:11:58 2024 (12 mins, 28 secs)       
Time.Estimated...: Thu Oct  3 17:24:26 2024 (0 secs)                 
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (filtered_words.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       10 H/s (5.98ms) @ Accel:2 Loops:64 Thr:1 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 7548/17576 (42.94%)
Rejected.........: 0/7548 (0.00%)
Restore.Point....: 7544/17576 (42.92%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4032-4096
Candidate.Engine.: Device Generator
Candidates.#1....: blee -> bleh
Hardware.Mon.#1..: Util: 87%
```

### 5. MD4 Hash
- **Hash:** `279412f945939ba78ce0758d3fd83daa`
- **Plaintext:** `Eternity22`

## Level 2

### 1. SHA256 Hash
- **Hash:** `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85`
- **Plaintext:** `paule`
- **Command:** 
   ```bash
   hashcat -m 1400 hash.txt --show
   ```

### 2. NTLM Hash
- **Hash:** `1DFECA0C002AE40B8619ECF94819CC1B`
- **Plaintext:** `pn63umy8lkf4i`

### 3. SHA-512 Hash
- **Hash:** `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`
- **Plaintext:** `waka99`

### 4. HMAC-SHA1 Hash
- **Hash:** `e5d8870e5bdd26602cab8dbe07a942c8669e56d6`
- **Plaintext:** `481616481616`
