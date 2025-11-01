# 🧭 cPanel Migration Guide

This guide explains how to **migrate cPanel/WHM** from one VM (source server) to another (destination server).

---

## 🖥️ Step 1: Install cPanel on Source VM (Old Server)

### 1️⃣ SSH into the server
```bash
ssh root@<source-server-ip>
```

### 2️⃣ Update packages
```bash
dnf update -y
dnf install -y perl curl wget
reboot
```

### 3️⃣ Set a proper hostname
> cPanel requires a valid **Fully Qualified Domain Name (FQDN)** not used for hosting your actual websites.

```bash
hostnamectl set-hostname server.yourdomain.com
hostnamectl
```

### 4️⃣ Disable SELinux (for installation)
> SELinux restricts service permissions and can block cPanel installation.

```bash
sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
setenforce 0
```

💡 **Note:** You can re-enable SELinux later after configuring proper policies.

---

## ⚙️ Step 2: Install cPanel

### Navigate to the home directory
```bash
cd /home
```

### Download and run the installer
```bash
curl -o latest -L https://securedownloads.cpanel.net/latest
sh latest
```

🕐 Installation Time: **30–60 minutes** depending on server speed.

---

## 🌐 Step 3: Access WHM (WebHost Manager)

After installation completes, open your browser and visit:
```
https://<your-server-ip>:2087
```

**Login Credentials:**
- Username: `root`
- Password: your root password

If root login is disabled:
```bash
sudo passwd root
```

---

## 🔒 Step 4: Post-Installation Tasks

- Install **CSF Firewall**
  ```bash
  yum install csf
  ```

- Set up **Let’s Encrypt SSL**
  > WHM → Manage AutoSSL

- Create new cPanel accounts  
  > WHM → Create a New Account → Fill in domain, username, and password

---

## 🚚 Step 5: Migration to New VM (Using WHM Transfer Tool)

### 1️⃣ Install cPanel on the new VM
Follow **Steps 1–3** above on the **destination server**.

### 2️⃣ Access WHM on new server
```
https://<new-server-ip>:2087
```
Login as `root`.

### 3️⃣ Open Transfer Tool
> WHM → Transfers → **Transfer Tool**

### 4️⃣ Enter source server details
- Server IP: `<old-server-ip>`
- Authentication: Root password or SSH key

### 5️⃣ Select accounts to migrate  
WHM will automatically copy:
- All cPanel accounts (files, emails, databases)
- DNS zones
- SSL certificates
- Packages and configuration

### 6️⃣ Monitor progress
WHM shows real-time migration status.

---

## 🌍 Step 6: Update DNS and Verify

After migration:

1. **Update your domain’s A record** or **nameservers** to point to the **new server’s IP**.  
2. Verify:
   - Websites load correctly  
   - Emails are working  
   - Databases are connected  
3. Once confirmed, you can safely **decommission the old VM**.

---

## 🔑 Step 7: Licensing Note

Each cPanel installation needs a separate license.

- You can **transfer your old license** to the new server IP, or  
- Use a **free trial license** for testing:  
  🔗 [https://verify.cpanel.net/app/free-trial](https://verify.cpanel.net/app/free-trial)

---

## ⚠️ Step 8: Common Mistakes to Avoid

❌ Don’t manually copy `/home` directories.  
❌ Don’t reuse old WHM login credentials on a new server.  
❌ Don’t move cPanel files manually — **always use the Transfer Tool**.

---

## 🧬 Step 9: Optional Post-Migration Configuration

- Set up **remote backups**:  
  WHM → Backup Configuration  
- Enable **Two-Factor Authentication (2FA)**  
- Re-enable **SELinux** (optional)  
- Allow essential ports in firewall:  
  ```bash
  2083, 2087, 2096, 21, 22, 80, 443
  ```

---

## ✅ Final Checklist

☑️ WHM accessible on new server  
☑️ All websites working  
☑️ Email accounts synced  
☑️ DNS updated  
☑️ License applied  
☑️ Backups configured

