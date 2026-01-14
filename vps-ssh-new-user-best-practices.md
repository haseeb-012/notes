# VPS SSH Setup Guide with ed25519-sk Key

This guide follows **best practice**: SSH key access is configured **only for a non-root user**. Root is used **temporarily** for initial setup and then restricted.

---

## 1. Update System & Change Root Password (INITIAL LOGIN)

After first VPS login (usually via provider password):

```bash
apt update && apt upgrade -y    # Debian/Ubuntu
yum update -y                   # CentOS/RHEL
```

Change root password:

```bash
passwd
```

- Use a **strong, unique password**
- Root password is kept for emergency recovery only

---

## 2. Create a New User (Primary SSH User)

SSH access will be configured **only for this user**.

```bash
adduser haseeb
```

Add user to sudo group:

```bash
usermod -aG sudo haseeb
```

---

## 3. Generate SSH Key on Local Machine

On your **local system** (not VPS):

```bash
ssh-keygen -t ed25519-sk -C "your_email@example.com"
```

- Hardware-backed FIDO2 key
- Default path: `~/.ssh/id_ed25519_sk`
- Set a **strong passphrase**

---

## 4. Copy SSH Key to New User (NOT ROOT)

### Preferred method:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub haseeb@your_vps_ip
```

### Manual method:

Login as root:

```bash
ssh root@your_vps_ip
```

Switch to user:

```bash
su - haseeb
```

Create SSH directory:

```bash
mkdir  ~/.ssh

```

Add public key:

```bash
nano ~/.ssh/authorized_keys
# Paste the contents of your local ~/.ssh/id_ed25519_sk.pub here
```

---

## 5. Test SSH Login as New User

From local machine:

```bash
ssh -i ~/.ssh/id_ed25519_sk haseeb@your_vps_ip
```

✅ Confirm this works **before disabling root SSH**

---

## 6. Secure SSH Configuration (AFTER USER SSH WORKS)

Edit SSH config as root:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended hardened configuration:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
AllowUsers haseeb
```

Restart SSH:

```bash
sudo service ssh restart
```

⚠️ Do NOT restart SSH until user login is confirmed

---
## 6.5. Enable Automatic Security Updates (unattended-upgrades)

Install unattended-upgrades:

```bash
sudo apt install unattended-upgrades apt-listchanges -y
```

Enable the service:

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

Configure automatic updates:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Recommended settings:

```
Unattended-Upgrade::Allowed-Origins {
  "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Mail "root";
Unattended-Upgrade::MailReport "on-change";
```

Verify it's enabled:

```bash
sudo systemctl status unattended-upgrades
```

✅ Security updates now install automatically

---

## 7. Configure Firewall (UFW - Uncomplicated Firewall)

Enable UFW:

```bash
sudo ufw enable
```

Allow SSH (port 22):

```bash
sudo ufw allow 22/tcp
```

Allow HTTP (port 80):

```bash
sudo ufw allow 80/tcp
```

Allow HTTPS (port 443):

```bash
sudo ufw allow 443/tcp
```

View firewall status:

```bash
sudo ufw status
```

Expected output:

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

⚠️ Always allow SSH before enabling UFW to avoid lockout

---
