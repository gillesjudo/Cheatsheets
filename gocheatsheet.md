# John the Ripper Cheatsheet

John the Ripper (JtR) is a fast password cracker, available for many flavors of Unix, Windows, DOS, BeOS, and OpenVMS. Its primary purpose is to detect weak Unix passwords. Besides several crypt(3) password hash types most commonly found on various Unix flavors, supported hash types include Kerberos AFS and Windows LM/NTLM.

## Basic Usage

* **Crack passwords from a shadow file:**
    ```bash
    john /etc/shadow
    ```

* **Specify a wordlist:**
    ```bash
    john --wordlist=./wordlist.txt hashes.txt
    ```

* **Specify a rule file (using a custom rule set named 'MyRules'):**
    ```bash
    john --rules=MyRules hashes.txt
    ```

* **Show cracked passwords:**
    ```bash
    john --show hashes.txt
    ```

* **Show uncracked passwords (useful for further cracking):**
    ```bash
    john --remaining hashes.txt
    ```

* **Remove cracked passwords from the pot file:**
    ```bash
    john --keep-all-cand hashes.txt # Use with caution, can remove non-cracked too
    ```

* **Single crack mode (fastest for simple rules):**
    ```bash
    john --single hashes.txt
    ```

* **Incremental mode (brute-force based on character sets):**
    ```bash
    john --incremental hashes.txt
    ```
    * To specify a mode (e.g., `alpha`, `digits`, `all`):
        ```bash
        john --incremental:all hashes.txt
        ```

## Common Options

* `--format=<type>`: Specify hash format (e.g., `raw-md5`, `phpass`, `nt`). Crucial for non-standard hashes.
    * **To list all supported formats:**
        ```bash
        john --list=formats
        ```
* `--stdout`: Print candidate passwords to stdout without cracking. Useful for generating wordlists.
    ```bash
    john --wordlist=mywordlist.txt --rules --stdout > generated_list.txt
    ```
* `--session=<name>`: Save/restore a session.
    ```bash
    john --session=mysession hashes.txt
    ```
* `--fork=<N>`: Use N CPU cores/threads (OpenMP build only).
    ```bash
    john --fork=4 hashes.txt
    ```
* `--pot=./john.pot`: Specify an alternative pot file location.
* `--max-length=<N>`: Maximum password length for incremental/single mode.
* `--min-length=<N>`: Minimum password length for incremental/single mode.
* `--input-file=<file>`: Specify an input file (alternative to positional argument).
* `--wordlist=<file>` / `-w`: Path to a wordlist.
* `--rules[=<mode>]` / `-r`: Enable wordlist rules. Can be specified with a mode (e.g., `wordlist`, `john.conf`).
* `--restore[=<session>]`: Restore a saved session.

## Hash Formats (Examples)

* **MD5:** `john --format=raw-md5 hashes.txt`
* **SHA1:** `john --format=raw-sha1 hashes.txt`
* **NTLM:** `john --format=nt hashes.txt`
* **LM:** `john --format=lm hashes.txt`
* **MySQL:** `john --format=mysql hashes.txt`
* **Phpass:** `john --format=phpass hashes.txt`
* **DES (Unix crypt):** `john hashes.txt` (often auto-detected)
* **MD5crypt (Linux/BSD):** `john hashes.txt` (often auto-detected)

## Cracking Strategies

1.  **Wordlist Attack:** The most common and effective.
    * Use common wordlists (e.g., rockyou.txt).
    * Use target-specific wordlists (e.g., company names, project names).
2.  **Rules-based Attack:** Apply transformations (e.g., capitalization, numbers, special characters) to wordlist entries.
    * John's `john.conf` contains powerful default rules.
    * Create custom rules for specific patterns.
3.  **Single Crack Mode:** Attempts to crack passwords based on login/GECOS information from the password file. Very fast for simple passwords.
4.  **Incremental Mode (Brute-force):** Tries all possible character combinations. This is very slow and should be a last resort or used with specific character sets and lengths.
5.  **External Mode:** Allows for custom cracking functions written in C.

## John the Ripper Files

* `john.conf`: Main configuration file, contains rules, character sets, and more.
* `john.pot`: Pot file, stores cracked passwords and their corresponding hashes.
* `john.rec`: Session resume file.

## Creating Hash Files

* For simple hashes (e.g., raw-md5, raw-sha1):
    ```
    echo "password_hash" > hashes.txt
    ```
* For /etc/shadow format: Copy relevant lines from `/etc/shadow`.

## Tips and Tricks

* **Prioritize:** Start with wordlist attacks, then rules, then incremental (if necessary).
* **Custom Rules:** Learn to write custom rules in `john.conf` for highly targeted attacks.
* **Combine Tools:** Use `hashcat` for GPU cracking or more advanced attack modes. Use `crunch` to generate custom wordlists.
* **Monitor Progress:** John shows cracking progress in real-time.
* **`--loopback`:** Feed cracked passwords back into the cracking process as potential candidates for other hashes (advanced).
* **Optimizing Performance:**
    * Use `--fork` for multi-core CPUs.
    * Ensure your John build is optimized for your CPU (e.g., using `--enable-native-cpu-optimizations` during compilation).
    * Use optimized hash formats.
* **Don't forget to check the `doc/` directory in the John the Ripper distribution for more in-depth documentation.**

---

## How to Make Rules in John the Ripper

John the Ripper's rules are powerful tools for "mangling" words from your wordlist, generating variations that might match a user's password habits. Rules are defined in the `john.conf` configuration file.

### Rule Syntax Basics

Rules are typically placed in sections within `john.conf`, under a header like `[List.Rules:MyRulesetName]`. Each line below this header represents a single rule or a preprocessor directive that generates multiple rules.

A rule consists of a series of single-character commands, which are applied sequentially to the word from the wordlist.

### Common Rule Commands:

Here's a breakdown of some fundamental rule commands:

* **Case Manipulation:**
    * `l`: Convert word to lowercase.
    * `u`: Convert word to uppercase.
    * `c`: Capitalize the first letter (e.g., `password` -> `Password`).
    * `C`: Lowercase the first letter, uppercase the rest (e.g., `Password` -> `pASSWORD`).
    * `t`: Toggle case of all characters (e.g., `Password` -> `pASSWORD`).
    * `TN`: Toggle case of the character at position `N` (e.g., `T0` toggles the first char).

* **Append/Prepend Characters:**
    * `$X`: Append character `X` to the end of the word. `X` can be a specific character or a character class (e.g., `$[0-9]` for any digit).
    * `^X`: Prepend character `X` to the beginning of the word. `X` can be a specific character or a character class.

* **Character Classes for `$` and `^`:**
    * `[0-9]`: Digits 0-9
    * `[a-z]`: Lowercase letters
    * `[A-Z]`: Uppercase letters
    * `[a-zA-Z]`: All letters
    * `[a-zA-Z0-9]`: Letters and digits
    * `[special_chars]`: You can define your own set, e.g., `[!@#$]`

* **Deletion/Insertion/Overstrike:**
    * `D`: Delete the first character.
    * `X`: Delete the last character.
    * `DN`: Delete character at position `N`.
    * `iNX`: Insert character `X` at position `N` (e.g., `i0!` inserts `!` at the beginning).
    * `oNX`: Overstrike (replace) character at position `N` with `X`.

* **Length Control:**
    * `<N`: Reject word if its length is greater than `N`.
    * `>N`: Reject word if its length is less than `N`.
    * `'N`: Truncate word to length `N`.

* **Rejection Flags:**
    * `!X`: Reject word if it contains character `X`.
    * `/X`: Reject word unless it contains character `X`.
    * `(?C`: Reject word unless its first character is in class `C`.
    * `=NX`: Reject word unless character at position `N` is `X`.

### Example: Custom Rule Set in `john.conf`

Let's say you want to create a rule set called `MyCustomRules` that does the following:
1.  Capitalizes the first letter of the word.
2.  Appends "123" to the word.
3.  Appends a random digit (0-9) to the word.
4.  Appends "@" or "!" to the word.
5.  Reverses the word.

You would open your `john.conf` file (usually located in the same directory as the `john` executable, or in `/etc/john` on Linux) and add a new section, typically at the end: