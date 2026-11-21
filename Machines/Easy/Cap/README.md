# Hack The Box - Cap

Difficult: Easy
OS: Linux

## 1. Enumeration

### Nmap Scan
```bash
nmap -Pn -sC -sV 10.129.187.227
```

Nmap reveals three open ports running FTP (21), SSH (22) and an HTTP server on port 80.

## 2. Initial Access

### Check all function on the website

<img width="1919" height="892" alt="image" src="https://github.com/user-attachments/assets/c2517bc7-7bd1-4478-be85-841290f8291f" />
The user is nathan


<img width="1407" height="685" alt="image" src="https://github.com/user-attachments/assets/70cdcdec-348e-4fa4-a065-5e2644d12672" />

Knowning the ip of user nathan is 10.129.188.37


### Try to download the data
<img width="1407" height="690" alt="image" src="https://github.com/user-attachments/assets/7d53d885-37cc-4f8e-aaf6-82e1e902fc95" />

But see nothing in whireshark
<img width="1017" height="647" alt="image" src="https://github.com/user-attachments/assets/dd1bfb95-0259-4e2f-bd90-49f5bbaa6d87" />

### Then realize the url is /data/2 
Try to change the number to 2 and 1 and see the different

Check 1.pcap via whireshark
<img width="1328" height="810" alt="image" src="https://github.com/user-attachments/assets/41091da6-ba36-4f1e-bd48-c85b8f9b5855" />
See nonthing

Check 0.pcap via whireshark
<img width="1388" height="402" alt="image" src="https://github.com/user-attachments/assets/e4b98b90-ecb3-407e-9c83-e88f03694a5e" />
Login successful and then Password through FTP

## 3.Login with the password
Use Username - nathan and Password- Buck3tH4TF0RM3! to login

<img width="674" height="305" alt="image" src="https://github.com/user-attachments/assets/c07a018b-0164-4b03-84cb-98d3c4c0ec18" />


And get the user.txt flag

<img width="675" height="98" alt="image" src="https://github.com/user-attachments/assets/62da2df0-1eff-40dc-89e2-48b0fad553a0" />

## 4. Login ssh

```bash
ssh nathan@10.129.188.37 
```

Check Linux Capabilities

<img width="664" height="152" alt="image" src="https://github.com/user-attachments/assets/50744b9d-79e6-4bd8-a84d-e15ec1d470c3" />

See the python3.8 with cap_setuid

Run os.setuid(0) to set the uid for nathan = 0 (root)

<img width="648" height="164" alt="image" src="https://github.com/user-attachments/assets/49634564-9b0c-4cb7-92a0-80dfe0288919" />

Then open new shell by

```bash
os.system("/bin/bash")
```

<img width="657" height="141" alt="image" src="https://github.com/user-attachments/assets/4daf37a0-d087-43d8-8ed7-5988d3741b6d" />

Then check the id and see now nathan can read the root.txt


