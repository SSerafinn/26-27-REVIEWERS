# System Administration and Maintenance
## Prelim Reviewer

**Coverage:** Introduction to System Administration · The Linux Filesystem & Command Line · Users, Groups & Permissions · Software & Package Management · Processes, Services & systemd

**How to use this reviewer.** Read a section, then close it and try to explain the idea out loud in your own words. Anything you can't explain, go back to. Most of this course is muscle memory — the students who do well are the ones who ran the commands on their own VM, not the ones who only read about them. Boot your server and try things as you go.

---

## 1. Introduction to System Administration

### The role

System administration is the practice of installing, configuring, securing, and maintaining computer systems so they stay **reliable, secure, and available** to the people who depend on them. The administrator's mandate reduces to five words: availability, reliability, security, performance, and recoverability.

The work is broader than fixing what breaks. Recurring responsibilities include:

- Installing and configuring operating systems and services
- Managing user accounts and access
- Monitoring system health and performance
- Backing up data and being able to restore it
- Applying security patches and hardening systems
- Troubleshooting incidents and supporting users
- Automating repetitive tasks
- Documenting configurations, procedures, and changes

Specializations grow out of this base — Linux/Unix, Windows, network, database, cloud, DevOps/SRE, and security administration — but they overlap heavily. A competent Linux administrator is expected to understand networking, security, and increasingly the cloud.

### Availability and service levels

**Uptime** is the percentage of time a system is operational. A **service-level agreement (SLA)** is a formal commitment to a target uptime. The important intuition is how little slack a high percentage actually allows:

| Availability | Downtime per year (approx.) |
|---|---|
| 99% | 3 days, 15 hours |
| 99.9% | 8 hours, 46 minutes |
| 99.99% | 52 minutes |
| 99.999% | 5 minutes |

Two related measures: **MTBF** (mean time between failures — how long a system runs before breaking) and **MTTR** (mean time to recovery — how fast it comes back). Good administration pushes MTBF up and MTTR down.

Be ready to judge whether a given target is appropriate for a given system. A percentage that sounds impressive can still be unacceptable for something critical.

### Documentation, change management, and ethics

If it isn't documented, it doesn't exist. Documentation is what lets a system be maintained by someone other than the person who built it, and it's what saves you during an emergency. Document inventories and configurations, step-by-step procedures, and a change log recording **what** changed, **when**, **why**, and **by whom**. Change management means making changes deliberately and traceably so any change can be understood and reversed.

Administrators hold privileged access, which makes ethics part of the job rather than a footnote. The core principles: integrity, privacy, **least privilege** (grant only the access genuinely needed), accountability, compliance with law and policy, clear communication, and continuous learning. In the Philippines, anyone handling personal information is bound by the **Data Privacy Act of 2012 (RA 10173)**, enforced by the National Privacy Commission.

**Check yourself:** Why is least privilege an ethical principle and not only a technical one? What would you put in a change log entry?

---

## 2. The Linux Filesystem & Command Line

### The shell

The **shell** reads the commands you type and asks the kernel to carry them out. On Ubuntu the default is **Bash**. Administrators work at the command line because it's fast for repetitive work, usable over low-bandwidth remote connections, scriptable, and light on server resources.

Most commands follow one pattern:

```
command [options] [arguments]
```

In `ls -l /home`, `ls` is the command, `-l` an option, `/home` the argument. Options usually start with a dash and often have short (`-a`) and long (`--all`) forms.

### The filesystem hierarchy

Everything lives under a single tree starting at root (`/`). There are no drive letters — additional disks are **mounted** somewhere inside this one tree.

| Directory | Holds |
|---|---|
| `/` | The root of everything |
| `/home` | Regular users' home directories |
| `/root` | The administrator's home directory |
| `/etc` | System-wide configuration files |
| `/bin`, `/sbin` | Essential commands and admin programs |
| `/usr` | Most installed programs and their data |
| `/var` | Data that changes — logs (`/var/log`), mail, spools |
| `/tmp` | Temporary files, usually cleared on reboot |
| `/dev` | Device files representing hardware |
| `/proc` | A virtual view of the running kernel and processes |

Know what kind of thing lives where. Given a path, you should be able to say what each component is and what the file is likely for.

### Navigating

`pwd` prints where you are, `ls` lists contents, `cd` moves you. Useful `ls` options: `-l` (details), `-a` (include hidden dotfiles), `-h` (human-readable sizes); they combine as `ls -lah`.

Path shortcuts: `.` (current directory), `..` (parent), `~` (your home), `-` (previous directory). An **absolute** path starts from `/` and always points to the same place; a **relative** path is interpreted from wherever you currently are.

### Working with files

| Command | Does |
|---|---|
| `cat` | Print a whole file (short files) |
| `less` | Page through a long file (`q` quits) |
| `head` / `tail` | First / last lines |
| `wc` | Count lines, words, characters |
| `mkdir` | Make a directory (`-p` creates parents as needed) |
| `touch` | Create an empty file or update its timestamp |
| `cp` | Copy (`-r` for directories) |
| `mv` | Move — and how you rename |
| `rm` | Remove (`-r` for directories, `-i` to confirm) |

`rm` is permanent. There's no recycle bin. Read the command before you press Enter, especially when a wildcard is involved.

### Wildcards, redirection, and pipes

Wildcards let one command act on many files: `*` matches any number of characters, `?` matches exactly one, `[ ]` matches one character from a set. So `*.log` catches every log file, while `file?.txt` catches `file1.txt` but not `file10.txt`.

Redirection and pipes change where input and output go:

- `>` sends output into a file, **replacing** its contents
- `>>` **appends** output to the end of a file
- `|` pipes one command's output into the next command's input

`ls /etc > list.txt` saves a listing; `ls /etc | less` pages through it without saving. These three operators are the foundation of everything you'll script later.

### Working efficiently

`man command` opens the manual; `command --help` gives a quick summary. Use **Tab** to autocomplete names and the **Up arrow** to recall previous commands. Remember that Linux is case-sensitive: `File.txt` and `file.txt` are different files.

**Check yourself:** What's the difference between `>` and `|`? Starting in `/var/log`, where does `cd ..` put you? Which single command shows a long file one screen at a time?

---

## 3. Users, Groups & Permissions

### The multi-user model

Linux separates every person and service into its own account, which is the basis of its security model. Each account has a numeric **UID** and belongs to at least one group (**GID**). Three kinds of accounts:

- **root** (the superuser, UID 0) — unlimited power over the system
- **Regular users** — real people, UIDs usually starting at 1000
- **System/service accounts** — run background services, not meant for login

Accounts are defined in `/etc/passwd`, encrypted passwords in `/etc/shadow`, and group memberships in `/etc/group`. A line in `/etc/passwd` holds seven colon-separated fields: username, password placeholder, UID, GID, comment, home directory, and login shell.

### Managing users and groups

| Command | Purpose |
|---|---|
| `useradd -m -s /bin/bash name` | Create a user with a home directory and shell |
| `passwd name` | Set or change a password |
| `usermod` | Modify an existing account |
| `userdel -r name` | Delete the user and their home directory |
| `groupadd name` | Create a group |
| `usermod -aG group user` | Add a user to a group |
| `groups user` / `id user` | Inspect memberships |

Groups let you grant the same access to many users at once. Every user has one **primary** group (used for files they create) and any number of **supplementary** groups. The `-a` in `usermod -aG` means *append* — leaving it out replaces all of the user's supplementary groups, which is a classic way to lock someone out of everything.

### Ownership and permissions

Every file has one owner and one associated group, shown by `ls -l`. Change them with `chown` (owner), `chgrp` (group), or `chown user:group file` for both at once.

Three permissions — read (`r`), write (`w`), execute (`x`) — are granted separately to three classes: **owner**, **group**, and **others**. A long listing shows this as ten characters: one file-type character followed by three groups of three.

The permissions mean different things depending on what they're applied to:

| Permission | On a file | On a directory |
|---|---|---|
| read | View its contents | List the names inside |
| write | Change its contents | Create, rename, or delete entries |
| execute | Run it as a program | Enter and pass through it (`cd` into it) |

That last row explains a common puzzle: you can have read on a directory and still not be able to `cd` into it, because entering requires execute.

**Symbolic mode** names a class, an operator, and permissions — `u` (owner), `g` (group), `o` (others), `a` (all) with `+`, `-`, or `=`. So `chmod u+x script.sh` grants the owner execute; `chmod o=r notes.txt` sets others to read-only.

**Numeric (octal) mode** adds values per class: read = 4, write = 2, execute = 1.

| Digit | Permissions | Digit | Permissions |
|---|---|---|---|
| 7 | `rwx` | 4 | `r--` |
| 6 | `rw-` | 0 | `---` |
| 5 | `r-x` | | |

Sets you'll use constantly: `755` (directories and scripts), `644` (ordinary files), `700` (private directory), `640` (owner edits, group reads), `600` (private files and keys), `770` (folder shared by one group).

Practice converting both directions — given `rwxr-xr--` produce the number, and given `750` describe who can do what.

### root, sudo, and umask

root can do anything, which makes logging in as root risky: one mistyped command and there's no safeguard. The standard practice is to work as a regular user and borrow root's power one command at a time with `sudo`. Only users in the sudo group may do it, and every use is logged — that's least privilege plus accountability in practice.

The **umask** filters default permissions on newly created files. Ubuntu's typical `022` yields `644` for new files and `755` for new directories.

**Check yourself:** What exactly does `chmod 750` allow, for each of the three classes? Why does forgetting `-a` in `usermod -aG` cause problems?

---

## 4. Software & Package Management

### Packages and repositories

A **package** bundles precompiled software with its metadata — version, file list, and dependencies. A **package manager** installs, updates, and removes packages and resolves dependencies automatically. A **repository** is an online collection of packages your system is configured to trust, and your machine keeps a local **index** of what's available there.

Two families dominate:

| Family | Format | High-level tool | Low-level tool |
|---|---|---|---|
| Debian / Ubuntu | `.deb` | `apt` | `dpkg` |
| Red Hat / Fedora | `.rpm` | `dnf` | `rpm` |

The high-level tool talks to repositories and resolves dependencies; the low-level tool operates on a single local package file and does neither. That difference explains most package-management confusion.

### Everyday apt

| Command | Does |
|---|---|
| `sudo apt update` | Refresh the local package index |
| `sudo apt upgrade` | Install available updates |
| `apt search` / `apt show` | Find and inspect packages |
| `sudo apt install pkg` | Install a package and its dependencies |
| `sudo apt remove pkg` | Remove the package, keep its config files |
| `sudo apt purge pkg` | Remove the package **and** its config files |
| `sudo apt autoremove` | Clean up dependencies nothing needs |

Two distinctions worth committing to memory: `update` refreshes the *list* while `upgrade` installs the newer *versions*; and `remove` leaves configuration behind while `purge` deletes it.

### Standalone packages and dependencies

When a vendor hands you a `.deb` directly, `sudo dpkg -i file.deb` installs it — but because `dpkg` doesn't resolve dependencies, it may stop with missing-dependency errors. The standard fix is to let apt finish the job with `sudo apt install -f` (`-f` = fix broken dependencies). Understand *why* that works, not just the sequence.

### Repositories and trust

Ubuntu organizes software into **main** (officially supported open source), **restricted** (supported, restrictive licence), **universe** (community-maintained open source), and **multiverse** (legally restricted). Packages are cryptographically signed so the system can verify where they came from.

When software isn't in the default repositories, or you need a newer version than Ubuntu ships, a **PPA** adds a community repository in one step (`sudo add-apt-repository`, then `apt update`, then install). The security consideration matters more than the commands: a third-party repository can install and update software on your system **with root privileges**, so add only sources you trust.

Routine maintenance is `sudo apt update && sudo apt upgrade`, run regularly, with security updates prioritized.

**Check yourself:** Why might `dpkg` fail where `apt` succeeds? What's the practical difference between removing and purging?

---

## 5. Processes, Services & systemd

### Processes

A **program** is a file on disk; a **process** is a running instance of it, loaded into memory and executing. Every process has a numeric **PID** and a parent process (**PPID**).

| Command | Shows |
|---|---|
| `ps aux` | A one-time snapshot of all processes |
| `top` | A live, interactive view (built in) |
| `htop` | A friendlier live view (must be installed) |

When something is slow, the workflow is: open a live view, sort by CPU or memory, read the **COMMAND** column to identify the culprit, and note its **PID** before acting.

### Signals and priority

You control a process by sending it a **signal**:

- **SIGTERM (15)** — the default `kill`. A polite request to shut down cleanly, letting the program save state.
- **SIGKILL (9)** — `kill -9`. Forced, immediate termination with no chance to clean up.

Always try `kill` before `kill -9`; the forced version is a last resort, not a first move. `killall name` signals every process with that name, and `nice`/`renice` adjust scheduling priority.

### Services and systemd

A **service** (or daemon) runs in the background providing ongoing functionality — a web server, a database, a scheduler. On modern Ubuntu, services are managed by **systemd**, the first process the kernel starts (PID 1), which brings up and supervises everything else. You control it with `systemctl`:

| Command | Does |
|---|---|
| `sudo systemctl start svc` | Start it now |
| `sudo systemctl stop svc` | Stop it now |
| `sudo systemctl restart svc` | Stop then start |
| `sudo systemctl reload svc` | Reload config without a full restart |
| `systemctl status svc` | Is it running, plus recent log lines |
| `sudo systemctl enable svc` | Start automatically at every boot |
| `sudo systemctl enable --now svc` | Enable and start in one step |

The distinction students most often miss: **start** affects right now; **enable** affects every future boot. They're independent — a service can be running but not enabled, or enabled but currently stopped.

### Logs and troubleshooting

systemd records service activity in a central journal you query with `journalctl`:

- `journalctl -u service` — everything for one service
- `journalctl -u service -f` — follow live, like `tail -f`
- `journalctl --since "10 min ago"` — recent entries only
- `journalctl -b` — since the last boot

When a service won't start, the pattern is always the same: `systemctl status` tells you it failed, and the journal tells you **why**. Read the last few lines — the real error is usually right there (a port already in use, a syntax error in a config file, a missing directory). Fix the cause rather than re-running the start command.

**Check yourself:** What's the difference between `start` and `enable`? Why is SIGTERM preferred over SIGKILL? Given a failing service, what are your first two commands?

---

## Practice questions

Answer these in your own words, then verify on your VM.

1. A service claims "four nines" availability. How much downtime is that per year, and would you accept it for a hospital's records system?
2. Explain the difference between an absolute and a relative path, using an example of each from your own filesystem.
3. You need every `.conf` file in a directory listed, and the listing saved to a file. What do you run?
4. Convert `rw-r-----` to octal, then describe in words who can do what.
5. A teammate can see a directory's contents listed but can't `cd` into it. What permission is missing?
6. Why is `sudo` preferred over logging in as root, from both a safety and an accountability angle?
7. You install a `.deb` from a vendor and it fails with dependency errors. What happened and what's the fix?
8. Distinguish `apt remove` from `apt purge`, and explain when you'd choose each.
9. A server is sluggish. Walk through your steps from noticing the problem to stopping the offending process.
10. A newly installed web server works now but is gone after a reboot. What was forgotten?

---

## Final advice

Roughly half of this material is *doing* rather than *knowing*. For every command table above, run the commands on your own server and watch what changes. When something fails, read the error message properly before trying anything else — that habit alone separates a systems administrator from someone who reinstalls and hopes.

Good luck.
