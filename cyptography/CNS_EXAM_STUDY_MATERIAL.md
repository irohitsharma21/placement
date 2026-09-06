# CRYPTOGRAPHY & NETWORK SECURITY - COMPLETE EXAM STUDY MATERIAL

### AKTU B.Tech 4th Year (Batch 2023-27) | KCS-074 | Units 1 & 2
### Exam Pattern: 20 Marks -> 5 x (1 mark) + 3 x (3 marks) + 1 x (6 marks)

---

## HOW TO USE THIS FILE

| Section | What's inside | Read when |
|---|---|---|
| **§0** | Exam strategy + blueprint | First, 5 min |
| **§1** | Security fundamentals (attacks, services, principles) | Guaranteed 1 & 3 markers |
| **§2** | Classical ciphers (Caesar, Playfair, Vigenere, Transposition, Steganography) | Very high chance of 6-marker |
| **§3** | Number theory (Euclid, Fermat, Euler, CRT, Miller-Rabin, Primitive roots) | ALL your 1 & 3 mark numericals |
| **§4** | RSA + Public key cryptosystem | THE most likely 6-marker |
| **§5** | All difference tables (ready to reproduce) | 3-markers |
| **§6** | Numerical answer-key drill (fast self-check) | Night before |
| **§7** | One-page formula sheet | Last 30 min before entering hall |
| **§8** | Examiner tips + common mistakes | Last 10 min |

---

# §0. EXAM BLUEPRINT & STRATEGY

## 0.1 What the paper will look like

```
+---------------------------------------------------------+
|  SECTION A - 5 x 1 = 5 marks   (Attempt ALL)            |
|  -> Definitions, one-line theorem statements,           |
|     small computations: phi(n), gcd, 3^61 mod 7         |
|  -> Write 1-2 lines MAX. Do not waste time.             |
+---------------------------------------------------------+
|  SECTION B - 3 x 3 = 9 marks   (Attempt ALL / choice)   |
|  -> Difference tables, short numericals (CRT, inverse,  |
|     Miller-Rabin), short explanations                   |
|  -> Write 8-12 lines + a table or small diagram         |
+---------------------------------------------------------+
|  SECTION C - 1 x 6 = 6 marks   (Long answer)            |
|  -> RSA (algorithm + full numerical)   <- 60% probable  |
|  -> Playfair / Vigenere full encryption                 |
|  -> Classical encryption techniques (all types)         |
|  -> Write 1.5-2 pages + flowchart + worked example      |
+---------------------------------------------------------+
```

## 0.2 Golden rules for writing

1. **Always start with a definition**, even in numericals. ("RSA is a public-key cryptosystem based on the difficulty of factoring large integers.")
2. **Draw a box/flowchart** for any algorithm question - examiners give marks for structure.
3. **For "difference between" - ALWAYS a table**, never paragraphs. Minimum 5 points, with a "Basis of Comparison" column.
4. **Show every modular step** in numericals: `77^2 = 5929 = 187 x 31 + 132 => 132`. Never jump.
5. **Underline / box final answers.**
6. Time budget: 1-mark -> 1.5 min each, 3-mark -> 5 min each, 6-mark -> 12 min. Total ~40 min of writing.

## 0.3 Question-to-section index (from your teacher's doc)

| Doc Q. No. | Topic | Section here |
|---|---|---|
| Q1, Q31 | Define cryptography, security services | §1.1 |
| Q2 | Stream vs Block cipher | §1.5 |
| Q3, Q19 | Fermat's Little Theorem | §3.4 |
| Q4, Q10 | Substitution & Transposition ciphers | §2.1 - §2.7 |
| Q5, Q17, Q24, Q30, Q33 | RSA | §4 |
| Q6 | Active vs Passive attacks | §1.4 |
| Q7, Q14, Q21 | Euclid's GCD | §3.2 |
| Q8, Q20, Q25, Q28 | Euler's Totient phi(n) | §3.5 |
| Q9 | Key security principles | §1.3 |
| Q11 | Vigenere cipher | §2.4 |
| Q12, Q34 | Multiplicative inverse / Extended Euclid | §3.3 |
| Q13 | Miller-Rabin primality test | §3.8 |
| Q15 | Primitive roots | §3.7 |
| Q16 | Steganography | §1.7 |
| Q18, Q22, Q27, Q29 | Chinese Remainder Theorem | §3.6 |
| Q19b | Caesar cipher decryption | §2.2 |
| Q23 | Playfair cipher | §2.3 |
| Q26 | 3^61 mod 7 | §3.4 |
| Q32 | Public vs Private key | §1.6 |

---

# §1. SECURITY FUNDAMENTALS

## §1.1 Q1 / Q31. Define cryptography and state any two security services. *(1-3 marks)*

**Definition:**

> **Cryptography** is the science and art of transforming intelligible information (**plaintext**) into an unintelligible form (**ciphertext**) using a **key** and a set of mathematical transformations, so that only an authorised party possessing the correct key can recover the original message.

*Etymology:* Greek **kryptos** (hidden) + **graphein** (writing).

**Basic model (draw this):**

```
          KEY (K)                                KEY (K)
             |                                      |
             v                                      v
 Plaintext  +-------------+   Ciphertext   +-------------+  Plaintext
    (P) --->| ENCRYPTION  |------------->  | DECRYPTION  |---> (P)
            |  C = E(K,P) |   (insecure    |  P = D(K,C) |
            +-------------+    channel)    +-------------+
                                  |
                                  v
                             CRYPTANALYST
                              (Attacker)
```

**The 3 dimensions on which cryptographic systems are classified** (write for extra marks):

1. **Type of operation** - Substitution / Transposition
2. **Number of keys used** - Symmetric (single key) / Asymmetric (two keys)
3. **Way plaintext is processed** - Block cipher / Stream cipher

**Two security services (state any two):**

| Service | Meaning |
|---|---|
| **Confidentiality** | Protecting data from unauthorised disclosure; only the intended receiver can read it. |
| **Authentication** | Assurance that the communicating entity is the one it claims to be. |
| *Integrity* | Data received is exactly as sent - no modification, insertion, deletion or replay. |
| *Non-repudiation* | Sender cannot later deny sending; receiver cannot deny receiving. |
| *Access Control* | Prevention of unauthorised use of a resource. |
| *Availability* | Resource is accessible to authorised entities on demand. |

**Extra point worth writing:** These services are defined in **ITU-T X.800 (OSI Security Architecture)**, which defines three concepts - **Security Attack, Security Mechanism, Security Service.**

**Related terms (if 3 marks):**
- **Cryptanalysis** - science of breaking ciphers *without* knowing the key.
- **Cryptology** = Cryptography + Cryptanalysis.

---

## §1.2 The OSI Security Architecture (X.800) - bonus 3-marker

```
                    X.800 SECURITY ARCHITECTURE
                              |
        +---------------------+---------------------+
        |                     |                     |
  SECURITY ATTACK      SECURITY MECHANISM     SECURITY SERVICE
  (any action that     (process designed      (a service that
   compromises the      to detect/prevent/     enhances security
   security of info)    recover from attack)   of data transfer)
        |                     |                     |
  Passive / Active     Encipherment,          Authentication,
                       Digital Signature,     Access Control,
                       Access Control,        Data Confidentiality,
                       Data Integrity,        Data Integrity,
                       Traffic Padding,       Non-repudiation,
                       Notarization           Availability
```

**Definitions to memorise:**
- **Security Attack** - Any action that compromises the security of information owned by an organisation.
- **Security Mechanism** - A process designed to detect, prevent or recover from a security attack.
- **Security Service** - A processing/communication service that enhances the security of data processing systems and information transfers.

---

## §1.3 Q9. Outline the key principles of security. *(3 marks)*

```
                    SECURITY PRINCIPLES
                            |
        +-------------------+-------------------+
        |                   |                   |
   [ CIA TRIAD ]     [ ADDITIONAL ]      [ SUPPORTING ]
        |                   |                   |
 +------+------+      +-----+-----+       +-----+-----+
 |      |      |      |           |       |           |
Confi- Integ- Avail- Authenti-  Non-repu- Access   Account-
denti- rity   ability cation    diation   Control  ability
ality
```

**1. Confidentiality** - Information is accessible only to authorised parties.
*Threatened by:* interception / snooping. *Achieved by:* encryption.

**2. Integrity** - Information can be modified only by authorised parties and only in authorised ways.
*Threatened by:* modification. *Achieved by:* hash functions, MAC.

**3. Availability** - Resources are available to authorised users whenever required.
*Threatened by:* Denial-of-Service (DoS). *Achieved by:* redundancy, backups, firewalls.

**4. Authentication** - Verifying the identity of the sender/entity.
*Threatened by:* masquerade / fabrication. *Achieved by:* digital signatures, passwords, biometrics.

**5. Non-repudiation** - Neither sender nor receiver can deny the transaction.
*Achieved by:* digital signatures, audit logs.

**6. Access Control** - Determining and enforcing who can access what resource.

**7. Accountability / Auditability** - Actions of an entity can be traced uniquely to that entity.

**Mnemonic: "C-I-A + A-N-A"** (Confidentiality, Integrity, Availability + Authentication, Non-repudiation, Access control)

---

## §1.4 Q6. Distinguish between Active and Passive attacks. *(3 marks)*

```
                    SECURITY ATTACKS
                          |
            +-------------+--------------+
            |                            |
      PASSIVE ATTACK               ACTIVE ATTACK
      (only observe)               (modify / create)
            |                            |
    +-------+--------+        +-----+----+-----+--------+
    |                |        |     |          |        |
Release of      Traffic   Masque-  Replay  Modification DoS
Message         Analysis   rade                of Msg
Contents
```

| Basis | **Passive Attack** | **Active Attack** |
|---|---|---|
| **Definition** | Attacker only **eavesdrops / monitors** transmission; does not alter data | Attacker **modifies the data stream** or creates a false stream |
| **Threat to** | **Confidentiality** | **Integrity & Availability** |
| **Modification** | No modification of information | Information is modified / fabricated |
| **Detection** | **Very difficult to detect** (leaves no trace) | **Comparatively easy to detect** |
| **Prevention** | **Prevention is feasible** (encryption) | Prevention is difficult |
| **Emphasis** | Emphasis is on **prevention** | Emphasis is on **detection and recovery** |
| **Harm to system** | Does not harm the system | Harms the system / disrupts service |
| **Victim awareness** | Victim is unaware | Victim eventually notices the effect |
| **Types** | 1. Release of message contents  2. Traffic analysis | 1. Masquerade 2. Replay 3. Modification of messages 4. Denial of Service |
| **Example** | Wiretapping a line to listen to a conversation | Injecting a fake "transfer Rs.10,000" message |

**Closing line for the answer:** *"Passive attacks are hard to detect but easy to prevent; active attacks are hard to prevent but easier to detect."*

**Four categories of attack (bonus diagram):**

```
Normal flow:      Source -------------> Destination

Interruption:     Source ----X          Destination   (Availability)
Interception:     Source ------+------> Destination   (Confidentiality)
                               v
                            Attacker
Modification:     Source ---[Attacker]--> Destination (Integrity)
Fabrication:                [Attacker]--> Destination (Authenticity)
```

---

## §1.5 Q2. Difference between Stream Cipher and Block Cipher. *(3 marks)*

```
STREAM CIPHER                        BLOCK CIPHER
-------------                        ------------
Plaintext:  b1 b2 b3 b4 ...          Plaintext: [b1..b64] [b65..b128]
            XOR XOR XOR XOR                        |            |
Keystream:  k1 k2 k3 k4 ...                   +----v----+  +----v----+
             |  |  |  |                       | ENCRYPT |  | ENCRYPT |
Ciphertext: c1 c2 c3 c4 ...                   |  (Key)  |  |  (Key)  |
                                              +----+----+  +----+----+
one bit / byte at a time                        [C1..C64]  [C65..C128]
```

| Basis | **Stream Cipher** | **Block Cipher** |
|---|---|---|
| **Unit of processing** | Encrypts **1 bit / 1 byte** at a time | Encrypts a **fixed-size block** (64 / 128 bits) at a time |
| **Working principle** | Plaintext XOR Keystream | Complex substitution + permutation over several rounds |
| **Key used** | A pseudorandom **keystream** as long as the plaintext | A single fixed-length key reused for every block |
| **Speed** | **Faster**, low hardware complexity | **Slower**, higher complexity |
| **Memory requirement** | Requires **less memory** | Requires **more memory** |
| **Error propagation** | A bit error affects **only that bit** | A bit error corrupts the **entire block** |
| **Confusion & Diffusion** | Uses only **confusion** | Uses both **confusion and diffusion** |
| **Padding** | Not required | Required if last block is incomplete |
| **Suitability** | Real-time data - voice, video, wireless | File / bulk data encryption, databases |
| **Examples** | **RC4, A5/1, SEAL, ChaCha20** | **DES, 3DES, AES, Blowfish, IDEA** |

---

## §1.6 Q32. Differentiate between Public Key and Private Key. *(3 marks)*

| Basis | **Public Key (Asymmetric)** | **Private Key (Symmetric)** |
|---|---|---|
| **Number of keys** | A **pair** of keys (PU, PR) is used | The **same single key** is shared |
| **Secrecy** | Public key is **openly published**, private key kept secret | Key must be kept **secret by both parties** |
| **Key distribution** | No secure channel needed for the public key | Needs a **secure channel** for key exchange (major problem) |
| **Keys for n users** | **2n** keys | **n(n-1)/2** keys |
| **Speed** | **Slow** - modular exponentiation on very large numbers | **Fast** - bit-level XOR / substitution |
| **Key length** | Large (1024-4096 bits for RSA) | Small (56-256 bits) |
| **Use case** | Key exchange, digital signature, authentication | Bulk data encryption |
| **Basis of security** | Hard mathematical problems (integer factorisation, discrete logarithm) | Confusion and diffusion |
| **Examples** | **RSA, Diffie-Hellman, ECC, ElGamal, DSS** | **DES, AES, 3DES, Blowfish, RC4, IDEA** |

**Diagram to draw:**

```
   PUBLIC KEY (Asymmetric)                PRIVATE KEY (Symmetric)

  Alice                Bob                Alice                Bob
    |  Encrypt with      |                  |  Encrypt with      |
    |  Bob's PUBLIC key  |                  |  SHARED SECRET K   |
    +------------------->|                  +------------------->|
    |                    | Decrypt with     |                    | Decrypt with
    |                    | Bob's PRIVATE key|                    | the SAME K
```

---

## §1.7 Q16. Define Steganography with example. *(1-3 marks)*

**Definition:**

> **Steganography** (Greek: *steganos* = covered, *graphein* = writing) is the technique of **hiding the very existence of a message** by concealing it inside another harmless-looking carrier medium (text, image, audio, video), so that an observer does not even suspect that secret communication is taking place.

**Key idea:** *Cryptography hides the MEANING; Steganography hides the EXISTENCE.*

**Examples:**

**(a) Character marking / word-order method** - take the first letter of each word:

```
Message sent :  "Send  Every  Now  Dollars"
Hidden text  :   S     E      N    D        ->  "SEND"
```

**(b) Classic newspaper method** - pin-pricks placed over selected letters of a newspaper article spell out the secret message.

**(c) LSB (Least Significant Bit) insertion in images** - the most important modern method:

```
Cover pixel bytes  : 10010101   00001100   11010101
Secret bits        :        1          0          1
Stego pixel bytes  : 1001010[1] 0000110[0] 1101010[1]
```

Changing only the LSB alters the colour so slightly that the human eye cannot perceive any difference.

**Other classical techniques:** invisible ink, pin punctures, typewriter correction ribbon, microdots.

### Cryptography vs Steganography (likely 3-marker)

| Basis | **Cryptography** | **Steganography** |
|---|---|---|
| **What is hidden** | The **meaning** of the message | The **existence** of the message |
| **Output visibility** | Ciphertext is visible but unreadable (scrambled) | Nothing suspicious is visible |
| **Structure of data** | Structure is changed | Structure is **not** changed |
| **Key requirement** | Key is essential | Key is optional (stego-key) |
| **Attacker's suspicion** | Attacker knows a secret exists | Attacker does not suspect anything |
| **Attack name** | **Cryptanalysis** | **Steganalysis** |
| **Overhead** | Small (ciphertext ~ plaintext size) | Large carrier file needed for a small payload |
| **Main drawback** | Draws attention (visible garbage) | Once detected, the message is fully exposed |
| **Example** | AES, RSA, Caesar cipher | LSB insertion, invisible ink |

**Best-practice line:** *"Steganography and cryptography are complementary - the strongest practice is to first encrypt the message and then hide the ciphertext, giving two layers of protection."*

---

# §2. CLASSICAL ENCRYPTION TECHNIQUES

## §2.1 Q4 / Q10. Substitution and Transposition techniques *(6 marks - very likely)*

**Master classification diagram - draw this first:**

```
                CLASSICAL ENCRYPTION TECHNIQUES
                            |
        +-------------------+--------------------+
        |                                        |
  SUBSTITUTION TECHNIQUE               TRANSPOSITION TECHNIQUE
  (letters are REPLACED by             (letters are REARRANGED /
   other letters/numbers/symbols;       positions permuted;
   POSITION unchanged)                  IDENTITY unchanged)
        |                                        |
   +----+----+----+----+----+----+          +----+-----+
   |    |    |    |    |    |    |          |          |
Caesar Mono- Play- Hill Poly- One-      Rail Fence  Row-Column
      alpha- fair       alpha- Time                (Columnar)
      betic              betic  Pad
                       (Vigenere)
```

**Core definitions (write both):**

- **Substitution cipher:** A technique in which each letter (or bit/byte) of the plaintext is **replaced** by some other letter, number or symbol. The *position* of the character in the message does not change, only its *identity*.
- **Transposition cipher:** A technique that performs some sort of **permutation** on the plaintext letters. The *identity* of the characters is not changed, only their *position/order*.

### Comparison table (guaranteed marks)

| Basis | **Substitution Cipher** | **Transposition Cipher** |
|---|---|---|
| **Operation** | Replaces characters with other characters | Rearranges the positions of characters |
| **Identity of letter** | Changed | Not changed |
| **Position of letter** | Not changed | Changed |
| **Letter frequency** | Frequency distribution is **changed** | Frequency distribution is **preserved** (same letters) |
| **Cryptanalysis clue** | Frequency analysis on ciphertext letters | Ciphertext letter frequency = plaintext frequency, so a transposition is easily *identified* |
| **Uses** | Confusion | Diffusion |
| **Examples** | Caesar, Monoalphabetic, Playfair, Hill, Vigenere, One-time pad | Rail Fence, Row-Column (Columnar) transposition |

---

## §2.2 Caesar Cipher (+ Q19b numerical)

**Definition:** The earliest known substitution cipher, proposed by Julius Caesar. Each letter of the plaintext is replaced by the letter standing **k places further down** the alphabet (cyclically). Here k = 3 classically.

**Formulae:**

```
Encryption : C = E(k, p) = (p + k) mod 26
Decryption : p = D(k, C) = (C - k) mod 26
```

Letter-to-number mapping: A=0, B=1, C=2, ... Z=25.

**Key space:** Only 25 possible keys -> **brute force attack breaks it instantly**. This is its main weakness.

### NUMERICAL Q19b: Apply Caesar cipher, decrypt ciphertext "PHHW PH", k = 3

```
p = D(3, C) = (C - 3) mod 26

C:  P    H    H    W       P    H
    |    |    |    |       |    |
    15   7    7    22      15   7        <- numeric value
    -3   -3   -3   -3      -3   -3
    ---  ---  ---  ---     ---  ---
    12   4    4    19      12   4
    |    |    |    |       |    |
p:  M    E    E    T       M    E
```

**ANSWER: Plaintext = "MEET ME"**

**Verification (encrypt back):** M(12)+3=15=P, E(4)+3=7=H, E(4)+3=7=H, T(19)+3=22=W -> "PHHW". Correct.

**Weakness to mention:** Since there are only 25 keys, an attacker simply tries all 25 shifts (brute force) and looks for meaningful text.

---

## §2.3 Q23. Playfair Cipher *(6 marks - high probability)*

**Definition:** The Playfair cipher is a **multiple-letter (digram) substitution** cipher invented by **Charles Wheatstone in 1854** and promoted by **Lord Playfair**. It encrypts **pairs of letters (digrams)** instead of single letters, using a **5 x 5 matrix** built from a keyword. Because it operates on 26 x 26 = 676 digrams, frequency analysis becomes much harder than for a monoalphabetic cipher.

### Step 1: Construct the 5 x 5 key matrix

Rules:
1. Fill the keyword letter by letter, **omitting repeated letters**.
2. Fill the remaining cells with the rest of the alphabet in order.
3. **I and J are treated as one letter** (25 cells only).

### Step 2: Prepare the plaintext (digram rules)

1. Break the plaintext into pairs of two letters.
2. If both letters of a pair are the **same**, insert a filler **'X'** between them.
3. If the total number of letters is **odd**, append **'X'** at the end.

### Step 3: Encryption rules (memorise this table)

| Case | Rule |
|---|---|
| **Same ROW** | Replace each letter by the letter to its **RIGHT** (wrap around to the first column) |
| **Same COLUMN** | Replace each letter by the letter **BELOW** it (wrap around to the top row) |
| **Rectangle (different row & column)** | Replace each letter by the letter in its **OWN ROW** but in the **COLUMN of the other letter** |

*(Decryption is exactly the reverse: LEFT for same row, UP for same column, same rule for rectangle.)*

### NUMERICAL Q23: Key = "playfair", Plaintext = "hide the gold in the treestump"

**(a) Build the matrix.** Key "PLAYFAIR" -> remove duplicates -> **P, L, A, Y, F, I, R**
Remaining alphabet (I/J merged, skipping used letters): B, C, D, E, G, H, K, M, N, O, Q, S, T, U, V, W, X, Z

```
        c1   c2   c3   c4   c5
      +----+----+----+----+----+
 r1   | P  | L  | A  | Y  | F  |
      +----+----+----+----+----+
 r2   | I/J| R  | B  | C  | D  |
      +----+----+----+----+----+
 r3   | E  | G  | H  | K  | M  |
      +----+----+----+----+----+
 r4   | N  | O  | Q  | S  | T  |
      +----+----+----+----+----+
 r5   | U  | V  | W  | X  | Z  |
      +----+----+----+----+----+
```

**(b) Prepare digrams.** Plaintext (spaces removed): `hidethegoldinthetreestump` (25 letters)

Pairing left to right: `HI DE TH EG OL DI NT HE TR EE ...`
The pair **EE** has two identical letters -> insert **X** -> becomes **EX**, and the remaining stream shifts.

Final digrams:
```
HI  DE  TH  EG  OL  DI  NT  HE  TR  EX  ES  TU  MP
```
(26 letters - even, so no trailing X is needed.)

**(c) Encrypt each digram:**

| # | Digram | Positions | Case | Cipher |
|---|---|---|---|---|
| 1 | **HI** | H(3,3), I(2,1) | Rectangle | H->(3,1)=E, I->(2,3)=B -> **EB** |
| 2 | **DE** | D(2,5), E(3,1) | Rectangle | D->(2,1)=I, E->(3,5)=M -> **IM** |
| 3 | **TH** | T(4,5), H(3,3) | Rectangle | T->(4,3)=Q, H->(3,5)=M -> **QM** |
| 4 | **EG** | E(3,1), G(3,2) | Same row 3 | E->(3,2)=G, G->(3,3)=H -> **GH** |
| 5 | **OL** | O(4,2), L(1,2) | Same col 2 | O->(5,2)=V, L->(2,2)=R -> **VR** |
| 6 | **DI** | D(2,5), I(2,1) | Same row 2 | D->(2,1)=I, I->(2,2)=R -> **IR** |
| 7 | **NT** | N(4,1), T(4,5) | Same row 4 | N->(4,2)=O, T->wrap(4,1)=N -> **ON** |
| 8 | **HE** | H(3,3), E(3,1) | Same row 3 | H->(3,4)=K, E->(3,2)=G -> **KG** |
| 9 | **TR** | T(4,5), R(2,2) | Rectangle | T->(4,2)=O, R->(2,5)=D -> **OD** |
| 10 | **EX** | E(3,1), X(5,4) | Rectangle | E->(3,4)=K, X->(5,1)=U -> **KU** |
| 11 | **ES** | E(3,1), S(4,4) | Rectangle | E->(3,4)=K, S->(4,1)=N -> **KN** |
| 12 | **TU** | T(4,5), U(5,1) | Rectangle | T->(4,1)=N, U->(5,5)=Z -> **NZ** |
| 13 | **MP** | M(3,5), P(1,1) | Rectangle | M->(3,1)=E, P->(1,5)=F -> **EF** |

### **FINAL ANSWER (Ciphertext):**

```
EB IM QM GH VR IR ON KG OD KU KN NZ EF

=  EBIMQMGHVRIRONKGODKUKNNZEF
```

**Points to add for full marks:**
- Playfair was used by the British Army in WWI and by the Australian Army in WWII.
- **Strength:** 676 digrams vs 26 letters -> relative frequencies of digrams are much flatter, so far more ciphertext is needed for analysis.
- **Weakness:** Still leaks plaintext structure; a few hundred letters of ciphertext are enough to break it. It was broken as early as WWI.

---

## §2.4 Q11. Vigenere Cipher *(6 marks - high probability)*

**Definition:** The Vigenere cipher is the best known **polyalphabetic substitution** cipher. It uses a set of **26 Caesar ciphers** with shifts 0 to 25. A **keyword** is repeated cyclically to match the length of the plaintext, and each plaintext letter is encrypted with the Caesar cipher determined by the corresponding key letter. Because a letter maps to different ciphertext letters at different positions, simple frequency analysis fails.

**Formulae:**

```
Encryption : C[i] = ( P[i] + K[i mod m] ) mod 26
Decryption : P[i] = ( C[i] - K[i mod m] ) mod 26

where m = length of the keyword
```

### NUMERICAL Q11: Plaintext = "Life is full of surprises", Key = "HEALTH"

**Step 1: Write plaintext (spaces removed, uppercase) and repeat the key.**

Plaintext: `LIFEISFULLOFSURPRISES` (21 letters)

```
Pos :  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21
P   :  L  I  F  E  I  S  F  U  L  L  O  F  S  U  R  P  R  I  S  E  S
K   :  H  E  A  L  T  H  H  E  A  L  T  H  H  E  A  L  T  H  H  E  A
```

**Step 2: Numeric values (A=0 ... Z=25) and compute C = (P + K) mod 26.**

| Pos | P | P# | K | K# | P#+K# | mod 26 | **C** |
|---|---|---|---|---|---|---|---|
| 1 | L | 11 | H | 7 | 18 | 18 | **S** |
| 2 | I | 8 | E | 4 | 12 | 12 | **M** |
| 3 | F | 5 | A | 0 | 5 | 5 | **F** |
| 4 | E | 4 | L | 11 | 15 | 15 | **P** |
| 5 | I | 8 | T | 19 | 27 | 1 | **B** |
| 6 | S | 18 | H | 7 | 25 | 25 | **Z** |
| 7 | F | 5 | H | 7 | 12 | 12 | **M** |
| 8 | U | 20 | E | 4 | 24 | 24 | **Y** |
| 9 | L | 11 | A | 0 | 11 | 11 | **L** |
| 10 | L | 11 | L | 11 | 22 | 22 | **W** |
| 11 | O | 14 | T | 19 | 33 | 7 | **H** |
| 12 | F | 5 | H | 7 | 12 | 12 | **M** |
| 13 | S | 18 | H | 7 | 25 | 25 | **Z** |
| 14 | U | 20 | E | 4 | 24 | 24 | **Y** |
| 15 | R | 17 | A | 0 | 17 | 17 | **R** |
| 16 | P | 15 | L | 11 | 26 | 0 | **A** |
| 17 | R | 17 | T | 19 | 36 | 10 | **K** |
| 18 | I | 8 | H | 7 | 15 | 15 | **P** |
| 19 | S | 18 | H | 7 | 25 | 25 | **Z** |
| 20 | E | 4 | E | 4 | 8 | 8 | **I** |
| 21 | S | 18 | A | 0 | 18 | 18 | **S** |

### **FINAL ANSWER (Ciphertext):**

```
S M F P B Z M Y L W H M Z Y R A K P Z I S

=  SMFPBZMYLWHMZYRAKPZIS
```

**Notice (great point to add):** The plaintext letter **S** at positions 6, 13, 19, 21 encrypts to **Z, Z, Z, S** - and the letter **L** maps to both **S** and **L**. This *many-to-many* mapping is exactly what defeats simple frequency analysis, which is the strength of a polyalphabetic cipher.

**Weakness:** If the keyword length *m* is discovered (via the **Kasiski test** or the **index of coincidence**), the ciphertext splits into *m* separate monoalphabetic (Caesar) ciphers, each of which is trivially broken.

**Related - One-Time Pad (Vernam):** If the key is truly random, as long as the message, and never reused, the Vigenere cipher becomes the **One-Time Pad**, which is the only **perfectly secure / unbreakable** cipher (proved by Claude Shannon). Its practical problems are key generation and key distribution.

---

## §2.5 Monoalphabetic Cipher (short note)

Instead of a fixed shift, use an arbitrary **permutation of the 26 letters** as the key.

```
Plain  : a b c d e f g h i j k l m n o p q r s t u v w x y z
Cipher : D K V Q F I B J W P E S C X H T M Y A U O L R G Z N
```

**Key space = 26! ~ 4 x 10^26** keys - brute force is infeasible.
**But** it is easily broken by **frequency analysis**, because the letter frequency pattern of English (E, T, A, O, I, N... most frequent) is directly carried into the ciphertext.

---

## §2.6 Hill Cipher (short note - sometimes asked)

A **polygraphic substitution** cipher developed by **Lester Hill (1929)**, based on **linear algebra**.

```
Encryption :  C = K * P  (mod 26)
Decryption :  P = K^-1 * C (mod 26)
```
where K is an m x m invertible matrix mod 26 (i.e. gcd(det K, 26) = 1).

**Strength:** Completely hides single-letter frequencies (an m x m Hill cipher hides m-letter frequencies).
**Weakness:** Easily broken by a **known-plaintext attack**, because the system is linear.

---

## §2.7 Transposition Techniques

### (a) Rail Fence Cipher

The simplest transposition. Plaintext is written down diagonally over a number of "rails" and then read off row by row.

**Example:** Plaintext = "meet me after the toga party", depth = 2

```
Row 1:  m   e   m   a   t   r   h   t   g   p   r   y
Row 2:    e   t   e   f   e   t   e   o   a   a   t
```

Read row-wise:
**Ciphertext = MEMATRHTGPRY ETEFETEOAAT = `MEMATRHTGPRYETEFETEOAAT`**

### (b) Row-Column (Columnar) Transposition

Write the message row by row in a rectangle, then read it out **column by column, in the order given by a numeric key**.

**Example:** Key = `4 3 1 2 5 6 7`, Plaintext = "attack postponed until two am xyz"

```
Key   :  4  3  1  2  5  6  7
         -------------------
Row 1 :  a  t  t  a  c  k  p
Row 2 :  o  s  t  p  o  n  e
Row 3 :  d  u  n  t  i  l  t
Row 4 :  w  o  a  m  x  y  z
```

Read the columns in key order 1, 2, 3, 4, 5, 6, 7:

| Key order | Column read | Letters |
|---|---|---|
| 1 | 3rd column | T T N A |
| 2 | 4th column | A P T M |
| 3 | 2nd column | T S U O |
| 4 | 1st column | A O D W |
| 5 | 5th column | C O I X |
| 6 | 6th column | K N L Y |
| 7 | 7th column | P E T Z |

**Ciphertext = `TTNAAPTMTSUOAODWCOIXKNLYPETZ`**

**Strengthening it:** Perform the columnar transposition **more than once** (double transposition) - this makes the permutation far more complex and much harder to cryptanalyse.

**Weakness of any transposition cipher:** The **letter frequency of the ciphertext is identical to that of normal English**, which immediately tells a cryptanalyst that a transposition (not a substitution) was used.

---

# §3. NUMBER THEORY FOR CRYPTOGRAPHY

## §3.1 Basic terms (write when needed)

- **Divisibility:** `b | a` means a = mb for some integer m.
- **GCD:** gcd(a, b) is the largest positive integer that divides both a and b.
- **Relatively prime / Coprime:** a and b are coprime if **gcd(a, b) = 1**.
- **Congruence:** `a ≡ b (mod n)` means n divides (a - b), i.e. a and b leave the same remainder on division by n.
- **Modular arithmetic properties:**
  - `(a + b) mod n = [(a mod n) + (b mod n)] mod n`
  - `(a x b) mod n = [(a mod n) x (b mod n)] mod n`
  - `a^k mod n = [(a mod n)^k] mod n`

---

## §3.2 Q7 / Q14 / Q21. Euclid's Algorithm for GCD *(1-3 marks - VERY frequent)*

**Principle:** For any non-negative integer a and positive integer b:

```
gcd(a, b) = gcd(b, a mod b)
```

Repeat until the remainder becomes 0; the **last non-zero remainder is the GCD**.

**Flowchart:**

```
        +-----------------+
        |   START         |
        |  Input a, b     |
        +--------+--------+
                 |
                 v
        +-----------------+
        |  Compute        |
        |  r = a mod b    |
        +--------+--------+
                 |
                 v
            +----------+     NO     +----------------+
            |  r == 0 ?|----------->|  a <- b        |
            +----+-----+            |  b <- r        |
                 | YES              +-------+--------+
                 v                          |
        +-----------------+                 |
        | GCD = b         |<----------------+
        | (last non-zero  |    (loop back)
        |  remainder)     |
        +--------+--------+
                 |
                 v
              [ STOP ]
```

### NUMERICAL Q7 / Q21: Find gcd(1970, 1066)

| Step | Division | Quotient | Remainder |
|---|---|---|---|
| 1 | 1970 = 1 x 1066 + **904** | 1 | 904 |
| 2 | 1066 = 1 x 904 + **162** | 1 | 162 |
| 3 | 904 = 5 x 162 + **94** | 5 | 94 |
| 4 | 162 = 1 x 94 + **68** | 1 | 68 |
| 5 | 94 = 1 x 68 + **26** | 1 | 26 |
| 6 | 68 = 2 x 26 + **16** | 2 | 16 |
| 7 | 26 = 1 x 16 + **10** | 1 | 10 |
| 8 | 16 = 1 x 10 + **6** | 1 | 6 |
| 9 | 10 = 1 x 6 + **4** | 1 | 4 |
| 10 | 6 = 1 x 4 + **2** | 1 | 2 |
| 11 | 4 = 2 x 2 + **0** | 2 | **0 <- STOP** |

Last non-zero remainder = **2**

### **ANSWER: gcd(1970, 1066) = 2**

*(Check: 1970 = 2 x 985, 1066 = 2 x 533; 985 and 533 share no common factor.)*

### NUMERICAL Q14: Compute gcd(24120, 1640)

| Step | Division | Remainder |
|---|---|---|
| 1 | 24120 = 14 x 1640 + **1160** | 1160 |
| 2 | 1640 = 1 x 1160 + **480** | 480 |
| 3 | 1160 = 2 x 480 + **200** | 200 |
| 4 | 480 = 2 x 200 + **80** | 80 |
| 5 | 200 = 2 x 80 + **40** | 40 |
| 6 | 80 = 2 x 40 + **0** | **0 <- STOP** |

### **ANSWER: gcd(24120, 1640) = 40**

*(Check: 24120 / 40 = 603, 1640 / 40 = 41; gcd(603, 41) = 1 since 41 is prime and 41 does not divide 603.)*

---

## §3.3 Q12 / Q34. Extended Euclidean Algorithm & Multiplicative Inverse *(3 marks)*

**Statement of the Extended Euclidean Algorithm:**

> For any two integers a and b (not both zero), the Extended Euclidean Algorithm finds not only d = gcd(a, b), but also two integers **x** and **y** (called Bezout coefficients) such that
>
> **a·x + b·y = d = gcd(a, b)**
>
> If gcd(a, b) = 1, then reducing this equation modulo b gives **a·x ≡ 1 (mod b)**, i.e. **x is the multiplicative inverse of a modulo b**.

**Existence condition (important 1-marker):** The multiplicative inverse of a modulo n **exists if and only if gcd(a, n) = 1.**

**Flowchart:**

```
   +--------------------------------+
   | Input a, n  (find a^-1 mod n)  |
   +---------------+----------------+
                   v
   +--------------------------------+
   | Run Euclid's algorithm on n, a |
   | recording every quotient       |
   +---------------+----------------+
                   v
          +--------------------+   NO
          | Is gcd(a, n) = 1 ? |------->  Inverse does NOT exist
          +--------+-----------+
                   | YES
                   v
   +--------------------------------+
   | Back-substitute the remainders |
   | to express 1 = a·x + n·y       |
   +---------------+----------------+
                   v
   +--------------------------------+
   | a^-1 = x mod n                 |
   | (add n if x is negative)       |
   +--------------------------------+
```

### NUMERICAL Q12: Compute the multiplicative inverse of 11 mod 26

**Step 1 - Euclid's algorithm on (26, 11):**

```
26 = 2 x 11 + 4        ... (i)
11 = 2 x  4 + 3        ... (ii)
 4 = 1 x  3 + 1        ... (iii)
 3 = 3 x  1 + 0        -> gcd = 1  =>  inverse EXISTS
```

**Step 2 - Back-substitution:**

```
From (iii):   1 = 4 - 1 x 3
From (ii) :   3 = 11 - 2 x 4
  =>          1 = 4 - 1 x (11 - 2 x 4)
              1 = 4 - 11 + 2 x 4
              1 = 3 x 4 - 1 x 11
From (i)  :   4 = 26 - 2 x 11
  =>          1 = 3 x (26 - 2 x 11) - 1 x 11
              1 = 3 x 26 - 6 x 11 - 1 x 11
              1 = 3 x 26 - 7 x 11
```

**Step 3 - Reduce mod 26:**

```
-7 x 11 ≡ 1 (mod 26)
So  11^-1 ≡ -7 (mod 26) = -7 + 26 = 19
```

### **ANSWER: 11^-1 mod 26 = 19**

**Verification:** 11 x 19 = 209 = 26 x 8 + 1 = 208 + 1. Correct.

*(This is exactly the value used in the affine cipher over the 26-letter alphabet.)*

### NUMERICAL Q34: Find the multiplicative inverse of 550 in GF(1759) using Extended Euclid

*(GF(1759) is a Galois field because 1759 is prime, so every non-zero element has an inverse.)*

**Step 1 - Euclid's algorithm on (1759, 550):**

```
1759 = 3 x 550 + 109       ... (i)     [3 x 550 = 1650]
 550 = 5 x 109 +   5       ... (ii)    [5 x 109 = 545]
 109 = 21 x  5 +   4       ... (iii)   [21 x 5 = 105]
   5 = 1 x   4 +   1       ... (iv)
   4 = 4 x   1 +   0       -> gcd = 1  =>  inverse EXISTS
```

**Step 2 - Back-substitution:**

```
From (iv) :  1 = 5 - 1 x 4
From (iii):  4 = 109 - 21 x 5
  =>         1 = 5 - (109 - 21 x 5)  =  22 x 5 - 1 x 109
From (ii) :  5 = 550 - 5 x 109
  =>         1 = 22 x (550 - 5 x 109) - 109
             1 = 22 x 550 - 110 x 109 - 109
             1 = 22 x 550 - 111 x 109
From (i)  :  109 = 1759 - 3 x 550
  =>         1 = 22 x 550 - 111 x (1759 - 3 x 550)
             1 = 22 x 550 - 111 x 1759 + 333 x 550
             1 = 355 x 550 - 111 x 1759
```

**Step 3 - Reduce mod 1759:**

```
355 x 550 ≡ 1 (mod 1759)
```

### **ANSWER: 550^-1 in GF(1759) = 355**

**Verification:** 550 x 355 = 195250; 1759 x 111 = 195249; 195250 - 195249 = 1. Correct.

**Tabular (exam-friendly) alternative for Extended Euclid:**

| Q | A1 | A2 | A3 | B1 | B2 | B3 |
|---|---|---|---|---|---|---|
| - | 1 | 0 | 1759 | 0 | 1 | 550 |
| 3 | 0 | 1 | 550 | 1 | -3 | 109 |
| 5 | 1 | -3 | 109 | -5 | 16 | 5 |
| 21 | -5 | 16 | 5 | 106 | -339 | 4 |
| 1 | 106 | -339 | 4 | -111 | 355 | 1 |

Since B3 = 1, **inverse = B2 = 355**.

---

## §3.4 Q3 / Q19 / Q26. Fermat's Little Theorem *(1-3 marks)*

### Q3. STATEMENT (1 mark - memorise word for word)

> **Fermat's Little Theorem:** If **p is a prime** number and **a** is a positive integer **not divisible by p** (i.e. gcd(a, p) = 1), then
>
> **a^(p-1) ≡ 1 (mod p)**
>
> **Alternative (general) form:** For **any** integer a and prime p,
>
> **a^p ≡ a (mod p)**

*(The second form needs no coprimality condition, since it also holds when p divides a.)*

### PROOF (for Q19 "state and prove")

Consider the set of non-zero residues modulo p:
```
S = { 1, 2, 3, ..., p-1 }
```

**Step 1.** Multiply every element of S by a and reduce mod p:
```
T = { a·1 mod p, a·2 mod p, ..., a·(p-1) mod p }
```

**Step 2.** *Claim: T is a permutation of S.*
- No element of T is 0, because p is prime and p does not divide a, so p cannot divide a·i for 1 <= i <= p-1.
- All elements of T are distinct: if a·i ≡ a·j (mod p), then since gcd(a, p) = 1 we may cancel a, giving i ≡ j (mod p), and as 1 <= i, j <= p-1 this forces i = j.
- So T contains p-1 distinct non-zero residues mod p, hence **T = S**.

**Step 3.** Multiply all elements of each set together:
```
(a·1)(a·2)...(a·(p-1))  ≡  1 · 2 · ... · (p-1)   (mod p)
a^(p-1) · (p-1)!        ≡  (p-1)!                (mod p)
```

**Step 4.** Since p is prime, gcd((p-1)!, p) = 1, so (p-1)! can be cancelled:
```
a^(p-1) ≡ 1 (mod p)                            [ Hence proved ]
```

### NUMERICAL Q19: Use Fermat's theorem to find a number 'a' between 0 and 72 with a ≡ 9^794 (mod 73)

**Step 1.** 73 is **prime**, and gcd(9, 73) = 1. So Fermat applies:
```
9^72 ≡ 1 (mod 73)
```

**Step 2.** Reduce the exponent 794 modulo 72:
```
794 = 72 x 11 + 2        [72 x 11 = 792,  794 - 792 = 2]
```

**Step 3.** Split the power:
```
9^794 = 9^(72 x 11 + 2)
      = (9^72)^11 x 9^2
      ≡ (1)^11 x 9^2      (mod 73)
      ≡ 81  (mod 73)
      ≡ 81 - 73
      ≡ 8   (mod 73)
```

### **ANSWER: a = 8**

### NUMERICAL Q26: Compute 3^61 mod 7

**Step 1.** 7 is prime, gcd(3, 7) = 1 -> by Fermat, `3^6 ≡ 1 (mod 7)`.

**Step 2.** Reduce the exponent mod 6:
```
61 = 6 x 10 + 1
```

**Step 3.**
```
3^61 = (3^6)^10 x 3^1 ≡ 1^10 x 3 ≡ 3 (mod 7)
```

### **ANSWER: 3^61 mod 7 = 3**

*(If the question literally means the number 361: `361 = 7 x 51 + 4`, so **361 mod 7 = 4**. Mention both if unsure - the Fermat interpretation is the intended one in a CNS paper.)*

**Application in cryptography (add this line):** Fermat's theorem is the mathematical basis for the **Fermat primality test** and, together with Euler's theorem, it is what makes **RSA decryption work**.

---

## §3.5 Q8 / Q20 / Q25 / Q28. Euler's Totient Function *(1-3 marks - almost certain)*

### DEFINITION (Q28 part 1)

> **Euler's Totient Function ϕ(n)** (also written phi(n)) is defined as the **number of positive integers less than n that are relatively prime to n**, i.e.
>
> **ϕ(n) = | { a : 1 <= a < n and gcd(a, n) = 1 } |**
>
> By convention, **ϕ(1) = 1**.

### THE FOUR FORMULAE (memorise - your 1-markers come from here)

| Case | Formula |
|---|---|
| **n = p** (prime) | **ϕ(p) = p - 1** |
| **n = p^k** (prime power) | **ϕ(p^k) = p^k - p^(k-1) = p^k (1 - 1/p)** |
| **n = p x q** (p, q distinct primes) | **ϕ(pq) = (p-1)(q-1)** |
| **General n = p1^a1 · p2^a2 ... pk^ak** | **ϕ(n) = n (1 - 1/p1)(1 - 1/p2)...(1 - 1/pk)** |

**Key property:** ϕ is **multiplicative** - if gcd(m, n) = 1 then ϕ(mn) = ϕ(m)·ϕ(n).

### Q28 PROOF: Prove that ϕ(pq) = (p-1)(q-1) where p, q are prime

Let n = pq with p and q distinct primes.

**Step 1.** Consider the complete residue set `{ 0, 1, 2, ..., pq - 1 }`, which has **pq** elements.

**Step 2.** Count the elements NOT relatively prime to n. Since the only prime factors of n = pq are p and q, an integer in this set fails to be coprime to n exactly when it is a multiple of p or a multiple of q.

- Multiples of **p** in the set: `{ 0, p, 2p, ..., (q-1)p }` -> **q** elements.
- Multiples of **q** in the set: `{ 0, q, 2q, ..., (p-1)q }` -> **p** elements.
- The element **0** is counted in both lists (it is the only common element, since p and q are distinct primes and any other common multiple would be >= pq).

**Step 3.** By the inclusion-exclusion principle, the number of integers NOT coprime to n is:
```
= q + p - 1
```

**Step 4.** Therefore
```
ϕ(pq) = pq - (p + q - 1)
      = pq - p - q + 1
      = p(q - 1) - (q - 1)
      = (p - 1)(q - 1)                     [ Hence proved ]
```

**Why this matters (write it!):** This is precisely the value used in RSA - the modulus n = pq is public, but computing ϕ(n) = (p-1)(q-1) requires knowing p and q, i.e. it requires **factoring n**. That single fact is the security foundation of RSA.

### NUMERICAL Q8: Compute ϕ(55)

```
55 = 5 x 11     (both 5 and 11 are prime, distinct)

ϕ(55) = ϕ(5) x ϕ(11) = (5 - 1)(11 - 1) = 4 x 10 = 40
```
### **ANSWER: ϕ(55) = 40**

### NUMERICAL Q20: Compute ϕ(35)

```
35 = 5 x 7      (both prime, distinct)

ϕ(35) = (5 - 1)(7 - 1) = 4 x 6 = 24
```
### **ANSWER: ϕ(35) = 24**

### NUMERICAL Q25: Find the value of ϕ(12)

```
12 = 2^2 x 3

Method 1 (general formula):
ϕ(12) = 12 x (1 - 1/2) x (1 - 1/3)
      = 12 x (1/2) x (2/3)
      = 4

Method 2 (multiplicative):
ϕ(12) = ϕ(4) x ϕ(3) = (4 - 2) x (3 - 1) = 2 x 2 = 4

Method 3 (direct listing):
Integers < 12 coprime to 12 : { 1, 5, 7, 11 }  ->  count = 4
```
### **ANSWER: ϕ(12) = 4**

**CAUTION (very common mistake):** ϕ(12) is NOT (2-1)(2-1)(3-1). The formula (p-1)(q-1) applies ONLY when p and q are **distinct primes**, i.e. n is a product of two different primes with exponent 1. For 12 = 2^2 x 3 you must use ϕ(2^2) = 4 - 2 = 2.

### Handy ϕ(n) reference table

| n | Factorisation | ϕ(n) |
|---|---|---|
| 10 | 2 x 5 | 4 |
| 12 | 2^2 x 3 | 4 |
| 13 | prime | 12 |
| 15 | 3 x 5 | 8 |
| 21 | 3 x 7 | 12 |
| 24 | 2^3 x 3 | 8 |
| 35 | 5 x 7 | 24 |
| 37 | prime | 36 |
| 55 | 5 x 11 | 40 |
| 100 | 2^2 x 5^2 | 40 |
| 187 | 11 x 17 | 160 |
| 221 | 13 x 17 | 192 |

### Euler's Theorem (state alongside - often asked with Fermat)

> For every a and n that are **relatively prime** (gcd(a, n) = 1):
>
> **a^ϕ(n) ≡ 1 (mod n)**

Fermat's Little Theorem is the special case n = p (prime), since ϕ(p) = p - 1.

---

## §3.6 Q18 / Q22 / Q27 / Q29. Chinese Remainder Theorem (CRT) *(3-6 marks - VERY frequent)*

### STATEMENT

> **Chinese Remainder Theorem:** Let m1, m2, ..., mk be **pairwise relatively prime** positive integers (gcd(mi, mj) = 1 for all i != j) and let M = m1 x m2 x ... x mk. Then the system of simultaneous congruences
>
> ```
> x ≡ a1 (mod m1)
> x ≡ a2 (mod m2)
>       ...
> x ≡ ak (mod mk)
> ```
>
> has a **unique solution modulo M**, given by
>
> **x = ( a1·M1·y1 + a2·M2·y2 + ... + ak·Mk·yk ) mod M**
>
> where **Mi = M / mi** and **yi = Mi^-1 (mod mi)** (the multiplicative inverse of Mi modulo mi).

**Why it matters in cryptography (add this line):** CRT allows a large computation modulo M to be replaced by several small computations modulo m1, m2, ... . In **RSA decryption**, using CRT with the primes p and q makes decryption roughly **4 times faster**.

### SOLVING PROCEDURE (flowchart)

```
   +-------------------------------------------+
   | Given: x ≡ ai (mod mi), i = 1..k          |
   +-----------------------+-------------------+
                           v
   +-------------------------------------------+
   | STEP 1: Check that all mi are PAIRWISE    |
   |         relatively prime                  |
   +-----------------------+-------------------+
                           v
   +-------------------------------------------+
   | STEP 2: M = m1 x m2 x ... x mk            |
   +-----------------------+-------------------+
                           v
   +-------------------------------------------+
   | STEP 3: Mi = M / mi   for each i          |
   +-----------------------+-------------------+
                           v
   +-------------------------------------------+
   | STEP 4: yi = Mi^-1 mod mi                 |
   |         (use Extended Euclid / trial)     |
   +-----------------------+-------------------+
                           v
   +-------------------------------------------+
   | STEP 5: x = SUM( ai x Mi x yi ) mod M     |
   +-----------------------+-------------------+
                           v
   +-------------------------------------------+
   | STEP 6: VERIFY x against every congruence |
   +-------------------------------------------+
```

### NUMERICAL Q18: Solve  x ≡ 2 (mod 3),  x ≡ 3 (mod 5)

```
STEP 1: m1 = 3, m2 = 5.  gcd(3,5) = 1  -> pairwise coprime, CRT applies.
STEP 2: M = 3 x 5 = 15
STEP 3: M1 = 15/3 = 5     M2 = 15/5 = 3
STEP 4: y1 = 5^-1 mod 3 :  5 ≡ 2 (mod 3);  2 x 2 = 4 ≡ 1 (mod 3)  -> y1 = 2
        y2 = 3^-1 mod 5 :  3 x 2 = 6 ≡ 1 (mod 5)                   -> y2 = 2
STEP 5: x = (a1·M1·y1 + a2·M2·y2) mod M
          = (2 x 5 x 2  +  3 x 3 x 2) mod 15
          = (20 + 18) mod 15
          = 38 mod 15
          = 8
```

### **ANSWER: x = 8 (mod 15)**, i.e. x = 8, 23, 38, 53, ...

**Verification:** 8 mod 3 = 2. 8 mod 5 = 3. Both satisfied.

### NUMERICAL Q22 / Q29: Solve  x ≡ 2 (mod 3),  x ≡ 3 (mod 5),  x ≡ 2 (mod 7)

```
STEP 1: m1=3, m2=5, m3=7 -> all pairwise coprime.
STEP 2: M = 3 x 5 x 7 = 105
STEP 3: M1 = 105/3 = 35     M2 = 105/5 = 21     M3 = 105/7 = 15
STEP 4: y1 = 35^-1 mod 3 :  35 ≡ 2 (mod 3),  2 x 2 = 4 ≡ 1 (mod 3)  -> y1 = 2
        y2 = 21^-1 mod 5 :  21 ≡ 1 (mod 5),  1 x 1 = 1 ≡ 1 (mod 5)  -> y2 = 1
        y3 = 15^-1 mod 7 :  15 ≡ 1 (mod 7),  1 x 1 = 1 ≡ 1 (mod 7)  -> y3 = 1
STEP 5: x = (2 x 35 x 2) + (3 x 21 x 1) + (2 x 15 x 1)   mod 105
          = 140 + 63 + 30   mod 105
          = 233 mod 105
          = 233 - 210
          = 23
```

### **ANSWER: x = 23 (mod 105)**

**Verification table:**

| Congruence | Check | Result |
|---|---|---|
| x ≡ 2 (mod 3) | 23 = 3 x 7 + 2 | Yes |
| x ≡ 3 (mod 5) | 23 = 5 x 4 + 3 | Yes |
| x ≡ 2 (mod 7) | 23 = 7 x 3 + 2 | Yes |

### NUMERICAL Q27: Solve  x ≡ 1 (mod 5),  x ≡ 2 (mod 7),  x ≡ 3 (mod 9),  x ≡ 4 (mod 11)

```
STEP 1: m = 5, 7, 9, 11.
        5, 7, 11 are prime; 9 = 3^2 shares no factor with 5, 7 or 11.
        -> all PAIRWISE coprime, CRT applies.

STEP 2: M = 5 x 7 x 9 x 11 = 3465

STEP 3: M1 = 3465/5  = 693
        M2 = 3465/7  = 495
        M3 = 3465/9  = 385
        M4 = 3465/11 = 315

STEP 4: y1 = 693^-1 mod 5  :  693 ≡ 3 (mod 5);  3 x 2 = 6 ≡ 1  -> y1 = 2
        y2 = 495^-1 mod 7  :  495 ≡ 5 (mod 7);  5 x 3 = 15 ≡ 1 -> y2 = 3
        y3 = 385^-1 mod 9  :  385 ≡ 7 (mod 9);  7 x 4 = 28 ≡ 1 -> y3 = 4
        y4 = 315^-1 mod 11 :  315 ≡ 7 (mod 11); 7 x 8 = 56 ≡ 1 -> y4 = 8

STEP 5: x = (1 x 693 x 2) + (2 x 495 x 3) + (3 x 385 x 4) + (4 x 315 x 8)  mod 3465
          =    1386       +     2970      +     4620      +    10080        mod 3465
          = 19056 mod 3465
          = 19056 - (3465 x 5)
          = 19056 - 17325
          = 1731
```

### **ANSWER: x = 1731 (mod 3465)**

**Verification table:**

| Congruence | Check | Result |
|---|---|---|
| x ≡ 1 (mod 5) | 1731 = 5 x 346 + 1 | Yes |
| x ≡ 2 (mod 7) | 1731 = 7 x 247 + 2 | Yes |
| x ≡ 3 (mod 9) | 1731 = 9 x 192 + 3 | Yes |
| x ≡ 4 (mod 11) | 1731 = 11 x 157 + 4 | Yes |

**Time-saving tip for small y-values:** instead of Extended Euclid, just reduce Mi mod mi and test 1, 2, 3, ... until the product is ≡ 1. For a modulus below 12 this takes seconds.

---

## §3.7 Q15. Primitive Roots *(3 marks)*

### DEFINITIONS

- **Order of an element:** The **order** of a modulo n is the smallest positive integer k such that `a^k ≡ 1 (mod n)`.
- **Primitive root:** An integer **a** is a **primitive root of n** if the order of a modulo n is exactly **ϕ(n)**. Equivalently, the powers `a^1, a^2, ..., a^ϕ(n)` produce **all** ϕ(n) integers less than n that are relatively prime to n (i.e. a *generates* the whole group).
- **Existence:** Primitive roots exist only for n = **2, 4, p^k, 2p^k** where p is an odd prime.
- **Count:** If n has a primitive root, the **number of primitive roots is ϕ(ϕ(n))**.

### NUMERICAL Q15: Find all primitive roots of 13

**Step 1.** 13 is prime, so ϕ(13) = 12.
Number of primitive roots = ϕ(ϕ(13)) = **ϕ(12) = 4**. (So we are looking for exactly 4 answers.)

**Step 2.** Test a = 2 by computing all powers of 2 mod 13:

| k | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **2^k mod 13** | 2 | 4 | 8 | 3 | 6 | 12 | 11 | 9 | 5 | 10 | 7 | **1** |

The powers of 2 generate {2,4,8,3,6,12,11,9,5,10,7,1} = **all 12 non-zero residues mod 13**, and 2^k = 1 first happens at k = 12 = ϕ(13).
Therefore **2 is a primitive root of 13.**

**Step 3.** Once one primitive root g is known, **all** primitive roots are given by
```
g^k mod n   where  gcd(k, ϕ(n)) = 1
```
Here ϕ(n) = 12, and the k in 1..12 with gcd(k, 12) = 1 are **k = 1, 5, 7, 11**.

| k | 2^k mod 13 | Primitive root |
|---|---|---|
| 1 | 2 | **2** |
| 5 | 6 | **6** |
| 7 | 11 | **11** |
| 11 | 7 | **7** |

### **ANSWER: The primitive roots of 13 are { 2, 6, 7, 11 }**

**Quick cross-check for 6:** ord(6) must be 12. 6^2 = 36 ≡ 10, 6^3 ≡ 60 ≡ 8, 6^4 ≡ 48 ≡ 9, 6^6 ≡ 6^4 x 6^2 = 9 x 10 = 90 ≡ 12 ≡ -1 (not 1), so the order is not 6, not 4, not 3, not 2 -> it must be 12. Confirmed.

**Application (write this):** Primitive roots are the basis of the **discrete logarithm problem**, which is the security foundation of the **Diffie-Hellman key exchange**, **ElGamal** and **DSA/DSS**.

---

## §3.8 Q13. Miller-Rabin Primality Test *(3-6 marks)*

### CONCEPT

The Miller-Rabin test is a **probabilistic** algorithm used to decide whether a large odd integer n is prime. It relies on two properties:

1. **Fermat's Little Theorem:** if n is prime, a^(n-1) ≡ 1 (mod n) for any a coprime to n.
2. **Square-root property:** if n is prime, the only solutions of `x^2 ≡ 1 (mod n)` are `x ≡ +1` and `x ≡ -1 (mod n)`. If any *other* square root of 1 is found, n **must be composite**.

The result is one-sided: it either proves **"composite"** with certainty, or returns **"inconclusive" (probably prime)**. Repeating the test with t different random bases a reduces the error probability to less than **(1/4)^t**.

### PSEUDOCODE (write exactly this)

```
TEST (n)
1.  Find integers k > 0 and q (odd) such that  (n - 1) = 2^k * q
2.  Select a random integer a,  1 < a < n - 1
3.  if  a^q mod n == 1  then  return "inconclusive"     // probably prime
4.  for j = 0 to k - 1 do
5.        if  a^((2^j) * q) mod n == n - 1  then  return "inconclusive"
6.  return "composite"                                  // definitely composite


// Repeated (practical) version
MILLER_RABIN (n, t)
1.  for i = 1 to t do
2.        if TEST(n) == "composite" then return "COMPOSITE"
3.  return "PRIME with probability > 1 - (1/4)^t"
```

### FLOWCHART

```
      +--------------------------------------+
      |  Input odd integer n > 2             |
      +------------------+-------------------+
                         v
      +--------------------------------------+
      |  Write n - 1 = 2^k * q,  q odd       |
      +------------------+-------------------+
                         v
      +--------------------------------------+
      |  Pick random a,  1 < a < n-1         |
      +------------------+-------------------+
                         v
      +--------------------------------------+
      |  Compute  x = a^q mod n              |
      +------------------+-------------------+
                         v
                 +---------------+  YES
                 |  x == 1 ?     |----------> "INCONCLUSIVE"
                 +-------+-------+             (probably prime)
                         | NO
                         v
              +-------------------------+
              |  j = 0                  |
              +------------+------------+
                           v
                  +-----------------+  YES
                  | x == n - 1 ?    |--------> "INCONCLUSIVE"
                  +--------+--------+
                           | NO
                           v
                  +-----------------+  YES  +---------------------+
                  |  j == k - 1 ?   |------>|  return "COMPOSITE" |
                  +--------+--------+       +---------------------+
                           | NO
                           v
                  |  x = x^2 mod n  |
                  |  j = j + 1      |
                  +-------+---------+
                          |
                          +-------> (loop back to "x == n-1 ?")
```

### NUMERICAL Q13: Apply Miller-Rabin to test the primality of n = 37

**Step 1 - Express n - 1 in the form 2^k · q with q odd:**
```
n - 1 = 36
36 = 2 x 18 = 2 x 2 x 9 = 2^2 x 9

=>  k = 2,  q = 9   (9 is odd - correct form)
```

**Step 2 - Choose a base:** let **a = 2** (any a with 1 < a < 36 is valid).

**Step 3 - Compute a^q mod n:**
```
2^9 = 512
512 mod 37 :  37 x 13 = 481
              512 - 481 = 31

=>  2^9 ≡ 31 (mod 37)
```
Is it 1? **No** (31 != 1). Is it n-1 = 36? **No**. So continue the loop.

**Step 4 - j = 0 already checked (that was a^(2^0 · q) = a^q = 31). Now j = 1:**
```
a^(2^1 x q) = (2^9)^2 = 31^2 = 961
961 mod 37 :  37 x 25 = 925
              961 - 925 = 36

=>  2^18 ≡ 36 ≡ (n - 1)  (mod 37)
```

**The condition `a^(2^j · q) ≡ n - 1` is SATISFIED at j = 1.**

**Step 5 - Return "INCONCLUSIVE".**

### **ANSWER: 37 passes the Miller-Rabin test for base a = 2, so the algorithm returns "inconclusive", i.e. 37 is PROBABLY PRIME.**
*(37 is in fact prime, so the test is consistent.)*

**Summary table for the answer sheet:**

| j | Quantity computed | Value mod 37 | Condition met? |
|---|---|---|---|
| - | 2^9 (= a^q) | 31 | != 1, != 36 -> continue |
| 1 | 2^18 (= a^(2q)) | 36 = n-1 | **YES -> inconclusive (probably prime)** |

**Sanity check with Fermat:** 2^36 = (2^18)^2 ≡ 36^2 ≡ (-1)^2 ≡ 1 (mod 37). Consistent with Fermat's Little Theorem.

**Points to add:**
- The test is **probabilistic, not deterministic** - it can never prove primality, only compositeness.
- A composite n that passes the test for base a is called a **strong pseudoprime to base a**.
- With t = 10 independent bases the error probability is below (1/4)^10 ~ 10^-6, which is acceptable for generating RSA primes.

---

# §4. PUBLIC KEY CRYPTOGRAPHY & RSA

## §4.1 Q33 (part a). Discuss Public Key Cryptosystem *(3 marks)*

**Definition:**
> A **public-key (asymmetric) cryptosystem** uses **two mathematically related but different keys** - a **public key (PU)** which is published openly, and a **private key (PR)** which is kept secret by its owner. What one key encrypts, only the other can decrypt. It is computationally infeasible to determine the private key from the public key.

**Concept proposed by:** **Diffie and Hellman, 1976** ("New Directions in Cryptography").

**Six ingredients of a public-key cryptosystem (write these):**
1. **Plaintext** - readable message input
2. **Encryption algorithm** - performs transformations on plaintext
3. **Public key** - one of the pair, made public
4. **Private key** - the other of the pair, kept secret
5. **Ciphertext** - scrambled output
6. **Decryption algorithm** - recovers the plaintext

**Two modes of operation (important diagram):**

```
(A) CONFIDENTIALITY  -  Encrypt with RECEIVER's PUBLIC key

  Alice                                              Bob
    |   C = E(PU_bob, M)                              |
    +-----------------------------------------------> |
                                        M = D(PR_bob, C)
  Only Bob has PR_bob  ->  only Bob can read it.


(B) AUTHENTICATION / DIGITAL SIGNATURE  -  Encrypt with SENDER's PRIVATE key

  Alice                                              Bob
    |   C = E(PR_alice, M)                            |
    +-----------------------------------------------> |
                                      M = D(PU_alice, C)
  Only Alice has PR_alice -> proves Alice sent it (non-repudiation).
  But no confidentiality (everyone has PU_alice).

(C) BOTH  ->  C = E(PU_bob, E(PR_alice, M))   (sign then encrypt)
```

**Requirements for a public-key algorithm (Diffie-Hellman conditions):**
1. Easy for a party B to generate a key pair (PU_b, PR_b).
2. Easy for sender A to compute C = E(PU_b, M).
3. Easy for receiver B to compute M = D(PR_b, C).
4. **Computationally infeasible** for an opponent knowing PU_b to determine PR_b.
5. **Computationally infeasible** for an opponent knowing PU_b and C to recover M.
6. (Useful) The two keys can be applied in either order: M = D(PU_b, E(PR_b, M)).

**Applications:** Encryption/decryption, Digital signatures, Key exchange.

---

## §4.2 Q5 / Q17 / Q24 / Q30 / Q33. RSA ALGORITHM *(6 marks - THE most likely long question)*

### INTRODUCTION

> **RSA** (named after **Rivest, Shamir and Adleman**, MIT, 1977) is the most widely used **public-key cryptosystem**. It is a **block cipher** in which plaintext and ciphertext are integers between 0 and n-1 for some modulus n. Its security rests on the fact that **multiplying two large primes is easy, but factoring their product back into the primes is computationally infeasible.**

### THE ALGORITHM - THREE PHASES

```
+==========================================================+
|  PHASE 1 : KEY GENERATION                                |
+==========================================================+
|  1. Select two large distinct PRIME numbers  p and q     |
|  2. Compute the modulus         n = p x q                |
|  3. Compute Euler's totient  phi(n) = (p-1)(q-1)         |
|  4. Select the public exponent e such that               |
|         1 < e < phi(n)   AND   gcd(e, phi(n)) = 1        |
|  5. Compute the private exponent d such that             |
|         d x e ≡ 1 (mod phi(n))                           |
|         i.e.  d = e^-1 mod phi(n)   [Extended Euclid]    |
|                                                          |
|     PUBLIC  KEY  PU = { e, n }   -> published            |
|     PRIVATE KEY  PR = { d, n }   -> kept secret          |
|     p, q and phi(n) are DISCARDED / kept secret          |
+==========================================================+
|  PHASE 2 : ENCRYPTION  (done by sender, using PU)        |
+==========================================================+
|     Plaintext  M   with   0 <= M < n                     |
|     Ciphertext C = M^e mod n                             |
+==========================================================+
|  PHASE 3 : DECRYPTION  (done by receiver, using PR)      |
+==========================================================+
|     Plaintext  M = C^d mod n                             |
+==========================================================+
```

### FLOWCHART

```
              +---------------------------+
              |  Choose primes p, q       |
              +-------------+-------------+
                            v
              +---------------------------+
              |  n = p x q                |
              |  phi(n) = (p-1)(q-1)      |
              +-------------+-------------+
                            v
              +---------------------------+
              |  Choose e :               |
              |  1 < e < phi(n),          |
              |  gcd(e, phi(n)) = 1       |
              +-------------+-------------+
                            v
              +---------------------------+
              |  d = e^-1 mod phi(n)      |
              |  (Extended Euclid)        |
              +-------------+-------------+
                            v
        +-------------------+-------------------+
        |                                       |
  PU = {e, n}  (public)                 PR = {d, n} (private)
        |                                       |
        v                                       v
+---------------------+              +----------------------+
|  SENDER             |              |  RECEIVER            |
|  C = M^e mod n      |------------->|  M = C^d mod n       |
+---------------------+  ciphertext  +----------------------+
```

### WHY RSA WORKS (proof of correctness - add for 6 marks)

Since `e·d ≡ 1 (mod ϕ(n))`, we can write `e·d = 1 + k·ϕ(n)` for some integer k. Then:

```
C^d mod n = (M^e)^d mod n
          = M^(ed) mod n
          = M^(1 + k·phi(n)) mod n
          = M x (M^phi(n))^k mod n
          = M x (1)^k mod n            [ by EULER'S THEOREM, since gcd(M,n)=1 ]
          = M mod n
```
So decryption correctly recovers M. **(This is why Euler's theorem is the mathematical heart of RSA.)**

---

## §4.3 Q17 / Q33. NUMERICAL: p = 17, q = 11, e = 7, d = 23

### KEY GENERATION

```
STEP 1 :  p = 17,  q = 11      (both prime)

STEP 2 :  n = p x q = 17 x 11 = 187

STEP 3 :  phi(n) = (p-1)(q-1) = 16 x 10 = 160

STEP 4 :  e = 7.  Check gcd(7, 160):
              160 = 22 x 7 + 6
                7 =  1 x 6 + 1
                6 =  6 x 1 + 0   ->  gcd = 1   VALID

STEP 5 :  d must satisfy  d x e ≡ 1 (mod 160)
              d = 23  ->  7 x 23 = 161 = 160 x 1 + 1 ≡ 1 (mod 160)   VALID
```

### **PUBLIC KEY  PU = { e, n } = { 7, 187 }**
### **PRIVATE KEY PR = { d, n } = { 23, 187 }**

---

### ENCRYPTION of M = 77  (Q17)

```
C = M^e mod n = 77^7 mod 187
```

Use **square-and-multiply**. Write 7 = 4 + 2 + 1, so `77^7 = 77^4 x 77^2 x 77^1`.

| Power | Computation | Reduction mod 187 | Result |
|---|---|---|---|
| 77^1 | - | - | **77** |
| 77^2 | 77 x 77 = 5929 | 5929 = 187 x 31 + 132 | **132** |
| 77^4 | 132 x 132 = 17424 | 17424 = 187 x 93 + 33 | **33** |

Now combine:
```
77^7 = 77^4 x 77^2 x 77^1
     ≡  33  x  132 x  77   (mod 187)

33 x 132 = 4356 ;   4356 = 187 x 23 + 55        -> 55
55 x 77  = 4235 ;   4235 = 187 x 22 + 121       -> 121
```

### **CIPHERTEXT C = 121**

---

### DECRYPTION of C = 121  (Q17)

```
M = C^d mod n = 121^23 mod 187
```

Write 23 = 16 + 4 + 2 + 1.

| Power | Computation | Reduction mod 187 | Result |
|---|---|---|---|
| 121^1 | - | - | **121** |
| 121^2 | 121 x 121 = 14641 | 14641 = 187 x 78 + 55 | **55** |
| 121^4 | 55 x 55 = 3025 | 3025 = 187 x 16 + 33 | **33** |
| 121^8 | 33 x 33 = 1089 | 1089 = 187 x 5 + 154 | **154** |
| 121^16 | 154 x 154 = 23716 | 23716 = 187 x 126 + 154 | **154** |

Combine:
```
121^23 = 121^16 x 121^4 x 121^2 x 121^1
       ≡  154   x   33  x   55  x  121   (mod 187)

154 x 33  = 5082  ;  5082  = 187 x 27 + 33     -> 33
 33 x 55  = 1815  ;  1815  = 187 x 9  + 132    -> 132
132 x 121 = 15972 ;  15972 = 187 x 85 + 77     -> 77
```

### **RECOVERED PLAINTEXT M = 77** - matches the original. Verified.

---

### ENCRYPTION / DECRYPTION of M = 88  (Q33 - same keys)

**Encryption:** `C = 88^7 mod 187`

| Power | Computation | Reduction mod 187 | Result |
|---|---|---|---|
| 88^1 | - | - | **88** |
| 88^2 | 88 x 88 = 7744 | 7744 = 187 x 41 + 77 | **77** |
| 88^4 | 77 x 77 = 5929 | 5929 = 187 x 31 + 132 | **132** |

```
88^7 = 88^4 x 88^2 x 88^1 ≡ 132 x 77 x 88  (mod 187)

132 x 77 = 10164 ;  10164 = 187 x 54 + 66     -> 66
 66 x 88 = 5808  ;   5808 = 187 x 31 + 11     -> 11
```
### **CIPHERTEXT C = 11**

**Decryption:** `M = 11^23 mod 187`

| Power | Computation | Reduction mod 187 | Result |
|---|---|---|---|
| 11^1 | - | - | **11** |
| 11^2 | 11 x 11 = 121 | < 187 | **121** |
| 11^4 | 121 x 121 = 14641 | 187 x 78 + 55 | **55** |
| 11^8 | 55 x 55 = 3025 | 187 x 16 + 33 | **33** |
| 11^16 | 33 x 33 = 1089 | 187 x 5 + 154 | **154** |

```
11^23 = 11^16 x 11^4 x 11^2 x 11^1 ≡ 154 x 55 x 121 x 11  (mod 187)

154 x 55  = 8470 ;  8470 = 187 x 45 + 55     -> 55
 55 x 121 = 6655 ;  6655 = 187 x 35 + 110    -> 110
110 x 11  = 1210 ;  1210 = 187 x 6  + 88     -> 88
```
### **RECOVERED PLAINTEXT M = 88** - matches. Verified.

*(This is the classic Stallings textbook example - if you remember only one RSA numerical, remember this one: p=17, q=11, e=7, d=23, M=88 -> C=11.)*

---

## §4.4 Q24. NUMERICAL: p = 13, q = 17, public key e = 35 - find the private key

```
STEP 1 :  n = p x q = 13 x 17 = 221

STEP 2 :  phi(n) = (p-1)(q-1) = 12 x 16 = 192

STEP 3 :  Verify e = 35 is a valid public exponent:
              gcd(35, 192) :
              192 = 5 x 35 + 17
               35 = 2 x 17 +  1
               17 = 17 x 1 + 0    ->  gcd = 1   VALID

STEP 4 :  Find d = 35^-1 mod 192 using Extended Euclid.

          Back-substitution:
              1 = 35 - 2 x 17
              17 = 192 - 5 x 35
          =>  1 = 35 - 2 x (192 - 5 x 35)
              1 = 35 - 2 x 192 + 10 x 35
              1 = 11 x 35 - 2 x 192

          Reducing mod 192 :   11 x 35 ≡ 1 (mod 192)
```

### **ANSWER: Private key d = 11, i.e. PR = { 11, 221 }**  (Public key PU = { 35, 221 })

**Verification:** 35 x 11 = 385 = 192 x 2 + 1 = 384 + 1 ≡ 1 (mod 192). Correct.

---

## §4.5 Q30. NUMERICAL: p = 13, q = 17, e = 7 - find d and encrypt "CRYPTOGRAPHY" (A=00 to Z=25)

### PART (a) - Key generation

```
n      = 13 x 17 = 221
phi(n) = 12 x 16 = 192
e      = 7 ;  gcd(7, 192) = 1  (192 = 27 x 7 + 3 ; 7 = 2 x 3 + 1 ; gcd = 1)  VALID

d = 7^-1 mod 192 :
    192 = 27 x 7 + 3
      7 =  2 x 3 + 1
    Back-substitute:
      1 = 7 - 2 x 3
      3 = 192 - 27 x 7
  =>  1 = 7 - 2 x (192 - 27 x 7) = 7 - 2 x 192 + 54 x 7 = 55 x 7 - 2 x 192
  =>  55 x 7 ≡ 1 (mod 192)
```

### **PUBLIC KEY PU = { 7, 221 }   ;   PRIVATE KEY PR = { 55, 221 }**
**(Decryption key d = 55.)** Check: 7 x 55 = 385 = 384 + 1 ≡ 1 (mod 192). Correct.

### PART (b) - Encode the message

Using A = 00, B = 01, ..., Z = 25:

| Letter | C | R | Y | P | T | O | G | R | A | P | H | Y |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **M** | 02 | 17 | 24 | 15 | 19 | 14 | 06 | 17 | 00 | 15 | 07 | 24 |

### PART (c) - Encrypt: C = M^7 mod 221

Compute `M^7 = M^4 x M^2 x M` for each distinct value.

**M = 02 (C):**
```
2^2 = 4 ;  2^4 = 16
2^7 = 16 x 4 x 2 = 128           ->  C = 128
```

**M = 17 (R):**
```
17^2 = 289 = 221 + 68            -> 68
17^4 = 68^2 = 4624 = 221 x 20 + 204   -> 204
17^7 = 204 x 68 x 17 :
       204 x 68 = 13872 = 221 x 62 + 170   -> 170
       170 x 17 = 2890  = 221 x 13 + 17    -> 17
                                  ->  C = 17
```

**M = 24 (Y):**
```
24^2 = 576  = 221 x 2 + 134           -> 134
24^4 = 134^2 = 17956 = 221 x 81 + 55  -> 55
24^7 = 55 x 134 x 24 :
       55 x 134 = 7370 = 221 x 33 + 77 -> 77
       77 x 24  = 1848 = 221 x 8 + 80  -> 80
                                  ->  C = 80
```

**M = 15 (P):**
```
15^2 = 225 = 221 + 4                  -> 4
15^4 = 4^2 = 16                       -> 16
15^7 = 16 x 4 x 15 = 960 = 221 x 4 + 76  ->  C = 76
```

**M = 19 (T):**
```
19^2 = 361 = 221 + 140                -> 140
19^4 = 140^2 = 19600 = 221 x 88 + 152 -> 152
19^7 = 152 x 140 x 19 :
       152 x 140 = 21280 = 221 x 96 + 64  -> 64
        64 x 19  = 1216  = 221 x 5 + 111  -> 111
                                  ->  C = 111
```

**M = 14 (O):**
```
14^2 = 196                            -> 196
14^4 = 196^2 = 38416 = 221 x 173 + 183 -> 183
14^7 = 183 x 196 x 14 :
       183 x 196 = 35868 = 221 x 162 + 66  -> 66
        66 x 14  = 924   = 221 x 4 + 40    -> 40
                                  ->  C = 40
```

**M = 06 (G):**
```
6^2 = 36                              -> 36
6^4 = 1296 = 221 x 5 + 191            -> 191
6^7 = 191 x 36 x 6 :
      191 x 36 = 6876 = 221 x 31 + 25 -> 25
       25 x 6  = 150                  -> 150
                                  ->  C = 150
```

**M = 00 (A):**
```
0^7 = 0                           ->  C = 000
```

**M = 07 (H):**
```
7^2 = 49                              -> 49
7^4 = 2401 = 221 x 10 + 191           -> 191
7^7 = 191 x 49 x 7 :
      191 x 49 = 9359 = 221 x 42 + 77 -> 77
       77 x 7  = 539  = 221 x 2 + 97  -> 97
                                  ->  C = 97
```

### **FINAL CIPHERTEXT TABLE**

| Letter | C | R | Y | P | T | O | G | R | A | P | H | Y |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **M** | 02 | 17 | 24 | 15 | 19 | 14 | 06 | 17 | 00 | 15 | 07 | 24 |
| **C = M^7 mod 221** | **128** | **17** | **80** | **76** | **111** | **40** | **150** | **17** | **000** | **76** | **97** | **80** |

**CIPHERTEXT = 128 017 080 076 111 040 150 017 000 076 097 080**

### PART (d) - Decryption check: M = C^55 mod 221

Verify the first block C = 128 (should give M = 02). Write 55 = 32 + 16 + 4 + 2 + 1.

| Power | Reduction mod 221 | Result |
|---|---|---|
| 128^1 | - | 128 |
| 128^2 | 16384 = 221 x 74 + 30 | 30 |
| 128^4 | 30^2 = 900 = 221 x 4 + 16 | 16 |
| 128^8 | 16^2 = 256 = 221 + 35 | 35 |
| 128^16 | 35^2 = 1225 = 221 x 5 + 120 | 120 |
| 128^32 | 120^2 = 14400 = 221 x 65 + 35 | 35 |

```
128^55 = 128^32 x 128^16 x 128^4 x 128^2 x 128^1
       ≡ 35 x 120 x 16 x 30 x 128   (mod 221)

35 x 120 = 4200 = 221 x 19 + 1     -> 1
 1 x 16  = 16                      -> 16
16 x 30  = 480 = 221 x 2 + 38      -> 38
38 x 128 = 4864 = 221 x 22 + 2     -> 2
```
### **M = 02 = 'C'** - decryption verified. The scheme works correctly.

---

## §4.6 Q5. Security considerations of RSA *(3-6 marks)*

**Possible attacks on RSA and their countermeasures:**

```
                    ATTACKS ON RSA
                          |
    +-------------+-------+--------+--------------+
    |             |                |              |
BRUTE FORCE  MATHEMATICAL      TIMING      CHOSEN CIPHERTEXT
(try all d)   ATTACKS          ATTACK         ATTACK (CCA)
                  |
        +---------+---------+---------+
        |         |         |         |
    Factor n   Determine  Determine  Common
   into p, q   phi(n)     d directly modulus
                          without
                          phi(n)
```

| Attack | How it works | Countermeasure |
|---|---|---|
| **Brute force** | Try all possible private keys d | Use a **large key size** (n >= 2048 bits) |
| **Mathematical (factoring)** | Factor n = p x q, then compute phi(n) and d | Large n; **p and q of similar but not too close magnitude**; (p-1) and (q-1) should have a large prime factor; gcd(p-1, q-1) should be small |
| **Timing attack** | Deduce d by measuring how long decryptions take | **Constant-time exponentiation**, random delay, **blinding** (multiply ciphertext by a random value before exponentiating) |
| **Chosen-ciphertext attack (CCA)** | Exploit the multiplicative property `E(M1)·E(M2) = E(M1·M2)` | Use **OAEP padding (Optimal Asymmetric Encryption Padding)** before encryption |
| **Low exponent attack** | If e is small (e.g. 3) and M is small, M^e < n so no wrap-around occurs and the cube root reveals M | Use **e = 65537**, and always pad the message |
| **Common modulus attack** | Same n used with different e for different users | Never share the modulus n across users |

**Other practical points to write:**
- Plaintext **M must satisfy 0 <= M < n**; longer messages are broken into blocks.
- The primes p and q should be **at least 1024 bits each** so that n is 2048 bits.
- **p and q must be discarded** (or protected) after key generation - anyone who learns them can compute d.
- RSA is **slow** (~1000x slower than AES), so in practice it is used only to encrypt a **symmetric session key**, which then encrypts the bulk data. This is the **hybrid cryptosystem** model.
- The **RSA problem** (recovering M from C = M^e mod n) is believed to be as hard as **integer factorisation**, but this has never been proved.

---

# §5. ALL DIFFERENCE TABLES - QUICK REFERENCE

*(All the "difference between" answers collected in one place for last-minute revision.)*

## 5.1 Symmetric vs Asymmetric Cryptography

| Basis | Symmetric | Asymmetric |
|---|---|---|
| Keys used | One shared secret key | Key pair (public + private) |
| Speed | Fast | Slow (1000x slower) |
| Key distribution | Difficult (needs secure channel) | Easy (public key is published) |
| Keys for n users | n(n-1)/2 | 2n |
| Key length | 56-256 bits | 1024-4096 bits |
| Services | Confidentiality | Confidentiality, Authentication, Non-repudiation |
| Examples | DES, AES, Blowfish, RC4 | RSA, DH, ECC, ElGamal |

## 5.2 Substitution vs Transposition
*(see §2.1)*

## 5.3 Stream vs Block cipher
*(see §1.5)*

## 5.4 Active vs Passive attacks
*(see §1.4)*

## 5.5 Cryptography vs Steganography
*(see §1.7)*

## 5.6 Confusion vs Diffusion (Shannon's principles - bonus)

| Basis | **Confusion** | **Diffusion** |
|---|---|---|
| **Purpose** | Hide the relationship between the **ciphertext and the KEY** | Hide the relationship between the **ciphertext and the PLAINTEXT** |
| **Achieved by** | **Substitution** (S-boxes) | **Permutation / Transposition** (P-boxes) |
| **Effect** | Each ciphertext bit depends on many key bits | Changing one plaintext bit changes many ciphertext bits (avalanche effect) |
| **Used in** | Both stream and block ciphers | Only block ciphers |

## 5.7 Fermat's vs Euler's Theorem

| Basis | **Fermat's Little Theorem** | **Euler's Theorem** |
|---|---|---|
| **Statement** | a^(p-1) ≡ 1 (mod p) | a^ϕ(n) ≡ 1 (mod n) |
| **Condition on modulus** | p must be **prime** | n can be **any** positive integer |
| **Condition on a** | p does not divide a | gcd(a, n) = 1 |
| **Relation** | Special case of Euler's theorem when n = p, since ϕ(p) = p - 1 | The general form |
| **Use** | Fermat primality test, exponent reduction | **Proof of RSA correctness**, exponent reduction |

## 5.8 Block Cipher Modes (one-liners, in case asked)

| Mode | Full form | Key idea |
|---|---|---|
| **ECB** | Electronic Codebook | Each block encrypted independently; identical blocks -> identical ciphertext (insecure for long messages) |
| **CBC** | Cipher Block Chaining | Each plaintext block XORed with previous ciphertext block; needs an IV |
| **CFB** | Cipher Feedback | Converts block cipher into a self-synchronising stream cipher |
| **OFB** | Output Feedback | Stream cipher mode; keystream independent of plaintext |
| **CTR** | Counter | Encrypts an incrementing counter; fully parallelisable |

---

# §6. NUMERICAL ANSWER-KEY DRILL

*Cover the right column and test yourself. If you get all of these in 15 minutes, you are ready.*

| # | Question | **Answer** |
|---|---|---|
| Q7/Q21 | gcd(1970, 1066) | **2** |
| Q14 | gcd(24120, 1640) | **40** |
| Q8 | ϕ(55) | **40** |
| Q20 | ϕ(35) | **24** |
| Q25 | ϕ(12) | **4** |
| Q12 | 11^-1 mod 26 | **19** |
| Q34 | 550^-1 in GF(1759) | **355** |
| Q13 | Miller-Rabin on n = 37, a = 2 | **Inconclusive -> probably prime** (n-1 = 2^2 x 9; 2^9 ≡ 31; 2^18 ≡ 36 = n-1) |
| Q15 | Primitive roots of 13 | **{2, 6, 7, 11}** |
| Q18 | CRT: x≡2(3), x≡3(5) | **x = 8 (mod 15)** |
| Q22/Q29 | CRT: x≡2(3), x≡3(5), x≡2(7) | **x = 23 (mod 105)** |
| Q27 | CRT: x≡1(5), x≡2(7), x≡3(9), x≡4(11) | **x = 1731 (mod 3465)** |
| Q19 | a ≡ 9^794 (mod 73) | **a = 8** |
| Q26 | 3^61 mod 7 | **3** (literal 361 mod 7 = 4) |
| Q19b | Caesar decrypt "PHHW PH", k=3 | **"MEET ME"** |
| Q11 | Vigenere "LIFEISFULLOFSURPRISES" key HEALTH | **SMFPBZMYLWHMZYRAKPZIS** |
| Q23 | Playfair "hidethegoldinthetreestump", key playfair | **EBIMQMGHVRIRONKGODKUKNNZEF** |
| Q17 | RSA p=17,q=11,e=7,d=23; M=77 | n=187, ϕ=160, PU={7,187}, PR={23,187}, **C = 121** |
| Q33 | RSA same keys; M=88 | **C = 11**, decrypt back to 88 |
| Q24 | RSA p=13,q=17, e=35; find d | n=221, ϕ=192, **d = 11** |
| Q30 | RSA p=13,q=17,e=7; find d, encrypt CRYPTOGRAPHY | **d = 55**; C = **128 017 080 076 111 040 150 017 000 076 097 080** |

---

# §7. ONE-PAGE FORMULA SHEET

```
+=============================================================================+
|  MODULAR ARITHMETIC                                                         |
|    (a + b) mod n = [(a mod n) + (b mod n)] mod n                            |
|    (a x b) mod n = [(a mod n) x (b mod n)] mod n                            |
|    a^k   mod n   = [(a mod n)^k] mod n                                      |
+=============================================================================+
|  EUCLID'S ALGORITHM                                                         |
|    gcd(a, b) = gcd(b, a mod b)      -> last non-zero remainder is the gcd   |
+=============================================================================+
|  EXTENDED EUCLID                                                            |
|    a·x + b·y = gcd(a, b)                                                    |
|    If gcd(a, n) = 1  then  a^-1 mod n = x mod n   (add n if negative)       |
|    INVERSE EXISTS  <=>  gcd(a, n) = 1                                       |
+=============================================================================+
|  EULER'S TOTIENT                                                            |
|    phi(p)    = p - 1                        (p prime)                       |
|    phi(p^k)  = p^k - p^(k-1)                                                |
|    phi(pq)   = (p-1)(q-1)                   (p, q DISTINCT primes)          |
|    phi(n)    = n (1 - 1/p1)(1 - 1/p2)...    (general)                       |
|    phi(mn)   = phi(m) x phi(n)              if gcd(m, n) = 1                |
+=============================================================================+
|  FERMAT'S LITTLE THEOREM  (p prime, gcd(a,p)=1)                             |
|    a^(p-1) ≡ 1 (mod p)          and       a^p ≡ a (mod p)                   |
+=============================================================================+
|  EULER'S THEOREM  (gcd(a, n) = 1)                                           |
|    a^phi(n) ≡ 1 (mod n)                                                     |
|    Exponent reduction:  a^b mod n = a^(b mod phi(n)) mod n                  |
+=============================================================================+
|  CHINESE REMAINDER THEOREM                                                  |
|    M = m1 x m2 x ... x mk    (mi pairwise coprime)                          |
|    Mi = M / mi ,   yi = Mi^-1 mod mi                                        |
|    x = SUM( ai x Mi x yi )  mod M                                           |
+=============================================================================+
|  PRIMITIVE ROOT                                                             |
|    a is a primitive root of n  <=>  order of a mod n = phi(n)               |
|    Number of primitive roots = phi(phi(n))                                  |
|    All roots = g^k mod n  where gcd(k, phi(n)) = 1                          |
+=============================================================================+
|  MILLER-RABIN                                                               |
|    n - 1 = 2^k x q  (q odd)                                                 |
|    if a^q ≡ 1  OR  a^(2^j · q) ≡ n-1 for some 0<=j<=k-1  -> inconclusive    |
|    else -> COMPOSITE                                                        |
+=============================================================================+
|  RSA                                                                        |
|    n = p x q ;  phi(n) = (p-1)(q-1)                                         |
|    gcd(e, phi(n)) = 1 ,  1 < e < phi(n)                                     |
|    d ≡ e^-1 (mod phi(n))     i.e.   e x d ≡ 1 (mod phi(n))                  |
|    PU = {e, n}      PR = {d, n}                                             |
|    Encrypt : C = M^e mod n            Decrypt : M = C^d mod n               |
+=============================================================================+
|  CLASSICAL CIPHERS                                                          |
|    Caesar    : C = (p + k) mod 26  ;  p = (C - k) mod 26                    |
|    Vigenere  : C[i] = (P[i] + K[i mod m]) mod 26                            |
|    Hill      : C = KP mod 26  ;  P = K^-1 C mod 26                          |
|    Playfair  : 5x5 matrix, digrams, same-row->RIGHT, same-col->DOWN,        |
|                rectangle->swap columns                                      |
|    Letter map: A=0 B=1 C=2 D=3 E=4 F=5 G=6 H=7 I=8 J=9 K=10 L=11 M=12       |
|                N=13 O=14 P=15 Q=16 R=17 S=18 T=19 U=20 V=21 W=22 X=23       |
|                Y=24 Z=25                                                    |
+=============================================================================+
```

## Square-and-multiply: how to compute a^b mod n fast (the ONE skill you need)

```
Example: 88^7 mod 187

1. Write the exponent as a sum of powers of 2:   7 = 4 + 2 + 1
2. Build up by repeated squaring, reducing mod n at EVERY step:
       88^1 = 88
       88^2 = 88 x 88 mod 187 = 77
       88^4 = 77 x 77 mod 187 = 132
3. Multiply the needed pieces together, reducing at each step:
       88^7 = 132 x 77 x 88 mod 187 = 11
```

**NEVER compute 88^7 as a full number.** Reduce after every single multiplication.

---

# §8. EXAMINER TIPS & COMMON MISTAKES

## 8.1 Presentation checklist

- [ ] Start each answer on a **new page or with a clear heading** and write the question number.
- [ ] Every algorithm question -> **draw the flowchart/block diagram** (worth 1-2 marks on its own).
- [ ] Every "difference" question -> **table with a "Basis of comparison" column**, minimum 5 rows.
- [ ] Every numerical -> **state the formula first**, then substitute, then compute step by step.
- [ ] **Box or underline** every final answer.
- [ ] Add a **verification step** at the end of numericals - examiners love it and it catches your own mistakes.
- [ ] For a 6-mark question, aim for: definition (1) + algorithm/steps (2) + diagram (1) + worked example (2).

## 8.2 Top 10 mistakes to avoid

| # | Mistake | Correct approach |
|---|---|---|
| 1 | Using ϕ(n) = (p-1)(q-1) for n = 12 or n = 100 | That formula is only for **two DISTINCT primes**. Use the general product formula otherwise. |
| 2 | Computing 88^7 as a huge number | Use **square-and-multiply**, reduce mod n at every step |
| 3 | Forgetting to add n to a **negative** inverse | -7 mod 26 = **19**, not -7 |
| 4 | In Playfair, forgetting to **merge I/J** or insert X between doubled letters | Both rules are worth marks |
| 5 | In Playfair "same row", moving LEFT instead of RIGHT | Encryption: RIGHT and DOWN. Decryption: LEFT and UP. |
| 6 | In Vigenere, misaligning the repeated key | Write the key under the plaintext letter by letter before starting |
| 7 | Forgetting the CRT precondition | Always write "moduli are pairwise relatively prime, so CRT applies" |
| 8 | Writing d = e^-1 mod n | It is **d = e^-1 mod ϕ(n)**, NOT mod n |
| 9 | Applying Fermat when p is not prime | Check primality first; otherwise use Euler's theorem |
| 10 | Answering "difference between" in paragraphs | Always a table |

## 8.3 30-second definitions to have on the tip of your tongue

| Term | One-line definition |
|---|---|
| **Plaintext** | The original intelligible message fed into the encryption algorithm |
| **Ciphertext** | The scrambled, unintelligible output of the encryption algorithm |
| **Cipher** | An algorithm for performing encryption and decryption |
| **Key** | Information used by the cipher, known only to the sender/receiver |
| **Cryptanalysis** | Techniques for deciphering a message without knowledge of the key |
| **Brute-force attack** | Trying every possible key until an intelligible plaintext is obtained |
| **Avalanche effect** | A small change in plaintext or key causes a large change in ciphertext |
| **Unconditionally secure** | Cipher cannot be broken no matter how much ciphertext or computing power is available (only the One-Time Pad) |
| **Computationally secure** | Cost of breaking exceeds the value of the information, or time to break exceeds the useful lifetime of the information |
| **Trapdoor one-way function** | A function easy to compute in one direction, infeasible to invert unless you know special secret information (the trapdoor) - the basis of public-key cryptography |
| **Digital signature** | Data appended to a message, produced with the sender's private key, that proves origin and integrity |

## 8.4 Cryptanalytic attack types (1-marker material)

| Attack | What the cryptanalyst knows |
|---|---|
| **Ciphertext-only** | Encryption algorithm + ciphertext (hardest attack) |
| **Known-plaintext** | Above + one or more plaintext-ciphertext pairs |
| **Chosen-plaintext** | Above + can choose plaintexts and obtain their ciphertexts |
| **Chosen-ciphertext** | Above + can choose ciphertexts and obtain their plaintexts |
| **Chosen-text** | Both chosen-plaintext and chosen-ciphertext capability (easiest for attacker) |

---

# FINAL 10-MINUTE REVISION LIST (read this on the way to the hall)

1. `gcd(a,b) = gcd(b, a mod b)` - last non-zero remainder.
2. `ϕ(pq) = (p-1)(q-1)` only for **distinct primes**; `ϕ(p^k) = p^k - p^(k-1)`.
3. Fermat: `a^(p-1) ≡ 1 (mod p)`. Euler: `a^ϕ(n) ≡ 1 (mod n)`.
4. Reduce big exponents: `a^b mod n = a^(b mod ϕ(n)) mod n`.
5. RSA: `n = pq`, `ϕ = (p-1)(q-1)`, `gcd(e,ϕ)=1`, `d = e^-1 mod ϕ`, `C = M^e mod n`, `M = C^d mod n`.
6. Remember the classic: **p=17, q=11, e=7, d=23, n=187, ϕ=160, M=88 -> C=11.**
7. CRT: `M = product`, `Mi = M/mi`, `yi = Mi^-1 mod mi`, `x = Σ ai·Mi·yi mod M`.
8. Miller-Rabin: `n-1 = 2^k·q`; check `a^q ≡ 1` or `a^(2^j·q) ≡ n-1`.
9. Primitive roots of 13 = {2, 6, 7, 11}; count = ϕ(ϕ(n)).
10. Playfair: same row -> RIGHT, same column -> DOWN, rectangle -> swap columns. I/J merged, X for doubles.
11. Caesar `C = (p+3) mod 26`; "PHHW PH" = "MEET ME".
12. Passive = eavesdrop (hard to detect, easy to prevent). Active = modify (easy to detect, hard to prevent).
13. Cryptography hides the meaning; steganography hides the existence.
14. Substitution changes identity; transposition changes position.

**All the best for your exam. Answer the 1-markers first, then the 6-marker while you are fresh, then the 3-markers.**
