# Linux / Unix / AIX – Learning Notes
From beginner to advanced: concepts, commands, and real-world tips.

## 1. What are Linux, Unix, and AIX?
- Unix: multitasking, multiuser OS family (1970s). "Everything is a file". Commercial versions: AIX, HP-UX, Solaris.
- Linux: Unix-like kernel (1991) + GNU tools. Distributions: Ubuntu, Debian, RHEL, Arch, etc.
- AIX: IBM’s proprietary Unix for POWER architecture. Enterprise-grade. Unique tools: SMIT, ODM, built-in LVM, JFS2.

Key takeaway: Linux is Unix-like but not certified Unix. AIX is a true Unix. Core commands are similar, system administration differs.

## 2. System Architecture & Kernel
- Kernel: manages hardware, memory, processes, I/O.
- Shell: command interpreter (bash, sh, ksh, csh, zsh). AIX default: ksh; Linux default: bash.
- User space: applications, libraries.
- Run levels / targets:
  - Traditional Unix (older Linux): runlevel 0–6.
  - Linux systemd: targets (multi-user.target, graphical.target).
  - AIX: /etc/inittab and rc scripts.

Commands to check:
uname -a             # kernel version, architecture
cat /etc/os-release  # Linux distro info
oslevel -s           # AIX version & service pack
prtconf              # AIX hardware configuration

## 3. Command Line Basics (Unix/Linux/AIX common)
ls -la               # list files
cd /path             # change directory
pwd                  # print working directory
cat, less, more      # view file content
mkdir dir            # create directory
rm file, rm -rf dir  # remove file/directory
cp, mv               # copy / move
find / -name "*.log" # find files
grep "pattern" file  # search inside files
ps -ef (Unix), ps aux (Linux)  # view processes
man command          # manual pages

Pipes & redirection:
ls -l | grep "txt" > list.txt
cat /var/log/messages | tail -20

File editing: vi/vim is universal; AIX also has smit for configuration.

## 4. Filesystem Hierarchy & Disks
Standard tree (simplified):
/            Root
/bin         Essential binaries
/sbin        System binaries
/etc         Configuration files
/home        User home directories
/var         Variable data (logs, spool)
/tmp         Temporary files
/usr         User programs
/opt         Optional third-party software
/proc        Process & kernel info (virtual)

Linux specific:
- /sys – kernel & device info
- /dev/sda – first SCSI/SATA disk, partitions /dev/sda1, /dev/sda2 …

AIX specific:
- Disks named hdisk0, hdisk1, etc.
- No /dev/sda; logical volumes under /dev/.
- /proc is present but less rich; use lparstat, vmstat, svmon.

## 5. File Permissions & Ownership
-rwxr-xr-- 1 alice developers 4096 Jan 1 12:00 script.sh
Types: - file, d directory, l symlink.
Permissions: owner (rwx), group (r-x), others (r--).
Numeric: read=4, write=2, execute=1 (e.g., chmod 754 file).

Commands:
chmod 755 script.sh
chown bob:staff file
chgrp staff file
umask 022         # default permission mask

Linux extended attributes: lsattr, chattr (immutable, append-only).
AIX: standard Unix permissions plus ACLs via aclget / aclput.

## 6. User & Group Management
Linux:
useradd -m -s /bin/bash john
passwd john
usermod -aG sudo john
userdel -r john
groupadd developers
User info in /etc/passwd, /etc/shadow, /etc/group.

AIX:
mkuser john
chuser shell=/usr/bin/ksh john
passwd john
rmuser john
mkgroup developers
AIX user attributes stored in ODM. Use lsuser -f john to see details.
Manage via smit → Security & Users.

## 7. Process Management
- Foreground/background: Ctrl+Z to suspend, bg, fg, & to start in background.
- Job control: jobs, kill %1

View processes:
ps -ef          # Unix style
ps aux          # Linux style
top             # interactive, Linux; AIX: topas
htop            # Linux, nicer
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu  # Linux custom output

Signals:
kill -9 PID     # SIGKILL (force)
kill -15 PID    # SIGTERM (graceful)
kill -HUP PID   # reload config

Process tree: pstree (Linux), ps -T (AIX).
Tracing: strace -p PID (Linux), truss -p PID (AIX).

## 8. Package & Software Management
| System          | Command                  | Format          |
|-----------------|--------------------------|-----------------|
| Debian/Ubuntu   | apt, dpkg                | .deb            |
| RHEL/CentOS/Fedora | dnf, yum, rpm         | .rpm            |
| AIX             | installp, rpm (optional) | .bff (backup)   |

Linux examples:
sudo apt update && sudo apt install nginx
sudo dnf install httpd
rpm -qa | grep kernel

AIX:
lslpp -l                    # list installed software
installp -acXd /dev/cd0 all # install from CD
rpm -ivh package.rpm        # Linux RPM packages (Linux toolbox)
smit install_latest         # SMIT menu

## 9. Shell Scripting (Bash / Ksh)
- Shebang: #!/bin/bash or #!/usr/bin/ksh
- Variables: NAME="Linux", use $NAME
- Conditionals:
  if [ "$1" = "start" ]; then
      echo "Starting..."
  fi
- Loops:
  for i in {1..5}; do echo $i; done
  while read line; do echo $line; done < file.txt
- Functions:
  myfunc() { echo "Hello $1"; }
- Command substitution: now=$(date)
- Arithmetic: $((a+b))
- Exit codes: exit 0 (success), exit 1 (fail). Check with $?.
AIX scripting note: AIX ksh93 has its own features; many Linux bash scripts work if ksh is used.

## 10. Networking Essentials
View IP and interfaces:
ip a                  # Linux modern
ifconfig -a           # Deprecated but works, AIX has ifconfig
AIX specific:
ifconfig -a
lsdev -Cc if          # list network devices
netstat -in           # interface stats

Connectivity:
ping google.com
traceroute host
nslookup / dig host   # Linux: dig; AIX: nslookup
curl / wget

Network config files:
- Linux: /etc/network/interfaces (Debian) or /etc/sysconfig/network-scripts/ifcfg-eth0 (RHEL)
- AIX: smit tcpip or mktcpip command, stored in ODM. Hostname in /etc/hosts.

Firewall:
- Linux: iptables/nftables, firewalld
- AIX: genfilt, mkfilt (IPsec filter rules), or smit ips4filter

SSH: universal ssh user@host, key management, scp, sftp.

## 11. Storage Management – LVM and Filesystems
Linux LVM:
Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV)
pvcreate /dev/sdb
vgcreate vg_data /dev/sdb
lvcreate -L 10G -n lv_home vg_data
mkfs.ext4 /dev/vg_data/lv_home
mount /dev/vg_data/lv_home /home
/etc/fstab for permanent mounts.

AIX LVM (built-in):
- PV = hdisk, VG = volume group, LV = logical volume.
lspv          # list physical volumes
lsvg          # list volume groups
lsvg rootvg   # details of rootvg
lslv hd2      # info about LV (hd2 is /usr usually)
Create FS:
crfs -v jfs2 -g rootvg -a size=1G -m /newfs
mount /newfs
Or via smit fs.
Filesystem types: JFS2 (default), enhanced JFS2; /etc/filesystems holds mount info.

Disk space:
df -h          # Linux
df -g          # AIX (in GB)
du -sh /dir

## 12. System Monitoring & Performance
Common tools:
top             # Linux
topas           # AIX full screen performance monitor
vmstat 1 5      # virtual memory stats
iostat 1        # I/O
sar -u 1 5      # CPU history (sysstat package on Linux)
free -m         # memory (Linux)
svmon           # AIX memory (svmon -G)

Linux extras: htop, iotop, dstat, pidstat.
AIX: lparstat for LPAR utilization, nmon (often installed).

Logs:
- Linux: /var/log/messages, syslog, journalctl
- AIX: errpt for hardware/software errors (errpt -a)

## 13. Security Basics
- Root access control: sudo on Linux; AIX can use sudo or RBAC.
- Password policies: /etc/login.defs (Linux), /etc/security/user (AIX, via chsec).
- File integrity: chkrootkit, rkhunter (Linux); AIX: tripwire or AIX Security Expert (aixpert).
- SSH hardening: disable root login, key auth, change port.
Audit: Linux: auditd, AIX: audit subsystem (audit start, audit shutdown).

## 14. System Initialization & Services
Linux (SysV init / systemd):
- SysV: /etc/init.d/, service httpd start, chkconfig
- systemd: systemctl start nginx, systemctl enable nginx, journalctl -u nginx

AIX:
- /etc/inittab controls init, rc scripts in /etc/rc.d/.
- Start/stop services: startsrc -s sshd, stopsrc -s sshd, lssrc -a.
- smit subsys menu.

## 15. Backup & Recovery
- File-level: tar, cpio, rsync.
- Linux disk imaging: dd, partclone.
- AIX backups:
  - mksysb – creates bootable system backup of rootvg.
  - savevg – backup a non-root volume group.
  - restore command for individual files.
- Boot recovery: Linux use GRUB/live CD; AIX uses boot from installation media and “Access a Root Volume Group”.

## 16. Virtualization & Containers
- Linux: KVM, Xen, VirtualBox. Containers: Docker, Podman, LXC. Orchestration: Kubernetes.
- AIX:
  - LPARs (Logical Partitions) – hardware-level virtualization (POWER hypervisor).
  - WPARs (Workload Partitions) – OS-level virtualization, like containers.
  - Commands: lparstat, hmc (Hardware Management Console).
  - WPAR admin: mkwpar, lswpar.

## 17. Automation & Scheduling
- cron: crontab -e (user), /etc/crontab (system). AIX: crontab same, also at for one-time tasks.
- Ansible/Puppet: work across Linux and AIX (AIX modules exist).
- Shell scripts for repeated tasks.

## 18. AIX-Specific Tools and Concepts
| Tool / Concept | Purpose |
|----------------|---------|
| SMIT (smit, smitty) | Menu-driven system administration interface. |
| ODM (Object Data Manager) | Configuration database, stores device info, software, user attributes. |
| LVM | Built-in, highly integrated. mirrorvg, extendvg, reducevg. |
| errpt | Hardware/software error reporting. |
| diag | Hardware diagnostics. |
| JFS2 | Journaled File System, features snapshots, encryption. |
| RBAC | Role Based Access Control – alternative to sudo. |
| PowerVM | Virtualization engine for LPARs, Micro-partitioning. |

## 19. Advanced Topics
Kernel Tuning:
- Linux: /etc/sysctl.conf, sysctl -p.
- AIX: vmo (Virtual Memory Manager), ioo (I/O), no (network), schedo (scheduler).

System Calls & Debugging:
- strace (Linux) / truss (AIX)
- lsof – list open files
- tcpdump – packet analysis
- Core dump analysis: gdb (Linux), dbx (AIX)

Performance Monitoring & Tuning:
- perf (Linux), AIX uses tprof, curt.
- Memory: huge pages configuration.
- I/O elevator tuning (Linux), AIX I/O pacing.

High Availability & Clustering:
- Linux: Pacemaker, Corosync, keepalived.
- AIX: PowerHA (formerly HACMP).

## 20. Learning Path Suggested Order
1. Basic Linux/Unix commands, navigation, file manipulation.
2. Permissions, users, groups.
3. Shell scripting fundamentals.
4. Process management and monitoring.
5. Storage – partitions, LVM (Linux and AIX).
6. Networking.
7. System startup, services, systemd vs AIX init.
8. Package management.
9. Security essentials.
10. Virtualization concepts (LPARs/WPARs on AIX).
11. Automation, backups.
12. Performance analysis & tuning.
13. Advanced AIX administration (ODM, SMIT, mksysb, PowerHA).
