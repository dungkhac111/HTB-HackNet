<img width="1416" height="153" alt="image" src="https://github.com/user-attachments/assets/57dd4677-59ff-46c4-9c53-12791b0d29a7" />

Nmap
```$ nmap -sC -sV 10.129.232.4
Starting Nmap 7.94SVN ( <https://nmap.org> ) at 2025-12-10 19:42 CST
Nmap scan report for 10.129.232.4
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey:
|   256 95:62:ef:97:31:82:ff:a1:c6:08:01:8c:6a:0f:dc:1c (ECDSA)
|_  256 5f:bd:93:10:20:70:e6:09:f1:ba:6a:43:58:86:42:66 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to <http://hacknet.htb/>
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel```
