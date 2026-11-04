# System Administration and Maintenance
## Reviewer: Scripting, Scheduling, Automation, and Networking

**Coverage:** Networking Fundamentals · Network Configuration on Linux · Bash Scripting Basics · Scheduling with cron · Automating Maintenance Tasks

**How to use this reviewer.** Read a section, close it, and explain the idea out loud in your own words. Anything you cannot explain, go back to. Both topics here are skills more than facts: the students who do well are the ones who typed the scripts and changed the network settings on their own VM. Keep your server running and try things as you read.

---

## 1. Networking Fundamentals

### Addresses and prefixes

An **IPv4 address** is 32 bits, written as four numbers from 0 to 255 (for example `172.16.4.100`). Every address has two parts: the **network** part, which all hosts on the same subnet share, and the **host** part, which is unique to one machine.

The **prefix length** (the `/n` after an address) says how many bits belong to the network. It is another way of writing the **subnet mask**:

| Prefix | Subnet mask | Addresses in the subnet | Usable hosts |
|---|---|---|---|
| /8 | 255.0.0.0 | 16,777,216 | 16,777,214 |
| /16 | 255.255.0.0 | 65,536 | 65,534 |
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /30 | 255.255.255.252 | 4 | 2 |

Two addresses in every subnet cannot go to a host: the first one is the **network address**, and the last one is the **broadcast address**. That is why usable hosts are always two fewer than the total.

### A quick subnetting method

For prefixes from /25 to /30, work in the last octet:

1. **Block size** = 256 minus the last octet of the mask (a /27 mask ends in 224, so the block is 32).
2. Subnets start at multiples of the block: .0, .32, .64, .96, .128, …
3. The **network address** is the largest multiple that is not above the host's last octet.
4. The **broadcast address** is one less than the next network address.

*Worked example:* `172.16.4.100/27` → block 32 → multiples 96 and 128 surround 100 → network `172.16.4.96`, broadcast `172.16.4.127`, usable hosts `.97` to `.126`.

Two hosts can talk directly only when they fall in the **same** subnet. To check, work out the network address of each one and compare.

### Special address ranges

| Range | Purpose |
|---|---|
| `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | **Private** addresses, used inside organizations and homes and never routed on the internet |
| `127.0.0.0/8` (usually `127.0.0.1`) | **Loopback**: the machine talking to itself |
| `169.254.0.0/16` | Link-local: what a host gives itself when DHCP does not answer |
| Everything else (with a few exceptions) | Public addresses, reachable across the internet |

Private networks reach the internet through **NAT** (Network Address Translation): the router rewrites the private source address into its own public one. VirtualBox's default network mode works the same way, with VirtualBox playing the router.

### MAC addresses

A **MAC address** (for example `08:00:27:5e:10:a4`) identifies a network card on the local link. IP addresses get packets across networks; MAC addresses deliver them on the last hop inside one network. The first half of a MAC identifies the manufacturer (VirtualBox cards start with `08:00:27`).

### Routing and the default gateway

Every host decides, packet by packet, where to send traffic:

- **Same subnet** → deliver directly on the local link.
- **Any other network** → hand the packet to the **default gateway**, the router that connects this subnet to everything else.

Without a correct gateway, a host can still reach its neighbours but nothing beyond them. Two hosts on different subnets cannot talk even through the same switch unless a router connects the subnets.

### DHCP and DNS

**DHCP** hands a joining host everything it needs: an address, a prefix, a default gateway, and DNS servers. The address comes with a **lease**, a time limit after which the host must renew it. Workstations normally use DHCP. **Servers** normally use **static** addresses (or a DHCP reservation), because clients, DNS records, firewall rules, and scripts point to them and must always find them in the same place.

**DNS** turns names into addresses. When DNS fails, almost everything appears "down" even though the network itself works, because people and programs use names.

### ping and ICMP

`ping` sends ICMP echo requests and reports each reply. Read it from the bottom: the summary line shows packets sent, received, and the percentage lost. Test from near to far, because each success rules out a layer:

1. The **gateway** by address: the local link and your own address work.
2. An **internet host by address**: routing out through the gateway works.
3. An **internet host by name**: DNS works.

If step 2 works but step 3 fails, the problem is name resolution, not the network.

---

## 2. Network Configuration on Linux

### Live state versus saved configuration

This split explains most of the lesson:

| Layer | What it holds | Survives a reboot? |
|---|---|---|
| The **kernel** | The live state: interfaces, addresses, routes | No |
| **Netplan** (`/etc/netplan/*.yaml`) | The saved configuration | Yes, it is read at every boot |
| **systemd-networkd** | Applies Netplan's configuration and requests DHCP | — |
| **systemd-resolved** | Answers name lookups through a local stub at `127.0.0.53` | — |

`ip` reads and changes the **live** state only. Anything you add with `ip` is gone after a reboot. A setting that must last belongs in Netplan.

### Interface names

Ubuntu names cards after where they sit in the hardware, so names stay stable. `enp0s3` means **en** (Ethernet), **p0** (PCI bus 0), **s3** (slot 3). `lo` is the loopback interface.

### The ip command

| Command | Shows or does |
|---|---|
| `ip -br link` | One line per interface: name, state, MAC |
| `ip -br a` | One line per interface with its addresses |
| `ip a show enp0s3` | Full detail for one interface |
| `ip route` | The routing table, including the default gateway |
| `ip route get 9.9.9.9` | Which interface and gateway a packet to that address would use |
| `sudo ip addr add` / `del` | Add or remove an address, until the next reboot |
| `sudo ip link set enp0s3 down` / `up` | Switch an interface off or on (never over SSH!) |

When reading `ip a`, know these words: `state UP` (working), `link/ether` (the MAC), `inet` (the IPv4 address and prefix), `brd` (broadcast), `dynamic` with a counting-down `valid_lft` (a DHCP lease), and `valid_lft forever` (a static address). Lines starting with `inet6` are IPv6.

In `ip route`, the line beginning with `default via` names the gateway. `proto dhcp` means DHCP supplied the route; `proto static` means it came from the saved configuration.

### Netplan files

Netplan files are YAML, and YAML is strict:

- Indent with **spaces only**, never tabs.
- Keys at the same level must line up exactly.
- Write `key: value`, always with a space after the colon.
- List items start with a dash and a space, or go in square brackets on one line.

A static configuration needs the values you collect before changing anything: the interface name, the address with its prefix, the gateway, and the DNS servers. The shape is:

```yaml
network:
    version: 2
    ethernets:
        enp0s8:
            dhcp4: false
            addresses:
                - 192.168.56.20/24
            routes:
                - to: default
                  via: 192.168.56.1
            nameservers:
                addresses: [192.168.56.1, 9.9.9.9]
```

Old tutorials use `gateway4:` for the gateway. It still works but is **deprecated**, and Netplan warns about it; use `routes:` with `to: default` instead. Netplan files should have permissions `600`, because they can contain secrets such as Wi-Fi passwords.

### Applying changes safely

| Command | Behaviour |
|---|---|
| `sudo netplan try` | Checks the YAML, applies it, and waits for you to press Enter. If you don't, it **rolls back** the live network. |
| `sudo netplan apply` | Applies immediately, with no safety net. |
| `sudo netplan generate` | Checks the YAML and reports the **line and column** of a mistake. |
| `sudo netplan get` | Prints the merged configuration. |

Two facts about the rollback matter:

1. It restores the **running network**, not the **file**. After a rollback, the mistake is still in the YAML and will come back at the next reboot or apply. Fix the file.
2. It rolls back to the configuration that was **last generated**. A successful `netplan generate` moves that point to whatever the file holds now. So the safe habit is: edit, then go straight to `netplan try`, and use `generate` only to locate an error that `try` reported.

Always back up the working file before editing it, so returning to a known-good state is one copy away.

### The hostname

`hostname` shows the name, and `hostname -I` lists the machine's IPv4 addresses. `sudo hostnamectl set-hostname NAME` changes the name immediately and saves it in `/etc/hostname`. It does **not** update `/etc/hosts`; edit the `127.0.1.1` line yourself. The prompt shows the new name after you log in again.

### Name resolution

The `hosts:` line in `/etc/nsswitch.conf` sets the lookup order. On Ubuntu Server it is `files dns`: **`/etc/hosts` first, then DNS.** So an entry in `/etc/hosts` wins, and it is a quick way to name a few machines in a small lab, though it has to be copied to every machine that needs it.

DNS on Ubuntu goes through **systemd-resolved**. `/etc/resolv.conf` points programs to the stub at `127.0.0.53`; the real DNS servers are set in Netplan and shown by `resolvectl dns`.

| Tool | Where it looks |
|---|---|
| `getent hosts NAME` | Exactly where programs look: `/etc/hosts`, then DNS |
| `resolvectl query NAME` | DNS through systemd-resolved |
| `dig NAME` / `dig +short NAME` | DNS only; never reads `/etc/hosts` |

When a name works with `ping` but `dig` returns nothing, the name is almost certainly only in `/etc/hosts`. "Name or service not known" means the name could not be turned into an address; the network itself may be fine.

---

## 3. Bash Scripting Basics

Every example below was run on Ubuntu, and the output shown is real. Type them on your own VM; reading them is not enough.

### Anatomy of a script

A script is a text file of commands that Bash runs from top to bottom.

- The first line, the **shebang** `#!/bin/bash`, names the interpreter.
- It needs the **execute permission**: `chmod +x script.sh`.
- Run it with a path, `./script.sh`, because the current folder is not on the `PATH`. `bash script.sh` also works, even without the execute permission.
- Lines starting with `#` are comments.

```bash
#!/bin/bash
# hello.sh - greet the lab
echo "Good morning, SITE Lab!"
echo "Today is $(date +%A)."
```

```console
$ ./hello.sh
bash: ./hello.sh: Permission denied
$ chmod +x hello.sh
$ ./hello.sh
Good morning, SITE Lab!
Today is Wednesday.
```

The first run fails because a new file has no execute permission. After `chmod +x`, the shebang tells the kernel to hand the file to Bash.

### Variables and quoting

- Assign with **no spaces** around `=`: `site="SITE Lab"`. With spaces, Bash tries to run the variable name as a command.
- Read a variable with `$`: `echo "$site"`.
- **Quote your variables.** Without quotes, a value containing spaces splits into several words, so `rm $file` on `old report.txt` tries to delete two files. With `"$file"` it stays one.
- Single quotes print text literally (`'$site'` stays `$site`); double quotes allow variables to expand.

```console
$ room = "Lab 3"
bash: room: command not found
$ room="Lab 3"
$ echo "Room: $room"
Room: Lab 3
$ echo 'Room: $room'
Room: $room
```

Quoting in action, in an empty folder:

```console
$ report="final report.txt"
$ touch $report
$ ls
final  report.txt
```

Without quotes, `touch` received two words and made two files, `final` and `report.txt`. `touch "$report"` makes the single file `final report.txt`.

### Command substitution and arithmetic

| Form | Meaning | Example |
|---|---|---|
| `$(command)` | The output of a command | `now=$(date +%H:%M)` |
| `$((expression))` | Whole-number arithmetic | `echo $((7 * 6))` prints 42 |

Bash arithmetic uses whole numbers only, so `$((7 / 2))` is 3.

```bash
#!/bin/bash
# lab_count.sh - simple arithmetic on lab equipment
pcs=40
broken=3
working=$((pcs - broken))
echo "Working PCs: $working of $pcs"
echo "Groups of 4: $((working / 4))"
echo "Checked by $(whoami) on $(date +%Y-%m-%d)"
```

```console
$ ./lab_count.sh
Working PCs: 37 of 40
Groups of 4: 9
Checked by juan on 2026-11-04
```

Inside `$(( ))`, variables do not need `$`. Note that 37 ÷ 4 gives 9, not 9.25: the remainder is dropped.

### Input and arguments

- `read -p "Prompt: " var` asks a question; `-s` hides what is typed (for passwords); `-r` keeps backslashes as typed.
- Arguments arrive as `$1`, `$2`, …; `$#` is how many there are; `"$@"` is all of them; `$0` is the script's own name.
- `${1:-default}` uses a fallback when no argument is given.
- Scripts that run on a schedule should take **arguments**, not ask questions with `read`, because nobody is there to answer.

```bash
#!/bin/bash
# greet.sh - greet someone by name and section
name="${1:-Student}"
section="${2:-no section}"
echo "Hello, $name of $section!"
echo "You gave $# argument(s)."
```

```console
$ ./greet.sh Ana BSIT-2A
Hello, Ana of BSIT-2A!
You gave 2 argument(s).
$ ./greet.sh
Hello, Student of no section!
You gave 0 argument(s).
$ ./greet.sh "Ana Reyes"
Hello, Ana Reyes of no section!
You gave 1 argument(s).
```

Quotes on the command line keep `"Ana Reyes"` as one argument. With no arguments, both defaults take over.

Reading input instead:

```bash
#!/bin/bash
# login.sh - ask for a username and a hidden password
read -p "Username: " user
read -s -p "Password: " pass
echo
echo "Hi $user, your password has ${#pass} characters."
```

```console
$ ./login.sh
Username: juan
Password:
Hi juan, your password has 7 characters.
```

The password was typed but never shown. The plain `echo` moves to a new line, because `-s` also hides the Enter key. `${#pass}` is the length of the value.

### Exit status, && and ||

Every command finishes with an **exit status**: `0` means success, anything else means failure. `$?` holds the status of the last command. A script sets its own with `exit 0` or `exit 1`, which tells cron or a calling script whether it worked.

- `A && B` runs B only if A **succeeded**.
- `A || B` runs B only if A **failed**. `cd /data || exit 1` is a common safety line.

```console
$ ls /etc/hostname; echo "status: $?"
/etc/hostname
status: 0
$ ls /nothere; echo "status: $?"
ls: cannot access '/nothere': No such file or directory
status: 2
$ mkdir -p ~/demo && echo "created"
created
$ cd /nothere || echo "could not enter"
bash: cd: /nothere: No such file or directory
could not enter
```

### Tests and if

| Test | True when |
|---|---|
| `[ -f path ]` | path is a regular file |
| `[ -d path ]` | path is a directory |
| `[ -e path ]` | path exists |
| `[ -z "$x" ]` / `[ -n "$x" ]` | x is empty / not empty |
| `[ "$a" = "$b" ]` | two strings are equal |
| `-eq -ne -lt -le -gt -ge` | numeric comparisons |
| `!` | reverses a test |

Spaces inside the brackets are required: `[ -d "$dir" ]`. Use the numeric operators for numbers, not `=`. `if` can also test a command directly: `if grep -q "error" app.log; then`. An `if` can have any number of `elif` branches and one `else`, and ends with `fi`.

```bash
#!/bin/bash
# file_type.sh - describe the path given as an argument
if [ -z "$1" ]; then
    echo "Usage: $0 <path>"
    exit 1
fi

if [ -d "$1" ]; then
    echo "$1 is a folder"
elif [ -f "$1" ]; then
    echo "$1 is a file"
else
    echo "$1 does not exist"
fi
```

```console
$ ./file_type.sh /etc
/etc is a folder
$ ./file_type.sh /etc/hostname
/etc/hostname is a file
$ ./file_type.sh /etc/nothing
/etc/nothing does not exist
$ ./file_type.sh; echo "status: $?"
Usage: ./file_type.sh <path>
status: 1
```

The first `if` is a **guard**: it stops the script early, with a non-zero status, when the input is missing. The second checks one condition after another and runs only the first branch that is true.

### case

`case` is cleaner than a long `elif` chain when one value leads to different actions. Each option ends with `;;`, one option can list several patterns with `|`, the `*)` option catches everything else, and the block ends with `esac`. Matching is case-sensitive.

```bash
#!/bin/bash
# lab_status.sh - turn a short code into a message
case "$1" in
    on|up)    echo "The lab is open." ;;
    off|down) echo "The lab is closed." ;;
    *)        echo "Usage: $0 {on|off}"
              exit 1 ;;
esac
```

```console
$ ./lab_status.sh up
The lab is open.
$ ./lab_status.sh off
The lab is closed.
$ ./lab_status.sh OFF; echo "status: $?"
Usage: ./lab_status.sh {on|off}
status: 1
```

`OFF` in capitals matched nothing, so it fell through to `*)`.

### Loops

- `for` repeats over a list: typed items, a range such as `{1..10}` or `{01..10}`, or a wildcard such as `*.csv`.
- If a wildcard matches nothing, the loop runs once with the literal pattern. Guard with `[ -e "$f" ] || continue`.
- `while` repeats as long as its test succeeds, which suits retrying and unknown counts.
- To process a file **line by line**, use `while read -r line; do …; done < file`. Add `IFS=,` to split CSV fields at commas. Avoid `for line in $(cat file)`, which splits on spaces, not lines.
- `read` skips a final line that has no newline after it, so files should end with one.

A typed list and a padded range:

```console
$ for room in 301 302 303; do echo "Checking room $room"; done
Checking room 301
Checking room 302
Checking room 303
$ for n in {01..03}; do echo "pc$n"; done
pc01
pc02
pc03
```

A wildcard in a folder that has no `.csv` files, without and with the guard:

```console
$ for f in *.csv; do echo "Found $f"; done
Found *.csv
$ for f in *.csv; do [ -e "$f" ] || continue; echo "Found $f"; done
```

With the guard, the loop prints nothing at all, which is the correct result for an empty folder.

A `while` loop that counts:

```console
$ tries=1; while [ $tries -le 3 ]; do echo "Attempt $tries"; tries=$((tries + 1)); done
Attempt 1
Attempt 2
Attempt 3
```

Reading a CSV file line by line:

```console
$ cat grades.csv
Ana Reyes,92
Ben Cruz,74
Carla Lim,85
```

```bash
#!/bin/bash
# grades.sh - read a CSV file one line at a time
while IFS=, read -r name score; do
    if [ "$score" -ge 75 ]; then
        echo "PASSED  $name ($score)"
    else
        echo "FAILED  $name ($score)"
    fi
done < grades.csv
```

```console
$ ./grades.sh
PASSED  Ana Reyes (92)
FAILED  Ben Cruz (74)
PASSED  Carla Lim (85)
```

`IFS=,` splits each line at the comma, so the full name keeps its space. `< grades.csv` after `done` feeds the file into the whole loop.

### Functions

A function names a block of commands so it can be reused. Inside it, `$1`, `$2`, … are the function's own arguments. Variables are shared with the rest of the script unless declared with **`local`**, which keeps them inside the function. A function is called by name, like a command: `greet "Ana"`.

```bash
#!/bin/bash
# banner.sh - two small functions, one with a local variable
line() {
    echo "--------------------"
}

greet() {
    local who="$1"
    echo "Welcome, $who!"
}

who="outside"
line
greet "Ana"
greet "Ben"
line
echo "who is still: $who"
```

```console
$ ./banner.sh
--------------------
Welcome, Ana!
Welcome, Ben!
--------------------
who is still: outside
```

Each call to `greet` gets its own `$1`. Because `who` is `local` inside the function, the variable of the same name outside it is untouched.

---

## 4. Scheduling with cron

### The crontab

Each user has a personal schedule: `crontab -e` edits it, `crontab -l` lists it, and `crontab -r` **removes the whole thing**, so be careful. Every line is five time fields and a command:

| Field | Values |
|---|---|
| minute | 0–59 |
| hour | 0–23 |
| day of month | 1–31 |
| month | 1–12 |
| day of week | 0–6 (0 = Sunday) |

| Symbol | Meaning | Example |
|---|---|---|
| `*` | every value | `* * * * *` every minute |
| `*/n` | every n units | `*/20 * * * *` every 20 minutes |
| `a-b` | a range | `0 8 * * 1-5` weekdays at 8:00 AM |
| `a,b` | a list | `0 12,18 * * *` noon and 6:00 PM daily |

More examples: `15 4 * * 0` is Sundays at 4:15 AM; `0 22 15 * *` is 10:00 PM on the 15th of each month. Read a line field by field from the left, and remember the order is **minute first**.

### Why jobs fail under cron

A script that works from your terminal can fail under cron for three reasons:

1. cron starts with a very **short PATH**, so use full paths to scripts and programs.
2. It starts in the user's **home folder**, so relative paths such as `./helper.sh` point to the wrong place.
3. Its output goes to **no screen**, so failures are invisible unless you log them.

Also check that the script has the **execute permission**.

### Logging cron output

| Redirect | Effect |
|---|---|
| `> file` | Replaces the file each run, so only the last run is kept |
| `>> file` | Appends, so every run is kept |
| `2>&1` | Sends error messages to the same place as normal output |

A well-behaved cron line: `0 3 * * * /opt/tools/report.sh >> /var/log/report.log 2>&1`. Without a redirect, both the output and the errors of a job are lost.

### Testing a schedule

Do not wait until 3:00 AM to find out. Add a temporary `* * * * *` line, confirm its output after a minute, then replace it with the real schedule.

---

## 5. Automating Maintenance Tasks

### A pattern for maintenance scripts

Most maintenance scripts follow the same shape. Using a backup script as the example:

1. **Settings at the top**, in variables: what to back up, where to put it, the log file. Take the target as an argument with a default, `${1:-default}`.
2. **A timestamp** in every output name, for example `$(date +%Y-%m-%d_%H%M)`, so runs never overwrite one another. A date with no time collides when a job runs twice in one day.
3. **Guards** that check the source exists and stop with `exit 1` if it does not.
4. **The work**, for example `tar -czf archive.tar.gz folder` (`c` create, `z` compress, `f` file name; swap `c` for `t` to list an archive).
5. **A log function** that writes a timestamped line, so every run leaves evidence.
6. **Retention**: `find DIR -name "pattern" -mtime +N -delete` removes files older than N days. Run `find` without `-delete` first to see what it would remove, and only clean up after the new backup has succeeded.

### Debugging

- `bash -x script.sh` prints every line as it runs, with variables already replaced, so you see exactly where it goes wrong.
- `set -euo pipefail` on the second line makes a script stop at the first failing command or unset variable.
- **ShellCheck** (`sudo apt install shellcheck`) reads a script and warns about unquoted variables and common mistakes.
- Read error messages properly: "command not found" on a variable line usually means spaces around `=`, and `[: missing ']'` means a missing space before `]`.

---

## Final advice

Networking and scripting reward the same habit: **change one thing, then test**. After every Netplan edit, run `ip a`, `ip route`, and a ping. After every few lines of a script, run it. When something fails, read the message before trying anything else; it usually says exactly what is wrong. And before you change anything important, make sure you have a way back.

Good luck.
