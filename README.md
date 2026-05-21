# Lab 22-1a: Create User Account — `conadm`

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Root or `sudo` access on `server30`  
**Time Estimate:** ~5 minutes

---

## 🎯 Objective

Create a new local user account called `conadm` on `server30`. This is the foundational step before granting elevated privileges and running rootless containers in subsequent labs.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Create a new user | `useradd <username>` |
| Set a password | `passwd <username>` |
| Verify user was created | `id <username>` |
| Check `/etc/passwd` entry | `grep <username> /etc/passwd` |
| Inspect home directory | `sudo ls -la /home/<username>` |

---

## 🔧 Steps

### Step 1 — Elevate to root

```bash
sudo -i
# or
su - root
```

> **Why:** `useradd` and `passwd` write to protected system files (`/etc/passwd`, `/etc/shadow`, `/etc/group`). Regular users have no write access to these files — you must operate as root or use `sudo`.

---

### Step 2 — Create the `conadm` user

```bash
sudo useradd conadm
```

**Expected output:** *(none — silence means success on RHEL-based systems)*

#### What happened behind the scenes?

When `useradd` runs without flags, it uses defaults from `/etc/login.defs` and `/etc/default/useradd` to do the following automatically:

| Action | Location |
|---|---|
| Adds user record | `/etc/passwd` |
| Adds password placeholder | `/etc/shadow` |
| Creates a private group named `conadm` | `/etc/group` |
| Creates home directory | `/home/conadm` |
| Copies shell config templates | From `/etc/skel/` into `/home/conadm/` |

> **RHCSA Tip:** On Red Hat-based systems (RHEL, CentOS, Fedora, Amazon Linux), always use `useradd` — **not** `adduser`. The `adduser` command is an interactive wrapper common on Debian/Ubuntu systems. On RHEL, `useradd` is the standard binary.

---

### Step 3 — Set a password for `conadm`

```bash
sudo passwd conadm
```

**Expected interaction:**

```
Changing password for user conadm.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

#### Output explained line by line

| Line | Meaning |
|---|---|
| `Changing password for user conadm.` | Confirms the target account. |
| `New password:` | Input is hidden. Type the password and press Enter. |
| `BAD PASSWORD: ...` *(if shown)* | PAM (Pluggable Authentication Modules) flagged the password as weak. See note below. |
| `Retype new password:` | Confirmation prompt to catch typos. |
| `passwd: all authentication tokens updated successfully.` | The password was hashed (SHA-512 or yescrypt depending on distro version) and written to `/etc/shadow`. |

> **PAM & root override:** Linux uses PAM (configured in `/etc/pam.d/`) to enforce password complexity rules. If you set a short or simple password, PAM will warn you with a message like `BAD PASSWORD: The password is shorter than 8 characters`. However, because the command was run as root via `sudo`, the system **allows the override** and accepts the password anyway. A normal user running `passwd` on their own account would be **rejected outright** for the same weak password.

---

### Step 4 — Verify the account exists

```bash
id conadm
```

**Expected output:**

```
uid=1001(conadm) gid=1001(conadm) groups=1001(conadm)
```

#### Output explained field by field

| Field | Value | Meaning |
|---|---|---|
| `uid=1001` | User ID | Linux identifies users internally by number, not name. System accounts reserve UIDs 0–999. Standard users start at 1000. Since `ec2-user` is likely UID 1000, `conadm` received the next available number: **1001**. |
| `(conadm)` | Username label | The human-readable name mapped to UID 1001 in `/etc/passwd`. |
| `gid=1001` | Primary Group ID | The group this user belongs to by default. |
| `groups=1001(conadm)` | All group memberships | Lists every group the user belongs to. Right now, only the primary group is shown — no secondary groups have been assigned. |

> **User Private Group (UPG) scheme:** Red Hat systems automatically create a dedicated group with the exact same name and ID as the user (`conadm:1001`). This improves file permission security by ensuring users don't accidentally share a default group with unrelated accounts.

---

### Step 5 — Inspect the `/etc/passwd` entry

```bash
grep conadm /etc/passwd
```

**Expected output:**

```
conadm:x:1001:1001::/home/conadm:/bin/bash
```

#### Why no `sudo`?

The `/etc/passwd` file is world-readable (`-rw-r--r--`). Any user on the system can read it. `sudo` is not required here.

> **What is `grep`?** `grep` stands for **Global Regular Expression Print**. It scans a file line by line looking for a matching pattern (here, the string `"conadm"`) and prints every matching line. It is one of the most-used utilities in Linux administration.

#### `/etc/passwd` format — 7 colon-delimited fields

```
conadm : x : 1001 : 1001 :  : /home/conadm : /bin/bash
  1       2    3      4    5        6               7
```

| Field # | Value | Meaning |
|---|---|---|
| 1 | `conadm` | Username |
| 2 | `x` | Password placeholder. The `x` means the actual password hash is stored securely in `/etc/shadow`. In early Unix systems (1970s–80s), the hash lived here in plaintext — a significant security risk. |
| 3 | `1001` | UID — the kernel's numeric identifier for this user. |
| 4 | `1001` | GID — the user's primary group ID. |
| 5 | *(empty)* | GECOS field — free-form comment, typically the user's full name or contact info. We didn't provide one with `useradd`, so it's blank. |
| 6 | `/home/conadm` | Absolute path to the user's home directory. This is where the user lands after logging in. |
| 7 | `/bin/bash` | Default login shell. Bash is the standard shell for interactive users on RHEL. |

---

### Step 6 — Inspect the home directory

```bash
ls -la /home/conadm
```

**First attempt (without sudo):**

```
ls: cannot open directory '/home/conadm': Permission denied
```

#### Why did it fail?

You are logged in as `ec2-user`. When `useradd` creates a home directory on RHEL/Amazon Linux, it sets the directory permissions to `700` (`drwx------`). This means **only the owner** (`conadm`) and root can read, write, or traverse that directory. `ec2-user` is neither, so the kernel blocks the request.

**Corrected command:**

```bash
sudo ls -la /home/conadm
```

**Expected output:**

```
total 12
drwx------. 2 conadm conadm  62 May 21 14:45 .
drwxr-xr-x. 4 root   root    36 May 21 14:45 ..
-rw-r--r--. 1 conadm conadm  18 Oct 29  2024 .bash_logout
-rw-r--r--. 1 conadm conadm 144 Oct 29  2024 .bash_profile
-rw-r--r--. 1 conadm conadm 522 Oct 29  2024 .bashrc
```

#### Output explained

| Entry | Permissions | Meaning |
|---|---|---|
| `.` | `drwx------` | The home directory itself. Owner-only access (700). |
| `..` | `drwxr-xr-x` | The `/home` parent directory. World-readable/executable, root-owned. |
| `.bash_logout` | `-rw-r--r--` | Runs when `conadm` logs out of a login shell. Default: clears the terminal. |
| `.bash_profile` | `-rw-r--r--` | Runs on login shell startup. Sets environment variables like `$PATH`. |
| `.bashrc` | `-rw-r--r--` | Runs on every interactive (non-login) shell. Defines aliases and prompt settings. |

> **Where do these files come from?** `useradd` copies them from `/etc/skel/`. The skeleton directory (`/etc/skel/`) holds template files that every new user receives. Sysadmins can customize `/etc/skel/` to pre-configure environments for all future users.

> **`-la` flags explained:** `-l` = long format (permissions, ownership, size, date); `-a` = all files including hidden dot-files like `.bashrc`.

---

## ✅ Lab Checklist

- [ ] `sudo useradd conadm` ran silently (no error)
- [ ] Password set and `passwd: all authentication tokens updated successfully.` confirmed
- [ ] `id conadm` returns `uid=1001(conadm) gid=1001(conadm) groups=1001(conadm)`
- [ ] `/etc/passwd` entry shows all 7 fields correctly
- [ ] `sudo ls -la /home/conadm` shows `.bashrc`, `.bash_profile`, `.bash_logout`

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| Running `useradd` without root | `useradd: Permission denied` | Prefix with `sudo` |
| Username already exists | `useradd: user 'conadm' already exists` | Run `id conadm` to verify first |
| No password set | Login fails in container labs | Always run `passwd conadm` after `useradd` |
| Running `ls /home/conadm` without `sudo` | `Permission denied` | Use `sudo ls -la /home/conadm` |
| Wrong shell assigned | Non-interactive behavior | Verify `/bin/bash` in `/etc/passwd` field 7 |

---

## 🧠 Core Concepts Reference

### The `/home` Directory

The **Filesystem Hierarchy Standard (FHS)** defines where things live on a Linux system.

- `/home` is the parent directory for all regular users' personal data.
- On login, users land in their specific folder (e.g., `/home/conadm`).
- Users have full control over their own home directory (`700` by default).
- Users **cannot** write to system directories (`/etc`, `/usr`, `/sbin`) or peer into other users' home directories.

### The `/etc` Directory

`/etc` is the **configuration nerve center** of a Linux system.

- Contains system-wide text configuration files — no executables.
- Key files for user management: `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`.
- When configuring services (SSH, web servers, PAM), you edit files inside `/etc/`.

### `/etc/shadow` vs `/etc/passwd`

| File | Readable by | Contains |
|---|---|---|
| `/etc/passwd` | All users | Username, UID, GID, home, shell — **no real password** |
| `/etc/shadow` | Root only | Hashed password, expiry policies, account aging |

The `x` in field 2 of `/etc/passwd` is a pointer saying: *"the real secret is in `/etc/shadow`."*

---

## 📌 RHCSA Exam Strategy

- `useradd` on RHEL **automatically creates the home directory** — no `-m` flag needed (unlike Debian).
- UIDs **below 1000** are reserved for system accounts.
- **Always verify with `id`** after creating a user — never assume silent success.
- Know `useradd -u <uid> -g <gid> -s <shell> -d <homedir>` for custom user creation scenarios.
- Be comfortable explaining the 7 fields of `/etc/passwd` — it's a common exam question.
- **`Permission denied` on a home directory** = missing `sudo`, not a broken account.

---

## ➡️ Next Lab

**[Lab 22-1b: Grant conadm Full Sudo Rights](https://github.com/kelvintechnical/Grant-Conadm-Full-Rights)**

---

## 🔗 Series Index

| Lab | Topic |
|---|---|
| **22-1a** | *(this lab)* Create user `conadm` |
| [22-1b](https://github.com/kelvintechnical/Grant-Conadm-Full-Rights) | Grant `conadm` full sudo rights |
| [22-1c](https://github.com/kelvintechnical/Verify-Sudo-Access-Conadm) | Verify sudo access |
| [22-1d](https://github.com/kelvintechnical/Inspect-ubi9-with-skopeo) | Inspect ubi9 image with skopeo |
| [22-1e](https://github.com/kelvintechnical/Pull-ubi9-Image-with-podman) | Pull ubi9 image with podman |
| [22-1f](https://github.com/kelvintechnical/launch-container-interative-terminal) | Launch container with port mapping |
| [22-1g](https://github.com/kelvintechnical/run-commands-inside-terminal) | Run commands inside container |
| [22-1h](https://github.com/kelvintechnical/verify-port-mapping-from-host) | Verify port mapping from host |

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
