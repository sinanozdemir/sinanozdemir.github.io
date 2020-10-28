In this walkthrough, we'll go over a challenge (intermediate) level box called "CMesS". CMesS is one of the Linux Privesc boxes on [TryHackMe](https://tryhackme.com/room/cmess).

![CMesS](CMesS.png)
![CMesS IP](CMesS-IP.png)

Let's start the scanning process with nmap. The IP address would be different when you deploy it:
```bash
nmap -sTV -n -sC -T4 -p- 10.10.227.177 --open
```
![CMesS nmap](CMesS-nmap.png)

Let's also run the vulnerability scripts as well:
```bash
nmap --script vuln -p 22,80 10.10.227.177
```
![CMesS nmap vuln](CMesS-nmap-vuln.png)

However, nmap vuln scan didn't reveal much. Since port 80 is open and Apache httpd 2.4.18 is running, we can start a directory search:
```bash
dirsearch -u http://10.10.227.177 -e php,cgi,html,htm,bak,old,txt -r
```
![CMesS dirsearch-1](CMesS-dirsearch-1.png)
![CMesS dirsearch-2](CMesS-dirsearch-2.png)

After enumerating every folder, I realized that a valid set of credentials required to get into this application:

![Gila index](Gila-index.png)
![Gila admin](Gila-admin.png)

It looks like we have exhausted every option, on my end I also went through every Github exploit related to Apache 2.4.18 (Ubuntu). Next option is to look for subdomains:
```bash
wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hl 107 -H "Host: FUZZ.cmess.thm" -u http://cmess.thm -t 100
```
![CMesS wfuzz](wfuzz.png)

As we can see there is another domain "dev", let's add that to our /etc/hosts file and try to visit the site:

![CMesS /etc/hosts](CMesS-hosts.png)

![CMesS dev domain](CMesS-dev-domain.png)

