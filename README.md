# Server-Performance-Stats-test

#!/bin/bash

echo "============================================================"
echo "                 SERVER HEALTH REPORT"
echo "============================================================"
echo "Generated : $(date)"
echo ""

# ============================================================
# SYSTEM INFORMATION
# ============================================================
echo "SYSTEM INFORMATION"
echo "------------------------------------------------------------"

HOSTNAME=$(hostname)
OS=$(grep PRETTY_NAME /etc/os-release | cut -d= -f2 | tr -d '"')
KERNEL=$(uname -r)
UPTIME=$(uptime -p)

echo "Hostname       : $HOSTNAME"
echo "OS Version     : $OS"
echo "Kernel Version : $KERNEL"
echo "Uptime         : $UPTIME"
echo ""

# ============================================================
# LOAD AVERAGE
# ============================================================
echo "LOAD AVERAGE"
echo "------------------------------------------------------------"
uptime | awk -F'load average:' '{print $2}'
echo ""

# ============================================================
# CPU USAGE
# ============================================================
echo "CPU USAGE"
echo "------------------------------------------------------------"

CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')

printf "CPU Usage : %.2f%%\n" "$CPU_USAGE"
echo ""

# ============================================================
# MEMORY USAGE
# ============================================================
echo "MEMORY USAGE"
echo "------------------------------------------------------------"

read TOTAL USED FREE <<< $(free -m | awk '/Mem:/ {print $2, $3, $4}')

MEM_PERCENT=$(awk "BEGIN {printf \"%.2f\", ($USED/$TOTAL)*100}")

echo "Total Memory : ${TOTAL} MB"
echo "Used Memory  : ${USED} MB"
echo "Free Memory  : ${FREE} MB"
echo "Usage        : ${MEM_PERCENT}%"
echo ""

# ============================================================
# DISK USAGE
# ============================================================
echo "DISK USAGE"
echo "------------------------------------------------------------"

df -h /
echo ""

# ============================================================
# LOGGED IN USERS
# ============================================================
echo "LOGGED IN USERS"
echo "------------------------------------------------------------"

USER_COUNT=$(who | wc -l)

echo "Current Users Logged In : $USER_COUNT"
who
echo ""

# ============================================================
# FAILED LOGIN ATTEMPTS
# ============================================================
echo "FAILED LOGIN ATTEMPTS"
echo "------------------------------------------------------------"

if command -v lastb >/dev/null 2>&1; then
    FAILED_COUNT=$(lastb 2>/dev/null | grep -vc "^$")
    echo "Failed Login Attempts : $FAILED_COUNT"
    echo ""
    echo "Last 5 Failed Attempts:"
    lastb 2>/dev/null | head -5
else
    echo "lastb command not available."
fi

echo ""

# ============================================================
# NETWORK INFORMATION
# ============================================================
echo "NETWORK INFORMATION"
echo "------------------------------------------------------------"

IP=$(hostname -I | awk '{print $1}')

echo "Primary IP Address : $IP"
echo ""

# ============================================================
# TOP CPU PROCESSES
# ============================================================
echo "TOP 5 PROCESSES BY CPU"
echo "------------------------------------------------------------"

ps -eo pid,user,comm,%cpu --sort=-%cpu | head -6
echo ""

# ============================================================
# TOP MEMORY PROCESSES
# ============================================================
echo "TOP 5 PROCESSES BY MEMORY"
echo "------------------------------------------------------------"

ps -eo pid,user,comm,%mem --sort=-%mem | head -6
echo ""

# ============================================================
# ZOMBIE PROCESSES
# ============================================================
echo "ZOMBIE PROCESSES"
echo "------------------------------------------------------------"

ZOMBIES=$(ps aux | awk '$8 ~ /^Z/ {count++} END {print count+0}')

echo "Zombie Process Count : $ZOMBIES"
echo ""

# ============================================================
# DISK MOUNT SUMMARY
# ============================================================
echo "DISK MOUNT SUMMARY"
echo "------------------------------------------------------------"

df -h --output=source,size,used,avail,pcent,target | column -t
echo ""

echo "============================================================"
echo "                    REPORT COMPLETED"
echo "============================================================"
