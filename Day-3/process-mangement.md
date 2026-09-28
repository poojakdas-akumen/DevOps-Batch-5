# Process Management in Linux
 
A practical guide to viewing, controlling, and monitoring processes on Linux — with commands, examples, and real-world use cases.
 
---
 
## What is a Process?
 
A **process** is a running instance of a program. Every process in Linux has:
 
| Attribute | Meaning |
|---|---|
| **PID** | Process ID — unique number identifying the process |
| **PPID** | Parent Process ID — the process that started it |
| **UID** | User ID — the user the process runs as |
| **State** | Running, sleeping, stopped, zombie, etc. |
| **Priority / Nice** | Determines CPU scheduling order |
 
When you launch a program (e.g., `firefox`), the kernel creates a process, assigns it a PID, and tracks it until it exits.
 
---
 
## Process States
 
| State | Symbol | Meaning |
|---|---|---|
| Running | `R` | Currently executing or ready to run |
| Sleeping | `S` | Waiting for an event (interruptible) |
| Uninterruptible sleep | `D` | Waiting for I/O (cannot be killed easily) |
| Stopped | `T` | Suspended (paused with `Ctrl+Z`) |
| Zombie | `Z` | Finished but parent hasn't read its exit code |
 
---
 
## 1. Viewing Processes
 
### `ps` — snapshot of current processes
 
```bash
# Processes from the current shell
ps
 
# All processes for all users (BSD style — most common)
ps aux
 
# Full format listing with parent PID
ps -ef
 
# Filter by process name
ps aux | grep nginx
 
# Show process tree
ps -ejH
```
 
**Common columns in `ps aux`:** `USER`, `PID`, `%CPU`, `%MEM`, `STAT`, `START`, `TIME`, `COMMAND`.
 
**Use case:** Quick check of what's running, finding a PID before killing a process, or auditing which user owns which service.
 
---
 
### `top` — live, refreshing process view
 
```bash
top
```
 
**Useful keys inside `top`:**
- `P` → sort by CPU usage
- `M` → sort by memory usage
- `k` → kill a process (asks for PID)
- `r` → renice a process
- `1` → toggle per-CPU view
- `q` → quit
**Use case:** Watching CPU/memory live to spot a runaway process or system slowdown.
 
---
 
### `htop` — friendlier interactive `top`
 
```bash
htop
```
 
Color-coded, supports mouse, scrollable, and lets you kill/renice with function keys (F9, F7, F8). Install with:
 
```bash
sudo apt install htop      # Debian / Ubuntu
sudo dnf install htop      # Fedora / RHEL
```
 
**Use case:** Day-to-day monitoring on dev/admin machines — much nicer than plain `top`.
 
---
 
### `pgrep` and `pidof` — find PIDs by name
 
```bash
pgrep nginx                 # PIDs of all nginx processes
pgrep -u alice firefox      # firefox processes owned by alice
pidof sshd                  # PIDs of sshd
```
 
**Use case:** Scripting — get a PID without parsing `ps | grep`.
 
---
 
### `pstree` — show process tree
 
```bash
pstree              # full tree
pstree -p           # show PIDs
pstree alice        # tree for user alice
```
 
**Use case:** Understanding parent–child relationships (e.g., which shell spawned a hung script).
 
---
 
## 2. Killing & Signaling Processes
 
Linux controls processes through **signals**. `kill` doesn't always mean "destroy" — it means "send a signal."
 
### Common signals
 
| Signal | Number | Purpose |
|---|---|---|
| `SIGHUP` | 1 | Hang up — often used to reload config |    Signal Hang Up
| `SIGINT` | 2 | Interrupt (same as `Ctrl+C`) |   Signal Interrupt
| `SIGKILL` | 9 | Force kill — cannot be caught or ignored |   Signal Kill
| `SIGTERM` | 15 | Graceful termination (default) | Signal Terminate
| `SIGSTOP` | 19 | Pause process (cannot be caught) |   Signal Stop
| `SIGCONT` | 18 | Resume a stopped process |   Signal Continue
 
### `kill` — send signals by PID
 
```bash
kill 1234                 # SIGTERM (graceful)
kill -9 1234              # SIGKILL (force)
kill -HUP 1234            # reload config
kill -l                   # list all signal names
```
 
### `killall` — kill by process name
 
```bash
killall firefox
killall -9 chrome
```
 
### `pkill` — kill by name pattern (more flexible)
 
```bash
pkill nginx
pkill -u alice            # kill all processes owned by alice
pkill -f "python myapp"   # match against full command line
```
 
> ⚠️ **Always try `SIGTERM` (15) before `SIGKILL` (9).** `SIGKILL` gives the process no chance to clean up — it can leave corrupted files, locked databases, or stale lock files.
 
**Use case:** Stopping a frozen app, reloading a daemon's config without restart (`SIGHUP`), or cleaning up runaway scripts.
 
---
 
## 3. Background and Foreground Jobs

These commands are part of Linux job control, which manages processes running in the foreground and background of terminal.
 
The shell lets you run commands in the background so your terminal stays free.
 
```bash
# Run in background with &
./long-script.sh &

& starts the process in the background.
The terminal remains available for other commands.
 
# Suspend the foreground process
Ctrl+Z

The process is paused (stopped).
It remains in memory.
 
# List background jobs
jobs

Shows all jobs started from the current shell.
 
# Resume in background
bg %1

bg = background
%1 = job number 1

The job resumes running without taking over the terminal.
 
# Bring back to foreground
fg %1
 
# Kill a job by job number
kill %1

Sends SIGTERM to job 1.
Equivalent to killing its PID, but easier when working with shell jobs.
```
 
### `nohup` — keep running after logout
 
```bash
nohup ./long-script.sh &

Process continues running after logout
Useful when starting a long-running job
```
 
Output goes to `nohup.out` by default. The process survives terminal close.
 
### `disown` — detach an already-running job
 
```bash
./long-script.sh &
disown %1
```
Detaches an already running shell job from the current shell.

The shell no longer tracks the job
It won't receive SIGHUP from the shell when you log out
It disappears from the jobs list
 
**Use case:** Running long backups, builds, or training jobs over SSH without losing them when the connection drops.
 
---
 
## 4. Process Priority — `nice` and `renice`
 
Linux schedules processes using a **niceness** value from `-20` (highest priority) to `19` (lowest). Default is `0`. Only root can set negative (higher) priorities.
 
### Start a process with a specific priority
 
```bash
nice -n 10 ./heavy-job.sh         # lower priority (be nicer)
sudo nice -n -5 ./important.sh    # higher priority
```
 
### Change priority of a running process
 
```bash
renice -n 5 -p 1234               # change PID 1234 to nice 5
sudo renice -n -10 -p 1234        # raise priority (root only)
renice -n 10 -u alice             # all of alice's processes
```
 
**Use case:** Running a CPU-heavy compile/encode in the background without slowing down your editor or browser.
 
---
 
## 5. Monitoring Resource Usage
 
### `free` — memory usage
 
```bash
free -h            # human-readable (MB/GB)
```
 
### `vmstat` — virtual memory + CPU stats
 
```bash
vmstat 2 5         # sample every 2 seconds, 5 times
```
 
### `iostat` — disk I/O (from `sysstat` package)
 
```bash
iostat -x 2

```
iostat = Input/Output Statistics
-x = Extended disk statistics
2 = Refresh every 2 seconds

 
### `uptime` — load averages
 
```bash
uptime
# load average values represent 1, 5, and 15 minute averages
```
 Current time
How long the system has been running
Number of logged-in users
System load averages


### `lsof` — list open files (and which process opened them)
 
```bash
lsof -i :8080            # what's using port 8080?
lsof -p 1234             # files opened by PID 1234
lsof -u alice            # everything alice has open
```
 
**Use case:** "Why is port 8080 already in use?", "Which process is hammering the disk?", or "What's eating my RAM?"
 
---
 
## 6. The `/proc` Filesystem
 
`/proc` is a virtual filesystem exposing kernel and process info as files.

/proc provides information about the Linux kernel, system resources, and running processes.

It does not store real files on disk.
The kernel creates the contents dynamically in memory.
Used for monitoring and tuning the system.
 
```bash
ls /proc/1234/                   # everything about PID 1234
cat /proc/1234/status            # human-readable status
cat /proc/1234/cmdline           # the exact command used
ls -l /proc/1234/cwd             # current working directory
ls -l /proc/1234/exe             # path to the binary
cat /proc/cpuinfo                # CPU details
cat /proc/meminfo                # memory details
```
 
**Use case:** Deep debugging — finding the binary path of a mystery process, checking environment variables, or inspecting open file descriptors.
 
---
 
## 7. Services & Daemons (systemd)
 
Most modern Linux distros use **systemd**is the service manager responsible for starting, stopping, managing services (daemons)and long-running services.

A daemon is a background process that runs without user interaction.

systemctl is the main command used to manage services (daemons) in systems that use systemd.
 
```bash
# Status
systemctl status nginx
 
# Start / stop / restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
 
# Reload config without full restart
sudo systemctl reload nginx
 
# Enable on boot / disable
sudo systemctl enable nginx
sudo systemctl disable nginx
 
# List all running services
systemctl list-units --type=service
 
# View service logs
journalctl -u nginx -f         # follow live
journalctl -u nginx --since today
```
 
**Use case:** Managing web servers, databases, SSH, Docker, and any background service that should auto-start on boot.
 
---
 
## 8. Scheduling Processes
 
### `at` — run a command once at a specific time
 
```bash
echo "/home/alice/backup.sh" | at 23:00
atq                    # list scheduled jobs, Use at when you want a command or script to run one time at a specific date/time.
atrm 3                 # remove job 3
```
 
### `cron` — run commands on a schedule
 
```bash
crontab -e             # edit your crontab
crontab -l             # list scheduled jobs
```
 
Example crontab line — run a script every day at 2:30 AM:
 
```
30 2 * * * /home/alice/scripts/backup.sh
```
 
Format: `minute hour day-of-month month day-of-week command`
 
**Use case:** Automated backups, log rotation, periodic data syncs, certificate renewals.
 
---
 
## Quick Reference
 
| Task | Command |
|---|---|
| List all processes | `ps aux` |
| Live process viewer | `top` / `htop` |
| Find PID by name | `pgrep firefox` |
| Show process tree | `pstree -p` |
| Graceful kill | `kill PID` |
| Force kill | `kill -9 PID` |
| Kill by name | `pkill name` / `killall name` |
| Run in background | `command &` |
| Survive logout | `nohup command &` |
| Lower priority | `nice -n 10 command` |
| Change priority | `renice -n 5 -p PID` |
| Memory usage | `free -h` |
| What's using port X | `lsof -i :X` |
| Service status | `systemctl status name` |
| Service logs | `journalctl -u name -f` |
| Schedule once | `at` |
| Schedule recurring | `crontab -e` |
 
---
 
## Common Troubleshooting Recipes
 
**🔍 A process won't die:**
```bash
kill PID            # try graceful first
kill -9 PID         # force if needed
# Still stuck? Probably state 'D' (uninterruptible I/O) — wait or reboot
```
 
**🔍 Find what's using port 8080:**
```bash
sudo lsof -i :8080
# or
sudo ss -tulpn | grep 8080
```
 
**🔍 System feels slow — find the culprit:**
```bash
top                 # press P for CPU, M for memory
```
 
**🔍 Zombie processes piling up:**
```bash
ps aux | awk '$8=="Z"'      # list zombies
# Kill or restart the parent process — zombies cannot be killed directly
```
 
**🔍 SSH session dropped, lost my long job:**
```bash
# Next time, use:
nohup ./job.sh &
# Or run inside tmux/screen
tmux
```
 
---