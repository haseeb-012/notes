
---

# VPS SSH Setup Guide with ed25519-sk Key

## 1. Generate SSH Key on Local Machine

1. Open your terminal (Linux/macOS) or Git Bash (Windows).
2. Run the command to generate a secure hardware-backed key (`ed25519-sk`):

```bash
ssh-keygen -t ed25519-sk -C "your_email@example.com"
```

* `-t ed25519-sk` → Use the **secure key (FIDO2 / hardware-backed)**.
* `-C` → Add a comment (usually your email).
* Follow prompts:

  * Choose default save location (`~/.ssh/id_ed25519_sk`).
  * Set a **passphrase** for extra security.

**Tip:** Hardware-backed keys require a compatible device (like a security key) for authentication.

---

## 2. Copy Public Key to VPS

Use `ssh-copy-id` (recommended):

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub root@your_vps_ip
```

If `ssh-copy-id` is not available, manually add:

1. Connect to VPS via password (temporary):

```bash
ssh root@your_vps_ip
```

2. Create `.ssh` folder for root:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

3. Paste your public key into `authorized_keys`:

```bash
nano ~/.ssh/authorized_keys
# Paste the content of id_ed25519_sk.pub here
chmod 600 ~/.ssh/authorized_keys
```

4. Exit VPS and test login:

```bash
ssh -i ~/.ssh/id_ed25519_sk root@your_vps_ip
```

---

## 3. Create a New User (Non-Root)

It’s a best practice **not to use root for everyday tasks**.

1. SSH into VPS as root.
2. Create a new user (`haseeb` as example):

```bash
adduser haseeb
```

3. Set password when prompted.
4. Add user to `sudo` group for administrative privileges:

```bash
usermod -aG sudo haseeb
```

---

## 4. Setup SSH for the New User

1. Switch to the new user:

```bash
su - haseeb
```

2. Create `.ssh` folder and set permissions:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

3. Copy your public key from root (or local machine):

```bash
nano ~/.ssh/authorized_keys
# Paste the public key
chmod 600 ~/.ssh/authorized_keys
```

4. Test login from local:

```bash
ssh -i ~/.ssh/id_ed25519_sk haseeb@your_vps_ip
```

---

## 5. Secure SSH Configuration

Edit `/etc/ssh/sshd_config` as root:

```bash
nano /etc/ssh/sshd_config
```

Recommended settings:

```
PermitRootLogin prohibit-password      # disable password login for root
PasswordAuthentication no              # only allow key-based login
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
```

* Save and restart SSH:

```bash
systemctl restart ssh
```

---

## 6. Best Practices

* **Never log in as root** unless necessary; use `sudo`.
* **Use SSH key authentication only**; disable password login.
* **Regularly update VPS**:

```bash
apt update && apt upgrade -y    # Debian/Ubuntu
yum update -y                   # CentOS/RHEL
```

* **Keep your private key secure**; never share it.
* **Use a strong passphrase** for your SSH key.
* **Monitor login attempts**:

```bash
cat /var/log/auth.log | grep sshd   # Debian/Ubuntu
```

* **Optional:** Use `fail2ban` to block brute force attacks.

---

## 7. Folder Structure for Projects (Example)

You can create a structured folder for projects:

```bash
mkdir -p ~/projects/{webapp,scripts,logs}
chmod 755 ~/projects
```

---

✅ Now you can **log in with your new user key**, root login is secure, and your VPS is hardened for SSH access.

---


