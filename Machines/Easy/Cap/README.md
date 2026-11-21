# Hack The Box - Cap

| Information | Details |
| :--- | :--- |
| **OS** | Linux |
| **Difficulty** | Easy |
| **Vulnerabilities** | IDOR, Cleartext Credentials (FTP), Linux Capabilities (`cap_setuid`) |

---

## 1. Enumeration

### Nmap Scan
Start by scanning all open ports and running services on the target machine:

```bash
nmap -Pn -sC -sV 10.129.188.37
```

The scan reveals three open ports:
* **21/tcp:** FTP (`vsftpd 3.0.3`)
* **22/tcp:** SSH (`OpenSSH 8.2p1`)
* **80/tcp:** HTTP (`Gunicorn`)

### Web Enumeration & IDOR Discovery
Navigating to the web server on port `80` shows a network security dashboard logged in as the user **`nathan`**:

<img width="1919" height="892" alt="Dashboard" src="https://github.com/user-attachments/assets/c2517bc7-7bd1-4478-be85-841290f8291f" />

Checking the **IP Config** tab confirms the target's internal network configuration (`10.129.188.37`):

<img width="1407" height="685" alt="IP Config" src="https://github.com/user-attachments/assets/70cdcdec-348e-4fa4-a065-5e2644d12672" />

Moving to the **Security Snapshot (5 Second PCAP + Analysis)** feature, the application captures network traffic and redirects to `/data/2`. Downloading and inspecting `2.pcap` in **Wireshark** reveals no sensitive information:

<img width="1407" height="690" alt="Security Snapshot" src="https://github.com/user-attachments/assets/7d53d885-37cc-4f8e-aaf6-82e1e902fc95" />

<img width="1017" height="647" alt="Wireshark 2.pcap" src="https://github.com/user-attachments/assets/dd1bfb95-0259-4e2f-bd90-49f5bbaa6d87" />

> **Vulnerability Found — IDOR (Insecure Direct Object Reference):**
> The URL endpoint `/data/2` uses a sequential numeric ID without proper access control checks, allowing us to access earlier packet captures.

Testing `/data/1` (`1.pcap`) also yields nothing useful:

<img width="1328" height="810" alt="Wireshark 1.pcap" src="https://github.com/user-attachments/assets/41091da6-ba36-4f1e-bd48-c85b8f9b5855" />

However, changing the URL to `/data/0` allows us to download `0.pcap`. Inspecting this file in **Wireshark** exposes plaintext **FTP** authentication credentials:

<img width="1388" height="402" alt="Wireshark 0.pcap FTP Credentials" src="https://github.com/user-attachments/assets/e4b98b90-ecb3-407e-9c83-e88f03694a5e" />

* **Username:** `nathan`
* **Password:** `Buck3tH4TF0RM3!`

---

## 2. Foothold (Initial Access)

Using the discovered credentials, we log into the **FTP** service to enumerate accessible files and retrieve `user.txt`:

```bash
ftp 10.129.188.37
```

<img width="674" height="305" alt="FTP Login" src="https://github.com/user-attachments/assets/c07a018b-0164-4b03-84cb-98d3c4c0ec18" />

<img width="675" height="98" alt="User Flag" src="https://github.com/user-attachments/assets/62da2df0-1eff-40dc-89e2-48b0fad553a0" />

Since **SSH** (port 22) is also open, we check for password reuse and successfully gain an interactive shell as `nathan`:

```bash
ssh nathan@10.129.188.37
```

---

## 3. Privilege Escalation

Once inside the machine, we enumerate Linux capabilities across the system to look for misconfigured binaries:

```bash
getcap -r / 2>/dev/null
```

<img width="664" height="152" alt="Linux Capabilities" src="https://github.com/user-attachments/assets/50744b9d-79e6-4bd8-a84d-e15ec1d470c3" />

> [!NOTE]
> `/usr/bin/python3.8` has the **`cap_setuid+ep`** capability assigned, which allows any user executing this binary to manipulate process UIDs and escalate privileges to `root` (UID `0`).

We exploit this by launching `python3.8`, setting the UID to `0`, and spawning a privileged `/bin/bash` shell:

```python
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

<img width="648" height="164" alt="Python setuid" src="https://github.com/user-attachments/assets/49634564-9b0c-4cb7-92a0-80dfe0288919" />

Verifying our privileges with `id` confirms we are now `root`, allowing us to read `/root/root.txt`:

<img width="657" height="141" alt="Root Flag" src="https://github.com/user-attachments/assets/4daf37a0-d087-43d8-8ed7-5988d3741b6d" />

---

## 4. Remediation
* **Fix IDOR:** Implement proper session validation and ownership checks on `/data/<id>` endpoints instead of relying on predictable sequential IDs.
* **Avoid Cleartext Protocols:** Replace unencrypted FTP with SFTP or FTPS to prevent credential sniffing over the network.
* **Remove Unnecessary Capabilities:** Strip `cap_setuid` from `/usr/bin/python3.8` using `setcap -r /usr/bin/python3.8`.
