# Phase 3: Defensive Strategy with Fail2Ban
## Objective
Implement and evaluate a defensive strategy that protects the victim machine (Metasploitable3) from brute-force SSH login attacks. This phase involves configuring and deploying Fail2Ban, a log-based intrusion prevention tool, to automatically detect and block repeated failed login attempts. The goal is to demonstrate the effectiveness of this defense by comparing the attack results before and after Fail2Ban is activated, ensuring that unauthorized access is prevented and system security is enhanced.
---

## ⚔️ Recap from Phase 1: The Attack

In **Phase 1**, we launched a successful brute-force SSH attack from our **attacker machine (Kali Linux)** against **Metasploitable3 (victim)**.  
The attack used a script with Hydra and weak credentials to access SSH.

---

## 🛡️ Phase 3 Defense: Using Fail2Ban

To defend against this type of attack, we implement **Fail2Ban** on the victim machine.  
This tool automatically bans an IP address after a number of failed SSH login attempts.

---

## 🏁 Step 1: Update and Install Fail2Ban

```bash
sudo apt update
sudo apt install fail2ban -y
---

## ⚙️ Step 2: Configure `jail.local` for SSH Protection

First, we create a local configuration file based on the default one:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
and write the following:
enabled: activates protection for SSH
port: the SSH port to monitor
filter: uses the sshd fail2ban filter rules
logpath: log file to track failed logins
maxretry: number of failed attempts before banning
findtime: time window to count failures (in seconds)
bantime: duration of ban (in seconds)
action: 
```[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 600
bantime = 3600
action:```
## 🔍 Step 4: Monitor Fail2Ban Status

Check if the SSH jail is active and working:

```
sudo fail2ban-client status sshd```


