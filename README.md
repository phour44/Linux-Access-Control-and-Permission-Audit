# Linux File Permissions and System Administration Lab

**Platform:** Kali Linux (Oracle VirtualBox) &nbsp;|&nbsp;<br>
**Shell:** Bash &nbsp;|&nbsp;<br> 
**User:** phour &nbsp;|&nbsp;<br>
**Author:** Ibrahim Ayomide Fayomi(Cybersecurity Analyst)<br>
**Date:** March 31, 2026<br>

---

## Overview

This repository documents a complete hands-on lab covering Linux file system navigation, permission management, user and group administration, and privilege escalation concepts. Every command was executed live on a Kali Linux terminal. Screenshots at each stage serve as verifiable proof of execution and output.

**The lab is designed as a progressive workflow:** it starts by building a directory structure, applies every major form of permission change in both symbolic and numeric notation, transfers file ownership, creates a new user and group, and finishes with a real-world sudo misconfiguration scenario followed by its remediation.

---

## Key Concepts

**The Linux Permission Model**

Every file and directory has three independent permission classes. Each class has read (r), write (w), and execute (x) flags.

```
-  r w x  r w -  r - -
│  ─────  ─────  ─────
│  User   Group  Others
└─ File type: - = file, d = directory
```

Permissions are expressed in two formats. Symbolic notation uses characters and operators (`chmod o+w`, `chmod a+r`). Octal notation uses a three-digit number where each digit is the sum of granted permission values (r=4, w=2, x=1).

```
chmod 761 movieslist
  │  │  └─ Others: 1 = --x
  │  └──── Group:  6 = rw-  (4+2)
  └─────── User:   7 = rwx  (4+2+1)

Result: -rwxrw---x
```

**Ownership**

`chown` transfers user ownership. `chgrp` transfers group ownership. Both require root privilege — as demonstrated in this lab when a direct `chown` attempt without sudo was rejected with "Operation not permitted," and succeeded only after adding `sudo`.

**Privilege Escalation**

A newly created user has no sudo access by default. This lab captures the exact system response when Layla attempts a privileged command: `Layla is not in the sudoers file.` This event is logged to `/var/log/auth.log`. The remediation — `sudo adduser Layla sudo` — is shown in the final step.

---

## Lab Walkthrough Summary

| Step | Action | Command | Key Result |
|------|--------|---------|-----------|
| 01 | Build directory tree | `mkdir` | Videos/Horror structure created |
| 02 | Create empty file | `touch movieslist` | Default perms: `-rw-rw-r--` |
| 03 | Navigate file system | `cd Videos/Horror` | Directory traversal demonstrated |
| 04 | Navigate to subdirectory | `cd Korean` | Relative path navigation |
| 05 | Audit permissions | `ls -l` | Full permission table displayed |
| 06 | Add write for others | `chmod o+w Comedy` | Comedy → `drwxrwxrwx` |
| 07 | Add execute for all | `chmod u+x,g+x,o+x movieslist` | `movieslist` → `-rwxrwxr-x` |
| 08 | Remove read+execute from all | `chmod u-rx,g-rx,o-rx movieslist` | `movieslist` → `--w--w----` |
| 09 | Restore read for all | `chmod a+r movieslist` | `movieslist` → `-rw-rw-r--` |
| 10 | Octal: execute only | `chmod 111 movieslist` | `movieslist` → `---x--x--x` |
| 11 | Octal: read+write all | `chmod 666 movieslist` | `movieslist` → `-rw-rw-rw-` |
| 12 | Octal: mixed permissions | `chmod 761 movieslist` | `movieslist` → `-rwxrw---x` |
| 13 | Create user | `sudo adduser Layla` | Layla account created |
| 14 | Transfer file ownership | `sudo chown Layla movieslist` | Owner: phour → Layla |
| 15 | Create group | `sudo addgroup NewBorn` | NewBorn group added to system |
| 16 | Transfer group ownership | `sudo chgrp NewBorn movieslist` | Group: phour → NewBorn |
| 17 | Move file into directory | `mv movieslist Horror` | File relocated; permissions intact |
| 18 | Delete file | `sudo rm movieslist` | Permanent deletion; requires sudo |
| 19 | Remove empty directory | `rmdir Horror` | Only succeeds on empty target |
| 20 | Remove directories recursively | `rm -r zeeworld` | Permanent recursive deletion |
| 21 | Log out of phour | System tray → Log Out | Session ended |
| 22 | Sign in as Layla | GDM login screen | Layla's session started |
| 23 | Attempt sudo as Layla | `sudo adduser Cara` | DENIED — not in sudoers |
| 24 | Add Layla to sudo group | `sudo adduser Layla sudo` | Privilege escalation completed |

---

## Lab Highlights

### Permission Audit with ls -l

`ls -l` is the primary tool for reading the full permission state. This output from the lab shows `movieslist` alongside the directory tree:

![05_listing_permissions_users_groups](https://github.com/user-attachments/assets/bb0d8331-50b2-413a-9ef1-c208f58680b2)

```
-rw-rw-r--  1  phour  phour  0  Mar 31 05:08  movieslist
drwxrwxr-x  2  phour  phour  4096  Mar 31 05:07  Horror
```

---

### Stripping All Read and Execute Permissions

`chmod u-rx,g-rx,o-rx movieslist` removes read and execute from all three classes simultaneously, leaving only write. The file becomes practically inaccessible — it can receive data via redirection but cannot be read or executed by anyone, including its owner.

![08_removing_read_execute_ugo](https://github.com/user-attachments/assets/dad82d65-ba3a-4ffc-b337-388070f089ae)


```
Before: -rwxrwxr-x
After:  --w--w----
```

---

### Octal Notation: Setting Mixed Permissions

`chmod 761` demonstrates a realistic mixed-permission configuration. Owner gets full access. Group gets read+write. Others get execute only. Octal notation sets absolute state — it does not add or remove from existing bits.

![12_chmod_761_full_variations](https://github.com/user-attachments/assets/8d9f2490-a14f-44ad-933c-f3264401e045)


```
chmod 761 movieslist  →  -rwxrw---x
          ───
          7 = User:   rwx
          6 = Group:  rw-
          1 = Others: --x
```

---

### Ownership Transfer Requires Root

Attempting `chown Layla movieslist` without sudo returns "Operation not permitted." With sudo, the transfer completes and `ls -l` immediately confirms the new owner.

![14_chown_layla_movieslist](https://github.com/user-attachments/assets/b814041d-eebe-41ed-85a4-f18fe23c81d9)


```
chown Layla movieslist        →  Operation not permitted
sudo chown Layla movieslist   →  SUCCESS

Before: -rwxrw---x  1  phour  phour   0  movieslist
After:  -rwxrw---x  1  Layla  phour   0  movieslist
```

---

### Privilege Escalation: Failed sudo Attempt

Layla, logged in as a standard user, runs `sudo adduser Cara`. The system denies it and logs the event:

![23_layla_no_sudo](https://github.com/user-attachments/assets/9829c3de-550a-4742-905f-4057a15d9ffb)


```
(Layla@kali)-[~]$ sudo adduser Cara
[sudo] password for Layla:
Layla is not in the sudoers file.
```

---

### Remediation: Adding Layla to the sudo Group

From phour's session, a single command grants Layla sudo access. The second run confirms idempotence.

![24_layla_added_sudo_group](https://github.com/user-attachments/assets/1b9b6fc5-b466-462b-8005-cbe1144289f7)


```
sudo adduser Layla sudo
→ Adding user 'Layla' to group 'sudo' ...

sudo adduser Layla sudo   # run again
→ warn: The user 'Layla' is already a member of 'sudo'.
```

---

## Security Insights

**chmod 777 is an attack surface.** World-writable directories allow any local user to create, delete, or rename files within them. This enables symlink attacks, cron job hijacking, and library injection. Use `find / -perm -0002 -type f 2>/dev/null` to audit world-writable files on any system.

**chmod 666 on sensitive files is critical.** Configuration files, private keys, and credential files with world-readable+writable permissions are among the first targets in local privilege escalation. Default permissions on new files should always be validated with `umask`.

**Ownership and permissions are two separate controls.** A file set to `700` (owner-only) appears private — but if the wrong user owns it, they have full control regardless of other settings. Both must be correct.

**Sudo group membership grants unrestricted root access.** The rule `%sudo ALL=(ALL:ALL) ALL` in `/etc/sudoers` applies to all members of the sudo group. Over-assigning this access leads to privilege creep. Audit with `grep '^sudo:' /etc/group` and remove inactive members with `sudo deluser username sudo`.

**Failed sudo attempts are logged.** Every denied sudo call generates an entry in `/var/log/auth.log`. Security teams monitor this log for brute-force escalation attempts. The denial shown in Step 23 is not just a user-facing message — it is a recorded security event.

---

## Commands Used

```bash
# Permission management
chmod o+w Comedy                   # symbolic: add write for others
chmod u+x,g+x,o+x movieslist      # symbolic: add execute for all
chmod u-rx,g-rx,o-rx movieslist   # symbolic: remove read+execute from all
chmod a+r movieslist               # symbolic: add read for all
chmod 111 movieslist               # octal: execute only (---x--x--x)
chmod 666 movieslist               # octal: read+write all (-rw-rw-rw-)
chmod 761 movieslist               # octal: mixed (-rwxrw---x)

# Ownership control
sudo chown Layla movieslist        # transfer user ownership
sudo chgrp NewBorn movieslist      # transfer group ownership

# User and group management
sudo adduser Layla                 # create user interactively
sudo addgroup NewBorn              # create group
sudo adduser Layla sudo            # add user to sudo group
sudo deluser Layla sudo            # revoke sudo access

# File system operations
mkdir -p Videos/Horror             # create nested directories
touch movieslist                   # create empty file
mv movieslist Horror               # move file into directory
sudo rm movieslist                 # delete file (requires sudo here)
rmdir Horror                       # delete empty directory
rm -r zeeworld                     # recursive directory deletion
ls -l                              # audit permissions and ownership
```

---

## Octal Reference

| Symbolic | Octal | Description |
|----------|-------|-------------|
| `-rwxrwxrwx` | `0777` | Full access all — dangerous |
| `-rwxrw---x` | `0761` | Owner full, Group rw, Others execute |
| `-rw-rw-rw-` | `0666` | Read+write all, no execute |
| `-rwxr-xr-x` | `0755` | Standard executable |
| `-rw-r--r--` | `0644` | Standard file |
| `-rw-------` | `0600` | Private file, owner only |
| `---x--x--x` | `0111` | Execute only, all classes |
| `----------` | `0000` | No permissions |

---

*Conducted on Kali Linux 2024 in Oracle VirtualBox. All commands run as user `phour` with `sudo` applied where root privilege was required.*
*Created by: Ibrahim A Fayomi*<br>
*Email: fayomiibrahim44@gmail.com*<br>
*LinkedIn: https://www.linkedin.com/in/fayomi31/*<br>

