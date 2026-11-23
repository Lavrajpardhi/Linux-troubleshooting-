# Basic Linux Troubleshooting Scenarios

A hands-on record of five common Linux problems: how each one was troubleshot, which commands were run, and a simple explanation of every command.

> Most Linux problems are not difficult. The real challenge is knowing what to check first.

---

## Table of Contents

1. [Troubleshooting Approach](#troubleshooting-approach)
2. [Scenario 1: CPU is 100%](#scenario-1-cpu-is-100)
3. [Scenario 2: Memory is Full](#scenario-2-memory-is-full)
4. [Scenario 3: Disk is Full](#scenario-3-disk-is-full)
5. [Scenario 4: Website is Not Working](#scenario-4-website-is-not-working)
6. [Scenario 5: SSH is Not Working](#scenario-5-ssh-is-not-working)
7. [Summary](#summary)

---

## Troubleshooting Approach

Every scenario follows the same five stages, so no random commands are run and the root cause is found in a logical order.

| Stage | Action |
|-------|--------|
| **Problem** | Understand the issue and what the user is experiencing |
| **Check** | Use the relevant commands to collect information |
| **Find** | Identify the root cause from the command output |
| **Fix** | Apply the solution |
| **Verify** | Confirm that the system is working again |

---

## Scenario 1: CPU is 100%

**Problem:** The server is slow and unresponsive because a process is using all the CPU.

**Steps performed**

1. Open a live view of running processes to see which one uses the most CPU.
2. List processes sorted by CPU usage to confirm the top offenders.
3. Note the PID of the heaviest process and check whether it is expected.
4. Stop or restart the process, then check CPU usage again.

**Commands**

```bash
top
ps aux --sort=-%cpu | head
kill PID
```

| Command | Explanation |
|---------|-------------|
| `top` | Shows a live list of processes with CPU and memory usage. Press `q` to quit. |
| `ps aux --sort=-%cpu \| head` | Lists all processes, highest CPU users first, and shows only the top 10 lines. |
| `kill PID` | Asks the process with that PID to stop. Use `kill -9 PID` only if it refuses to stop. |

**Verify:** Run `top` again and confirm CPU usage is back to normal.

---

## Scenario 2: Memory is Full

**Problem:** The system is slow, applications crash, or processes are killed because the server ran out of RAM.

**Steps performed**

1. Check total, used, and free memory.
2. List processes sorted by memory usage to find the biggest consumer.
3. Stop the process or restart its service to free memory.
4. Recheck memory to confirm it was released.

**Commands**

```bash
free -h
ps aux --sort=-%mem | head
sudo systemctl restart service-name
```

| Command | Explanation |
|---------|-------------|
| `free -h` | Shows total, used, free, and available memory in readable units. Check the `available` column. |
| `ps aux --sort=-%mem \| head` | Lists processes, highest memory users first, top 10 lines only. |
| `sudo systemctl restart service-name` | Restarts a service that holds too much memory, which frees it. |

**Verify:** Run `free -h` again and confirm available memory has increased.

---

## Scenario 3: Disk is Full

**Problem:** Applications cannot write files, logs stop updating, or the error `No space left on device` appears.

**Steps performed**

1. Check disk usage of every filesystem to find the one at or near 100%.
2. Look inside the large directory (such as `/var`) to find which folders use the most space.
3. Drill down further, for example into `/var/log`, until the large files are found.
4. Clean up old logs or unused files, then recheck disk usage.

**Commands**

```bash
df -h
du -sh /var/*
sudo journalctl --vacuum-size=200M
```

| Command | Explanation |
|---------|-------------|
| `df -h` | Shows size, used, available, and Use% for each filesystem. |
| `du -sh /var/*` | Shows one total size for every item inside `/var`. Use `sudo` if access is denied. |
| `sudo journalctl --vacuum-size=200M` | Shrinks the systemd journal logs to 200 MB to free space safely. |

**Verify:** Run `df -h` again and confirm Use% has dropped.

---

## Scenario 4: Website is Not Working

**Problem:** The website does not open. The web service may be down, the port may not be listening, or the site may not respond locally.

**Steps performed**

1. Check whether the web server (nginx) is running.
2. Check whether the server is listening on port 80 or 443.
3. Test the site from the server itself to separate a server problem from a network or firewall problem.
4. Check the nginx configuration, then start or restart nginx.

**Commands**

```bash
systemctl status nginx
ss -tulpn
curl localhost
sudo nginx -t
sudo systemctl restart nginx
```

| Command | Explanation |
|---------|-------------|
| `systemctl status nginx` | Shows whether nginx is active, inactive, or failed, with recent log lines. |
| `ss -tulpn` | Lists listening ports and the process using each one. |
| `curl localhost` | Requests the website from the server itself. If it works here but not outside, suspect firewall or network. |
| `sudo nginx -t` | Tests the nginx configuration for syntax errors. |
| `sudo systemctl restart nginx` | Stops and starts nginx so it loads the corrected configuration. |

**Verify:** Run `curl localhost` and open the website in a browser.

---

## Scenario 5: SSH is Not Working

**Problem:** Connecting to the server over SSH fails. Common causes are the SSH service being stopped, port 22 being blocked, a wrong username, or a cloud Security Group blocking access.

**Steps performed**

1. Check whether the server is listening on port 22.
2. Check whether the SSH service is running.
3. Check the firewall and, on cloud servers such as AWS EC2, the Security Group rules for port 22.
4. Confirm the correct username and key, then retry with verbose output.

**Commands**

```bash
ss -tulpn | grep :22
systemctl status sshd
sudo systemctl start sshd
sudo ufw status
ssh -v user@server-ip
```

| Command | Explanation |
|---------|-------------|
| `ss -tulpn \| grep :22` | Shows only port 22. No output means nothing is listening for SSH. |
| `systemctl status sshd` | Shows whether the SSH service is running. On Ubuntu and Debian the name may be `ssh`. |
| `sudo systemctl start sshd` | Starts the SSH service if it was stopped. |
| `sudo ufw status` | Shows whether the firewall allows port 22 (systems using UFW). |
| `ssh -v user@server-ip` | Connects with verbose output to show exactly where the connection fails. |

**Verify:** Connect with `ssh user@server-ip` and reach the server shell.

---

## Summary

| # | Scenario | What I did |
|---|----------|------------|
| 1 | CPU is 100% | Found the heaviest process with `top` and `ps`, stopped or restarted it, confirmed with `top`. |
| 2 | Memory is full | Checked `free -h`, found the top memory process, restarted the service, rechecked memory. |
| 3 | Disk is full | Found the full filesystem with `df -h`, large folders with `du -sh`, cleaned old logs, confirmed with `df -h`. |
| 4 | Website is not working | Checked nginx status, port, and `curl localhost`, then fixed and restarted nginx. |
| 5 | SSH is not working | Checked port 22 and the sshd service, verified firewall or Security Group, reconnected with `ssh -v`. |

---
