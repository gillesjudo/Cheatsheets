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


### 1. Rule Structure

Each rule is a sequence of single-character commands, applied from left to right to the input word. You can also have reject flags at the beginning of a rule that determine if a rule should even be applied to a given word.

A rule line can also start with a colon `:` which indicates a *preprocessor directive*. This generates multiple rules from a single line.

### 2. Rule Commands (Simple Commands)

These are the core of John's rules, manipulating the word character by character or as a whole.

### Case Manipulation:

* `l`: Convert the entire word to **lowercase**.
    * `Password` -> `password`
* `u`: Convert the entire word to **uppercase**.
    * `password` -> `PASSWORD`
* `c`: **Capitalize** the first letter.
    * `password` -> `Password`
* `C`: Lowercase the first letter, **uppercase the rest**.
    * `Password` -> `pASSWORD`
* `t`: **Toggle case** of all characters.
    * `Password` -> `pASSWORD`
* `TN`: Toggle case of the character at position `N`. (Positions are 0-indexed).
    * `T0` on `Password` -> `password` (toggles 'P' to 'p')
    * `T1` on `Password` -> `PAssword` (toggles 'a' to 'A')

### Appending/Prepending/Inserting/Overstriking:

* `$X`: **Append** character `X` to the end. `X` can be a literal character (e.g., `$!`) or a character class (e.g., `$[0-9]` for any digit).
    * `password` + `$!` -> `password!`
    * `password` + `$[0-9]` -> `password0`, `password1`, ..., `password9` (generates multiple variations)
* `^X`: **Prepend** character `X` to the beginning. `X` can be a literal character or a character class.
    * `password` + `^A` -> `Apassword`
* `iNX`: **Insert** character `X` at position `N`. (Positions are 0-indexed).
    * `i0!`: Insert `!` at the beginning. `password` -> `!password`
    * `i3@`: Insert `@` at position 3. `password` -> `pas@sword`
* `oNX`: **Overstrike** (replace) character at position `N` with `X`.
    * `o0P`: Replace character at position 0 with `P`. `password` -> `Password`
* `AN"STR"`: **Insert string** `STR` into the word at position `N`. You can use any character as the string delimiter that is not in `STR`. `N` can be `0` for prefixing or `z` for appending.
    * `A0"hello"`: Prepend "hello". `world` -> `helloworld`
    * `Az"123"`: Append "123". `password` -> `password123`

### Deletion:

* `D`: Delete the **first** character.
    * `password` -> `assword`
* `X`: Delete the **last** character.
    * `password` -> `passwor`
* `DN`: Delete character at position `N`.
    * `D3`: Delete character at position 3. `password` -> `pasword`

### Length Control:

* `<N`: **Reject** the word if its length is **greater than N**.
* `>N`: **Reject** the word if its length is **less than N**.
* `'N`: **Truncate** the word to length `N`.
    * `password` + `'4` -> `pass`

### Other Transformations:

* `r`: **Reverse** the word.
    * `password` -> `drowssap`
* `d`: **Duplicate** the word.
    * `password` -> `passwordpassword`
* `f`: **Reflect** the word (append reversed word).
    * `pass` -> `passssap`
* `sXY`: **Substitute** all occurrences of character `X` with `Y`.
    * `sao`: Replace all 'a' with 'o'. `password` -> `posswo_rd`
* `s?C_Y`: **Substitute** all characters of class `C` with `Y`. (See Character Classes below).
    * `s?d_!`: Replace all digits with `!`. `pass123` -> `pass!!!`
* `@X`: **Purge** all occurrences of character `X` (delete them).
    * `@s`: Purge all 's' characters. `password` -> `paword`
* `@?C`: Purge all characters of class `C`.
    * `@?d`: Purge all digits. `pass123` -> `pass`

### Memory Commands:

* `M`: **Memorize** the current state of the word.
* `Q`: **Reject** the word if it hasn't changed since the last `M` command.

### 3. Rule Reject Flags

These appear at the beginning of a rule line, before any commands. If a condition is not met, the rule is skipped for the current word.

* `-`: No-op (always apply).
* `!X`: Reject if the word **contains** character `X`.
* `/X`: Reject unless the word **contains** character `X`.
* `!_?C`: Reject if the word contains a character in class `C`.
* `/_?C`: Reject unless the word contains a character in class `C`.
* `=NX`: Reject unless the character at position `N` is `X`.
* `=N?C`: Reject unless the character at position `N` is in class `C`.
* `(<X)`: Reject unless the word's first character is `X`.
* `(?C)`: Reject unless the word's first character is in class `C`.
* `)X`: Reject unless the word's last character is `X`.
* `)?C`: Reject unless the word's last character is in class `C`.
* `%NX`: Reject unless the word contains at least `N` occurrences of `X`.
* `%N?C`: Reject unless the word contains at least `N` characters of class `C`.

### 4. Character Classes

These are used with commands like `$`, `^`, `s?C_Y`, `@?C`, `!?C`, etc.

* `?a`: Any **alphabetic** character (`[a-zA-Z]`).
* `?d`: Any **digit** (`[0-9]`).
* `?l`: Any **lowercase** letter (`[a-z]`).
* `?u`: Any **uppercase** letter (`[A-Z]`).
* `?x`: Any **alphanumeric** character (`[a-zA-Z0-9]`).
* `?s`: Any **symbol** (`$%^&*()-_+=|\\<>[]{}#@/~`).
* `?p`: Any **punctuation** (`.,:;'?!"` and double quote).
* `?w`: Any **whitespace** (space, tab).
* `?v`: Any **vowel** (`aeiouAEIOU`).
* `?c`: Any **consonant** (`bcdfghjklmnpqrstvwxyzBCDFGHJKLMNPQRSTVWXYZ`).
* `?z`: **All** characters.

To specify the **complement** of a class (e.g., anything *but* digits), uppercase the class character:

* `?D`: Anything *but* a digit.
* `?L`: Anything *but* a lowercase letter.

### 5. Numeric Constants and Variables

Used for specifying positions (`N`), lengths (`N`), or counts.

* `0-9`: Literal digits.
* `A-Z`: Represent values 10-35.
* `*`: `max_length` (maximum plaintext length supported by the current hash type).
* `-`: `max_length - 1`.
* `+`: `max_length + 1`.
* `l`: The word's **current length**.
* `m`: The **last character position** (length - 1).
* `a-k`: User-defined numeric variables (set with `v` command).

### 6. Preprocessor Directives (Starting with `:`)

These are powerful for generating multiple rules from a single line. Each character class or range within a preprocessor directive will expand into multiple individual rules.

Example: `:$[0-9]$[0-9]`
This directive generates 100 rules (10 for the first `$[0-9]` multiplied by 10 for the second `$[0-9]`).
It would expand to rules like:
`$0$0`
`$0$1`
...
`$9$9`

This is far more efficient than writing out each rule individually.

## Example

This is a custom rule file to see an example of how this can look. 

~~~
#
# MyCustomRules.conf - Example John the Ripper Rule File
#
# This file defines custom rule sets for John the Ripper.
# To use this file, you would typically include it in your main john.conf,
# or specify a specific rule set with --rules=MyCommonRules.
#
# Rules are applied sequentially to each word from the wordlist.
#

[List.Rules:MyCommonRules]
# This rule set aims to cover common password variations
# based on a base word (e.g., from a wordlist).

# Basic capitalization and appending common numbers/symbols
c # Capitalize first letter (e.g., "password" -> "Password")
Az"1" # Append "1" (e.g., "Password" -> "Password1")
Az"!" # Append "!" (e.g., "Password" -> "Password!")
Az"@" # Append "@"
Az"23" # Append "23"
Az"789" # Append "789"
Az"0" # Append "0"

# Combine capitalize and append year (last 2 digits common for birthdays)
c Az"99" # Capitalize and append "99" (e.g., "password" -> "Password99")
c Az"00" # Capitalize and append "00"
c Az"01" # Capitalize and append "01"
c Az"02" # Capitalize and append "02"
c Az"03" # Capitalize and append "03"
c Az"04" # Capitalize and append "04"
c Az"05" # Capitalize and append "05"
c Az"06" # Capitalize and append "06"
c Az"07" # Capitalize and append "07"
c Az"08" # Capitalize and append "09"
c Az"09" # Capitalize and append "09"
c Az"10" # Capitalize and append "10"
c Az"11" # Capitalize and append "11"
c Az"12" # Capitalize and append "12"
c Az"13" # Capitalize and append "13"
c Az"14" # Capitalize and append "14"
c Az"15" # Capitalize and append "15"
c Az"16" # Capitalize and append "16"
c Az"17" # Capitalize and append "17"
c Az"18" # Capitalize and append "18"
c Az"19" # Capitalize and append "19"
c Az"20" # Capitalize and append "20"
c Az"21" # Capitalize and append "21"
c Az"22" # Capitalize and append "22"
c Az"23" # Capitalize and append "23"
c Az"24" # Capitalize and append "24"
c Az"25" # Capitalize and append "25"

# Leetspeak transformations (simple ones)
sao # Replace 'a' with 'o'
sa4 # Replace 'a' with '4'
se3 # Replace 'e' with '3'
si1 # Replace 'i' with '1'
so0 # Replace 'o' with '0'
st7 # Replace 't' with '7'
# Often, you'd chain these or use preprocessor directives for more complex leetspeak
# Example for a more comprehensive leet rule (use with caution, can generate many words):
# :s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l s?l?l

# Prepending and Appending common symbols/numbers (using preprocessor for efficiency)
# This will generate rules like: ^! , ^@ , ^# , $! , $@ , $#
:^[@!#$*&] # Prepend common symbols
:$[@!#$*&] # Append common symbols
:^?d # Prepend any digit
:Az?d # Append any digit

# Duplicate word (e.g., "test" -> "testtest")
d

# Reverse word (e.g., "test" -> "tset")
r

# Capitalize and append a common symbol, then a digit
c Az"!" Az?d
c Az"@" Az?d

[List.Rules:CustomLeetspeak]
# A more focused leetspeak rule set

# Simple substitutions
sa4 # a -> 4
se3 # e -> 3
si1 # i -> 1
so0 # o -> 0
st7 # t -> 7
sB8 # B -> 8
sG6 # G -> 6
sS5 # S -> 5
sz2 # z -> 2

# Combinations of common leetspeak
sa4 se3 # a->4, e->3
se3 si1 # e->3, i->1
si1 so0 # i->1, o->0

# Rule to try multiple common leet replacements at once (can be very generative)
# This specific rule generates many variations if combined with a wordlist
# It applies a series of substitutions to each word
sao sa4 se3 si1 so0 st7 sB8 sG6 sS5 sz2

[List.Rules:YearVariations]
# This rule set focuses on appending common year formats
Az"19" # Append "19"
Az"20" # Append "20"
Az"2023" # Append "2023"
Az"2024" # Append "2024"
Az"2025" # Append "2025" # Update this for current year!
Az"99" # Append "99"
Az"00" # Append "00"
Az"21" # Append "21" (for 2021)
Az"22" # Append "22" (for 2022)

# Capitalize and append years
c Az"19"
c Az"20"
c Az"2023"
c Az"2024"
c Az"2025"
c Az"99"
c Az"00"
c Az"21"
c Az"22"
~~~