# Nmap Penetration Testing Cheatsheet

Nmap (Network Mapper) is a free and open-source utility for network discovery and security auditing. This cheatsheet covers common Nmap switches, their explanations, and typical use cases in penetration testing.

## Table of Contents

* [Basic Scanning](#basic-scanning)
* [Host Discovery](#host-discovery)
* [Port Scanning](#port-scanning)
* [Service and Version Detection](#service-and-version-detection)
* [OS Detection](#os-detection)
* [Scripting Engine (NSE)](#scripting-engine-nse)
* [Timing and Performance](#timing-and-performance)
* [Firewall/IDS Evasion](#firewallids-evasion)
* [Output Formats](#output-formats)
* [Miscellaneous](#miscellaneous)

---

## Basic Scanning

* **`nmap <target>`**:
    * **Explanation**: Performs a basic TCP SYN scan to 1000 common ports. Also attempts host discovery (ping scan).
    * **Use Case**: Quick initial scan to get an overview of open ports on a target.

* **`nmap -v <target>`**:
    * **Explanation**: Increases verbosity, showing more output in real-time. Can be used multiple times (`-vv`) for even more detail.
    * **Use Case**: Useful for monitoring scan progress and understanding what Nmap is doing.

* **`nmap -A <target>`**:
    * **Explanation**: Enables aggressive scan options, including OS detection (`-O`), version detection (`-sV`), script scanning (`-sC`), and traceroute.
    * **Use Case**: Comprehensive scan for a general overview of a target's services, OS, and potential vulnerabilities.

---

## Host Discovery

* **`nmap -sL <target(s)>`**:
    * **Explanation**: List scan. Simply lists targets without sending any packets to them. Useful for verifying target lists.
    * **Use Case**: Confirming the range of IP addresses or hostnames Nmap will scan.

* **`nmap -sn <target(s)>`**: (formerly `-sP`)
    * **Explanation**: Ping scan. Disables port scanning and only performs host discovery. It sends an ICMP echo request, TCP SYN to port 443, TCP ACK to port 80, and an ICMP timestamp request.
    * **Use Case**: Quickly identify live hosts on a network. Does not provide port information.

* **`nmap -Pn <target(s)>`**:
    * **Explanation**: No ping. Treats all hosts as online, skipping the host discovery phase. Useful when targets block ICMP echo requests or other discovery probes.
    * **Use Case**: Scanning hosts that are behind a firewall or are configured not to respond to ping.

* **`nmap -PS <portlist> <target>`**:
    * **Explanation**: TCP SYN Ping. Sends a TCP SYN packet to specified ports (default 80, 443). If a SYN/ACK is received, the host is considered up.
    * **Use Case**: Discovering hosts that might block ICMP, but allow SYN packets to certain ports.

* **`nmap -PA <portlist> <target>`**:
    * **Explanation**: TCP ACK Ping. Sends a TCP ACK packet to specified ports (default 80, 443). If an RST is received, the host is considered up.
    * **Use Case**: Similar to `-PS`, but can sometimes bypass firewalls that block SYN packets.

* **`nmap -PU <portlist> <target>`**:
    * **Explanation**: UDP Ping. Sends UDP packets to specified ports (default 40125). If an ICMP port unreachable message is *not* received, the host is considered up.
    * **Use Case**: Discovering hosts that may only respond to UDP traffic.

* **`nmap -PE -PP -PM <target>`**:
    * **Explanation**: `PE` (ICMP echo), `PP` (ICMP timestamp), `PM` (ICMP netmask) discovery types.
    * **Use Case**: Advanced ICMP-based host discovery, useful for bypassing certain ICMP filters.

---

## Port Scanning

* **`nmap -sS <target>`**:
    * **Explanation**: TCP SYN scan (Stealth Scan or Half-open Scan). Sends a SYN packet and waits for a SYN/ACK. If received, it sends an RST to prevent a full connection. It's fast and stealthy as it doesn't complete the 3-way handshake.
    * **Use Case**: Standard and most popular scan type for identifying open TCP ports without logging a full connection.

* **`nmap -sT <target>`**:
    * **Explanation**: TCP Connect scan. Completes the full 3-way handshake with the target. This scan is less stealthy as connections are logged by the target OS.
    * **Use Case**: When SYN scan is not an option (e.g., non-root user permissions), or when you want to ensure a full connection is established.

* **`nmap -sU <target>`**:
    * **Explanation**: UDP scan. Sends UDP packets to target ports. Relies on ICMP "Port Unreachable" messages to determine if a port is closed. If no response, the port is open|filtered.
    * **Use Case**: Discovering UDP services (DNS, SNMP, DHCP, etc.). Can be very slow due to retransmission delays.

* **`nmap -sN <target>`**:
    * **Explanation**: TCP Null scan. Sends TCP packets with no flags set. Open ports will not respond, closed ports will send an RST.
    * **Use Case**: Attempting to bypass firewalls that are stateful and only inspect SYN packets.

* **`nmap -sF <target>`**:
    * **Explanation**: TCP FIN scan. Sends TCP packets with only the FIN flag set. Open ports will not respond, closed ports will send an RST.
    * **Use Case**: Similar to Null scan, for firewall evasion.

* **`nmap -sX <target>`**:
    * **Explanation**: TCP Xmas scan. Sends TCP packets with FIN, PSH, and URG flags set. Open ports will not respond, closed ports will send an RST.
    * **Use Case**: Another firewall evasion technique.

* **`nmap -sA <target>`**:
    * **Explanation**: TCP ACK scan. Sends TCP ACK packets. It's used to map firewall rulesets, specifically to determine if a firewall is stateful or stateless. It doesn't determine open ports.
    * **Use Case**: Firewall rule mapping, identifying if a firewall is present and how it filters traffic.

* **`nmap -sW <target>`**:
    * **Explanation**: TCP Window scan. Similar to ACK scan, but examines the TCP window size to determine open/closed ports.
    * **Use Case**: Can sometimes detect open ports through certain firewalls.

* **`nmap -sM <target>`**:
    * **Explanation**: TCP Maimon scan. Sends FIN/ACK packets. Closed ports send RST. Open ports may drop the packet.
    * **Use Case**: Advanced, less common scan for evasion.

* **`nmap -sI <zombie_host> <target>`**:
    * **Explanation**: Idlescan. A truly stealthy scan that uses a "zombie" host to bounce scan packets off. The target sees traffic from the zombie, not your IP. Requires a zombie host with incremental IP ID sequence numbers.
    * **Use Case**: Highly stealthy scanning, leaving no direct trace of your IP address on the target. Difficult to perform.

* **`nmap -p <port(s)> <target>`**:
    * **Explanation**: Specifies target port(s).
    * **Examples**:
        * `-p 80`: Port 80 only
        * `-p 21,22,23`: Ports 21, 22, 23
        * `-p 1-100`: Ports 1 through 100
        * `-p U:53,T:21-25`: UDP port 53 and TCP ports 21-25
        * `-p-`: All 65535 ports
    * **Use Case**: Scanning specific ports of interest.

* **`nmap -F <target>`**:
    * **Explanation**: Fast scan. Scans fewer ports than the default scan (around 100).
    * **Use Case**: When you need a quick scan and are less concerned with obscure ports.

---

## Service and Version Detection

* **`nmap -sV <target>`**:
    * **Explanation**: Version detection. Probes open ports to determine the service name and version running on them.
    * **Use Case**: Essential for identifying specific software versions, which can then be checked against vulnerability databases.

* **`nmap --version-intensity <level> <target>`**:
    * **Explanation**: Sets the intensity level for version scanning (0-9). Higher levels are more aggressive and accurate but take longer. Default is 7.
    * **Use Case**: Tuning the aggressiveness of version detection based on desired speed vs. accuracy.

* **`nmap --version-light <target>`**:
    * **Explanation**: A lighter version of `-sV` with intensity level 2.
    * **Use Case**: Faster version detection when time is critical.

* **`nmap --version-all <target>`**:
    * **Explanation**: Tries every single probe, potentially providing more information.
    * **Use Case**: When comprehensive version detection is needed, even if it takes a long time.

* **`nmap -sR <target>`**:
    * **Explanation**: RPC scan. Detects RPC services and their programs, versions, and port numbers.
    * **Use Case**: Identifying RPC-based services, common in Windows environments.

---

## OS Detection

* **`nmap -O <target>`**:
    * **Explanation**: OS detection. Attempts to determine the operating system and device type of the target using TCP/IP stack fingerprinting.
    * **Use Case**: Identifying the target's OS, which helps in tailoring further attacks or vulnerability research.

* **`nmap --osscan-limit <target>`**:
    * **Explanation**: Limits OS detection to promising targets (e.g., those with at least one open and one closed TCP port).
    * **Use Case**: Speeding up scans by not attempting OS detection on every host, only those with good conditions for it.

* **`nmap --osscan-guess <target>`**:
    * **Explanation**: When Nmap can't confidently determine an OS, it will make its best guess.
    * **Use Case**: Obtaining a potential OS match even when Nmap is uncertain.

---

## Scripting Engine (NSE)

* **`nmap -sC <target>`**:
    * **Explanation**: Equivalent to `--script=default`. Runs a set of common and safe scripts.
    * **Use Case**: Quick way to run a collection of useful scripts for information gathering and vulnerability detection.

* **`nmap --script=<script_name or category> <target>`**:
    * **Explanation**: Runs specified Nmap Scripting Engine (NSE) scripts.
    * **Examples**:
        * `--script=http-enum`: Enumerate common web directories.
        * `--script=vuln`: Runs scripts categorized as 'vuln' (vulnerability detection).
        * `--script=ssl-enum-ciphers`: Enumerate SSL/TLS ciphers.
        * `--script=smb-os-discovery`: Discover SMB OS information.
        * `--script=ftp-anon`: Check for anonymous FTP login.
        * `--script=banner`: Grab banners from open ports.
        * `--script="<script1>,<script2>"`: Run multiple specific scripts.
        * `--script="not <category>"`: Run all scripts except those in a category.
        * `--script="<category> and not <script_name>"`: Combine conditions.
    * **Use Case**: Extending Nmap's capabilities for specific tasks like vulnerability detection, service misconfiguration checks, and deeper information gathering.

* **`nmap --script-args <args> <target>`**:
    * **Explanation**: Passes arguments to Nmap scripts.
    * **Example**: `--script-args http-enum.basepath=/wordpress/`
    * **Use Case**: Customizing script behavior based on specific requirements.

* **`nmap --script-help <script_name>`**:
    * **Explanation**: Displays description, arguments, and categories for a specific script.
    * **Use Case**: Learning about what a script does and how to use it.

---

## Timing and Performance

* **`nmap -T<0-5> <target>`**:
    * **Explanation**: Sets the timing template (0-5). Higher numbers are faster but more aggressive and prone to detection.
        * `0`: Paranoid (very slow, IDS evasion)
        * `1`: Sneaky (slower, IDS evasion)
        * `2`: Polite (slow, reduces network load)
        * `3`: Normal (default, balanced)
        * `4`: Aggressive (faster, assumes good network)
        * `5`: Insane (fastest, high network load)
    * **Use Case**: Balancing scan speed with stealth and network impact.

* **`nmap --min-rate <number> <target>`**:
    * **Explanation**: Sets the minimum number of packets sent per second.
    * **Use Case**: Forcing a scan to be faster by sending packets at a minimum rate.

* **`nmap --max-rate <number> <target>`**:
    * **Explanation**: Sets the maximum number of packets sent per second.
    * **Use Case**: Limiting scan speed to avoid network congestion or detection.

* **`nmap --min-hostgroup <number> <target>`**:
    * **Explanation**: Sets the minimum number of hosts to scan in parallel.
    * **Use Case**: Optimizing parallel scanning for large networks.

* **`nmap --max-rtt-timeout <time> <target>`**:
    * **Explanation**: Sets the maximum round-trip time (RTT) for packet retransmission.
    * **Use Case**: Prevents Nmap from waiting too long for responses on unreliable networks.

* **`nmap --initial-rtt-timeout <time> <target>`**:
    * **Explanation**: Sets the initial RTT for packet retransmission.
    * **Use Case**: Useful for speeding up scans on fast, reliable networks by setting a lower initial timeout.

* **`nmap --max-retries <tries> <target>`**:
    * **Explanation**: Sets the maximum number of packet retransmissions.
    * **Use Case**: Controls how persistent Nmap is in getting a response.

* **`nmap --host-timeout <time> <target>`**:
    * **Explanation**: Aborts scanning a host if it takes longer than the specified time.
    * **Use Case**: Prevents Nmap from getting stuck on unresponsive hosts.

---

## Firewall/IDS Evasion

* **`nmap -f <target>`**:
    * **Explanation**: Fragment packets. Splits packets into smaller fragments to evade simple packet filters.
    * **Use Case**: Bypassing firewalls that inspect full packet headers.

* **`nmap -D RND:<num> <target>`**:
    * **Explanation**: Decoy scan. Sends packets from randomly generated decoy IP addresses, making it harder to determine the real source. Requires at least one decoy.
    * **Use Case**: Obscuring your actual IP address in logs.

* **`nmap -D <decoy1_ip>,<decoy2_ip>,ME,<decoy3_ip> <target>`**:
    * **Explanation**: Decoy scan with specific decoy IPs. `ME` represents your own IP address within the decoy list.
    * **Use Case**: Using specific decoy IPs to blend in with legitimate traffic or known internal IPs.

* **`nmap -S <spoof_ip> <target>`**:
    * **Explanation**: Source IP spoofing. Specifies a false source IP address. Responses will go to the spoofed IP, so it's often used with `-Pn` or for specific scenarios.
    * **Use Case**: Highly advanced and generally not practical for getting scan results back, but can be used for denial-of-service or targeted attacks where response is not needed.

* **`nmap -e <interface> <target>`**:
    * **Explanation**: Specifies the network interface to use.
    * **Use Case**: When you have multiple network interfaces and need to choose a specific one.

* **`nmap --source-port <port> <target>`**:
    * **Explanation**: Specifies the source port for all outgoing packets.
    * **Use Case**: Bypassing firewalls that only allow traffic from specific source ports (e.g., DNS, FTP-data).

* **`nmap --data-length <num> <target>`**:
    * **Explanation**: Appends random data to sent packets.
    * **Use Case**: Evading simple pattern-matching firewalls or IDSs.

* **`nmap --ttl <value> <target>`**:
    * **Explanation**: Sets the IPv4 Time-to-Live field.
    * **Use Case**: Mimicking traffic from a specific OS or machine, or trying to bypass TTL-based filtering.

* **`nmap --badsum <target>`**:
    * **Explanation**: Sends packets with a bogus TCP/UDP checksum.
    * **Use Case**: Evading firewalls or IDSs that don't validate checksums, or testing their checksum validation.

---

## Output Formats

* **`nmap -oN <file> <target>`**:
    * **Explanation**: Normal output to a file.
    * **Use Case**: Saving scan results in a human-readable format.

* **`nmap -oX <file> <target>`**:
    * **Explanation**: XML output to a file. Useful for parsing with other tools.
    * **Use Case**: Integrating Nmap results into other security tools or scripts.

* **`nmap -oG <file> <target>`**:
    * **Explanation**: Greppable output to a file. Simple format for scripting and `grep`.
    * **Use Case**: Quickly extracting specific information from scan results using command-line tools.

* **`nmap -oA <basename> <target>`**:
    * **Explanation**: Output to all major formats (normal, XML, greppable) with a specified base name.
    * **Use Case**: Saving all output formats for future reference and varied analysis.

* **`nmap -v -oN - <target>`**:
    * **Explanation**: Verbose output to stdout (`-`).
    * **Use Case**: Viewing real-time output in the terminal.

---

## Miscellaneous

* **`nmap -iL <inputfile> `**:
    * **Explanation**: Reads target specifications from a file. Each line can contain an IP address, hostname, CIDR range, etc.
    * **Use Case**: Scanning a large list of targets efficiently.

* **`nmap --exclude <host(s)> <target(s)>`**:
    * **Explanation**: Excludes specified hosts from the scan.
    * **Use Case**: Skipping hosts that are out of scope or should not be scanned.

* **`nmap --exclude-file <exclude_file> <target(s)>`**:
    * **Explanation**: Excludes hosts listed in a file.
    * **Use Case**: Managing a list of hosts to be excluded from scans.

* **`nmap -p <port> --reason <target>`**:
    * **Explanation**: Displays the reason for a port's state (e.g., "syn-ack", "conn-refused").
    * **Use Case**: Understanding why Nmap determined a port to be open, closed, or filtered.

* **`nmap --open <target>`**:
    * **Explanation**: Only show open ports.
    * **Use Case**: Filtering output to only see relevant open ports.

* **`nmap --packet-trace <target>`**:
    * **Explanation**: Shows every packet sent and received by Nmap.
    * **Use Case**: Debugging scan issues or understanding Nmap's network interactions in detail.

* **`nmap --iflist`**:
    * **Explanation**: Lists all network interfaces and routes.
    * **Use Case**: Troubleshooting network configuration or identifying available interfaces.

* **`nmap --append-output <file>`**:
    * **Explanation**: Appends scan results to an existing output file instead of overwriting it.
    * **Use Case**: Combining results from multiple scans into a single file.

* **`nmap --send-eth <target>`**:
    * **Explanation**: Sends packets at the Ethernet (data link) layer.
    * **Use Case**: Useful on some networks where IP layer raw sockets are blocked or for very specific low-level testing.

* **`nmap --send-ip <target>`**:
    * **Explanation**: Sends packets at the IP (network) layer (default).
    * **Use Case**: Standard packet sending method.

* **`nmap --dns-servers <server1,server2> <target>`**:
    * **Explanation**: Specifies custom DNS servers for resolution.
    * **Use Case**: When default DNS resolution is slow or you want to use specific DNS servers for reconnaissance.

* **`nmap --system-dns <target>`**:
    * **Explanation**: Uses the system's DNS resolver.
    * **Use Case**: Reverting to default system DNS settings.

* **`nmap --traceroute <target>`**:
    * **Explanation**: Performs a traceroute to the target.
    * **Use Case**: Mapping the network path to the target.

* **`nmap --resume <file> `**:
    * **Explanation**: Resumes an aborted scan from a previous XML output file.
    * **Use Case**: Continuing a long scan that was interrupted.

* **`nmap --scan-delay <time> <target>`**:
    * **Explanation**: Specifies a delay between probes.
    * **Use Case**: Evading rate-limiting mechanisms or being very stealthy.

* **`nmap --max-parallelism <num> <target>`**:
    * **Explanation**: Sets the maximum number of parallel probes for host discovery or port scanning.
    * **Use Case**: Controlling the concurrency of the scan.

* **`nmap --unprivileged <target>`**:
    * **Explanation**: Runs Nmap without root privileges, which disables raw socket operations (e.g., SYN scan, OS detection). Only connect scan (`-sT`) is possible.
    * **Use Case**: When you don't have root access but still want to perform basic port scanning.

* **`nmap -v --stats-every <time> <target>`**:
    * **Explanation**: Prints scan statistics periodically during the scan.
    * **Use Case**: Monitoring the progress and efficiency of long scans.

---