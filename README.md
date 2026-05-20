# Lab 22-1a: Create User Account — `conadm`

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Root or sudo access on server30  
**Time Estimate:** 5 minutes

---

## 🎯 Objective

Create a new local user account called `conadm` on server30.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Create a new user | `useradd <username>` |
| Set a password | `passwd <username>` |
| Verify user was created | `id <username>` |
| Check `/etc/passwd` entry | `grep <username> /etc/passwd` |

---

## 🔧 Steps

### Step 1 — Log in as root or a sudo-enabled user

```bash
su - root
# or
sudo -i
```

---

### Step 2 — Create the `conadm` user

```bash
useradd conadm
```

> No output = success on RHEL.

---

### Step 3 — Set a password for `conadm`

```bash
passwd conadm
```

**Expected prompt:**

```
Changing password for user conadm.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

---

### Step 4 — Verify the account was created

```bash
id conadm
```

**Expected output:**

```
uid=1001(conadm) gid=1001(conadm) groups=1001(conadm)
```

---

### Step 5 — Confirm `/etc/passwd` entry

```bash
grep conadm /etc/passwd
```

**Expected output:**

```
conadm:x:1001:1001::/home/conadm:/bin/bash
```

| Field | Meaning |
|---|---|
| `conadm` | Username |
| `x` | Password stored in `/etc/shadow` |
| `1001` | UID |
| `1001` | GID |
| `/home/conadm` | Home directory |
| `/bin/bash` | Default shell |

---

### Step 6 — Confirm home directory was created

```bash
ls /home/conadm
```

> Should show default shell config files (`.bash_profile`, `.bashrc`, etc.)

---

## ✅ Lab Checklist

- [ ] `useradd conadm` ran without error
- [ ] Password set successfully
- [ ] `id conadm` returns correct UID/GID
- [ ] `/etc/passwd` entry exists
- [ ] `/home/conadm` directory exists

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---|---|
| `useradd` run without root | Prefix with `sudo` |
| Username already exists | `id conadm` to check first |
| No password set | Container labs may require login — always set password |
| Wrong shell assigned | Verify `/bin/bash` in `/etc/passwd` |

---

## 📌 Exam Tips

- `useradd` on RHEL automatically creates the home directory — no `-m` flag needed.
- UIDs below 1000 are reserved for system accounts.
- Always verify with `id` after creating a user — don't assume success.

---

## ➡️ Next Lab

**[Lab 22-1b: Grant conadm Full Sudo Rights](./lab-22-1b-sudo-conadm.md)**

---

## 🔗 Series Index

- [Lab 22-1b: Grant sudo rights](./lab-22-1b-sudo-conadm.md)
- [Lab 22-1c: Verify sudo access](./lab-22-1c-verify-sudo.md)
- [Lab 22-1d: Inspect ubi9 with skopeo](./lab-22-1d-skopeo-inspect.md)
- [Lab 22-1e: Pull ubi9 image](./lab-22-1e-podman-pull.md)
- [Lab 22-1f: Launch container with port mapping](./lab-22-1f-launch-container.md)
- [Lab 22-1g: Run commands inside container](./lab-22-1g-container-commands.md)
- [Lab 22-1h: Verify port mapping from host](./lab-22-1h-verify-port.md)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
