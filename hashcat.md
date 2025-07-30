# Hashcat Cheatsheet

Hashcat is the world's fastest CPU-based password cracker, and also supports GPU, FPGA, DSP, and other accelerators. It's highly versatile, supporting a vast array of hash types and attack modes.

## Basic Usage

* **Syntax:** `hashcat [options] <hash(es)> [dictionary|mask|session]`

* **Crack a hash file with a wordlist (Attack Mode 0 - Straight):**
    ```bash
    hashcat -a 0 -m <hash_type> hashes.txt wordlist.txt
    ```
    * Example (NTLM): `hashcat -a 0 -m 1000 ntlm_hashes.txt rockyou.txt`

* **Specify a hash type (`-m`):**
    * **To list all supported hash types and their IDs:**
        ```bash
        hashcat --help | grep "Hash modes" -A 999
        ```
    * Common Hash Types:
        * `0`: MD5
        * `100`: SHA1
        * `1000`: NTLM
        * `1100`: NetNTLMv1-SSP/ESS
        * `500`: MD5crypt, scrypt (used by Linux `crypt`)
        * `1700`: SHA512crypt (used by Linux `crypt`)
        * `2500`: WPA/WPA2 (PMKID/EAPOL)
        * `1800`: SHA-512
        * `900`: MD4
        * `1400`: SHA256
        * `10000`: WPA-PMKID-PBKDF2
        * `11300`: MySQL 5.x/SHA1
        * `1600`: MS-SQL (2000)
        * `3000`: NTLMv2 (NetNTLMv2)
        * `13100`: Kerberos 5 AS-REP (Pre-Auth EType 23)

## Attack Modes (`-a`)

* `-a 0`: **Straight (Dictionary) Attack:** Combines a wordlist with optional rules.
    ```bash
    hashcat -a 0 -m 1000 hashes.txt wordlist.txt -r rules/best64.rule
    ```
* `-a 1`: **Combinator Attack:** Combines two wordlists. `wordlist1.txt` `wordlist2.txt`.
    * `word1` from list1 + `word2` from list2.
    ```bash
    hashcat -a 1 -m 0 hashes.txt names.txt years.txt
    ```
* `-a 3`: **Brute-force Attack (Mask Attack):** Generates passwords based on a character mask.
    * `?l`: lowercase letters `a-z`
    * `?u`: uppercase letters `A-Z`
    * `?d`: digits `0-9`
    * `?s`: special characters `!@#$%^&*()_+-=[]{};':"|,.<>/?`
    * `?a`: all printable ASCII (`?l?u?d?s`)
    * `?b`: all possible bytes `0x00 - 0xff` (256 characters)
    * **Custom charsets:**
        * `-1 <chars>`: define `?1`
        * `-2 <chars>`: define `?2`
        * `-3 <chars>`: define `?3`
        * `-4 <chars>`: define `?4`
    * Example (8 lowercase alpha chars):
        ```bash
        hashcat -a 3 -m 1000 hashes.txt ?l?l?l?l?l?l?l?l
        ```
    * Example (password starting with `pass` followed by 4 digits):
        ```bash
        hashcat -a 3 -m 0 hashes.txt pass?d?d?d?d
        ```
    * Example (Custom charset `?1` for `abc`, 3 chars long):
        ```bash
        hashcat -a 3 -m 0 hashes.txt -1 abc ?1?1?1
        ```
* `-a 6`: **Hybrid Attack (Wordlist + Mask):** Wordlist + append mask.
    ```bash
    hashcat -a 6 -m 1000 hashes.txt wordlist.txt ?d?d?d
    ```
* `-a 7`: **Hybrid Attack (Mask + Wordlist):** Prepend mask + wordlist.
    ```bash
    hashcat -a 7 -m 1000 hashes.txt ?d?d?d wordlist.txt
    ```

## Common Options

* `-o <file>`: Write cracked passwords to `file` (default: `hashcat.potfile`).
* `--force`: Force hashcat to run even if warnings are present (e.g., old drivers).
* `--show`: Show cracked hashes from the potfile.
* `--remove`: Remove cracked hashes from the hash file.
* `--session <name>`: Start/restore a named session.
* `--restore`: Resume the last session.
* `--status`: Show live cracking status.
* `--benchmark`: Run a benchmark on your hardware for a given hash type.
* `-O`: Optimized kernel (reduces max password length to 32 but gives speedup).
* `-w <num>`: Workload profile (1-4, default 2). 3 is aggressive, 4 is "insane".
    * `1`: Low (less aggressive GPU usage)
    * `2`: Default
    * `3`: High (more aggressive GPU usage)
    * `4`: Headache (very aggressive GPU usage, can cause system instability)
* `-D <num>`: Device type to use (`1=CPU`, `2=GPU`, `3=FPGA/DSP`).
* `-d <num>`: Specify a specific device ID (e.g., `-d 1` for second GPU).
* `--potfile-disable`: Do not use or write to the potfile.
* `--skip <num>`: Skip `num` candidates from the beginning of the attack.
* `--limit <num>`: Only process `num` candidates.
* `--truecrypt-keyfiles <file(s)>`: Specify keyfiles for TrueCrypt/VeraCrypt.
* `--loopback`: Appends recovered passwords to the working dictionary so they can be used to crack other passwords (useful for chained attacks).

## Rules (`-r`)

Hashcat rules are extremely powerful for mangling words from a dictionary. They are similar in concept to John the Ripper rules but use a slightly different syntax.

### How Rules Work

Each rule in a rule file is a single line, specifying a sequence of transformations to apply to an input word. Hashcat takes each word from your wordlist and applies every rule in the specified rule file, generating new candidate passwords.

### Common Rule Commands:

Hashcat rules consist of single-character commands followed by optional arguments.

* **`:` (No-op):** Does nothing. Useful for comments or as a placeholder.
* `c`: Capitalize the first letter. `password` -> `Password`
* `C`: Uncapitalize the first letter. `Password` -> `password`
* `l`: Lowercase the entire word. `PASSWORD` -> `password`
* `u`: Uppercase the entire word. `password` -> `PASSWORD`
* `t`: Toggle case of the entire word. `Password` -> `pASSWORD`
* `T<N>`: Toggle case of the character at position `N` (0-indexed). `T0` on `Password` -> `password`
* `r`: Reverse the word. `password` -> `drowssap`
* `d`: Duplicate the word. `password` -> `passwordpassword`
* `f`: Reflect the word (append reversed word). `pass` -> `passssap`
* `p<N>`: Duplicate the N-th character. `p0` on `pass` -> `ppass`
* `p<N><X>`: Duplicate the N-th character `X` times. `p02` on `pass` -> `ppass`
* `$<X>`: Append character `X`. `$1` -> `password1`
* `^<X>`: Prepend character `X`. `^A` -> `Apassword`
* `D<N>`: Delete character at position `N`. `D0` on `password` -> `assword`
* `D$`: Delete the last character. `D$` on `password` -> `passwor`
* `x<N><M>`: Extract substring from position `N` for `M` characters. `x03` on `password` -> `pas`
* `O<N><X>`: Overstrike (replace) character at position `N` with `X`. `O0P` on `password` -> `Password`
* `i<N><X>`: Insert character `X` at position `N`. `i0!` -> `!password`
* `s<X><Y>`: Substitute all occurrences of `X` with `Y`. `sa4` -> `p4ssword`
* `@<X>`: Purge (delete) all occurrences of `X`. `@a` on `banana` -> `bnn`
* `!<X>`: Reject if word contains `X`. (This is a reject rule, typically placed at the beginning of a line)
* `/<X>`: Reject unless word contains `X`. (This is a reject rule)
* `><N>`: Reject if word length is greater than `N`.
* `<N>`: Reject if word length is less than `N`.
* `'<N>`: Truncate word to length `N`. `'8` on `longerword` -> `longerwo`

### Character Classes (used with some commands):

* `?l`: lowercase letters `a-z`
* `?u`: uppercase letters `A-Z`
* `?d`: digits `0-9`
* `?s`: special characters `!@#$%^&*()_+-=[]{};':"|,.<>/?`
* `?a`: all printable ASCII (`?l?u?d?s`)
* `?b`: all possible bytes `0x00 - 0xff` (256 characters)

### Example Rule File (`my_rules.rule`)

You would save this content to a file, e.g., `my_rules.rule`.

~~~
This is a comment line
Each line is a rule. Commands are applied left to right.
Common variations
c                       # Capitalize first letter (e.g., "password" -> "Password")
$1                      # Append '1' (e.g., "Password" -> "Password1")
$!                      # Append '!' (e.g., "Password" -> "Password!")
r                       # Reverse word (e.g., "password" -> "drowssap")
d                       # Duplicate word (e.g., "password" -> "passwordpassword")
Numeric appends (using built-in digit character class)
$0
$1
$2
$3
$4
$5
$6
$7
$8
$9
Common year appends
$19
$20
$21
$22
$23
$24
$25
Simple Leetspeak substitutions
sa4                     # a -> 4
se3                     # e -> 3
si1                     # i -> 1
so0                     # o -> 0
st7                     # t -> 7
sS5                     # S -> 5
sO0                     # O -> 0
Combination rules
c$1                     # Capitalize, then append '1'
c$!                     # Capitalize, then append '!'
c$@                     # Capitalize, then append '@'
c$0                     # Capitalize, then append '0'
c$123                   # Capitalize, then append "123"
Rule to swap first char and append digit
x1$D0                   # Extract from index 1 to end, then delete first character (effectively rotate left)
s01                     # Replace '0' with '1'
Reject rules example
!@                      # Reject if word contains '@'

>8                      # Reject if word is longer than 8 characters
<6                      # Reject if word is shorter than 6 characters
~~~

### How to use the rule file:

```bash
hashcat -a 0 -m 1000 hashes.txt wordlist.txt -r my_rules.rule
```

Hashcat comes with many excellent pre-built rule files in its rules/ directory (e.g., best64.rule, rockyou-30000.rule, dive.rule). It's often beneficial to start with these and then create your own custom rules for specific targets.

### Hashcat-Utils
A suite of tools often used with Hashcat:

- `cap2hccapx`: Converts `.pcap` (Wi-Fi capture) files to `.hccapx` format for WPA/WPA2 cracking.

- `hccapx2john`: Converts `.hccapx` to a format usable by John the Ripper (if needed).

- `keyspace`: Calculates the keyspace of a given mask, useful for estimating brute-force time.