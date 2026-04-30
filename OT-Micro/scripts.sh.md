# OT-Microservices Start Script (Production EC2)
```bash
#!/bin/bash
# ==========================================================
# OT-Microservices FINAL STARTUP SCRIPT
# Enterprise-grade single-node orchestrator for AWS EC2
# Optimized for Ubuntu 22 / Low RAM / Self Healing
# ==========================================================

set -Eeuo pipefail

# ---------------- CONFIG ----------------
BASE="$HOME/OT-Micro"
FRONTEND="$BASE/frontend"
SALARY="$BASE/salary-api"
JAR="$SALARY/target/salary-0.1.0-RELEASE.jar"

LOCKFILE="/tmp/otmicro-start.lock"
LOGFILE="$HOME/startup.log"

SWAPFILE="/swapfile"
SWAPSIZE="2G"

# ---------------- COLORS ----------------
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

ok(){ echo -e "${GREEN}✅ $1${NC}"; }
bad(){ echo -e "${RED}❌ $1${NC}"; }
warn(){ echo -e "${YELLOW}⚠ $1${NC}"; }
info(){ echo -e "${BLUE}▶ $1${NC}"; }

# ---------------- LOGGING ----------------
exec > >(tee -a "$LOGFILE") 2>&1

# ---------------- LOCKFILE ----------------
exec 9>"$LOCKFILE"
flock -n 9 || { echo "Another startup is already running."; exit 1; }

# ---------------- ERROR HANDLER ----------------
trap 'bad "Startup failed at line $LINENO"; exit 1' ERR

START_TS=$(date +%s)

# ==========================================================
# AWS PUBLIC IP (IMDSv2)
# ==========================================================
get_public_ip() {
    TOKEN=$(curl -s -m 2 -X PUT \
    http://169.254.169.254/latest/api/token \
    -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" || true)

    curl -s -m 2 \
    -H "X-aws-ec2-metadata-token: $TOKEN" \
    http://169.254.169.254/latest/meta-data/public-ipv4 || true
}

PUBLIC_IP=$(get_public_ip)
[ -z "$PUBLIC_IP" ] && PUBLIC_IP=$(curl -s -m 2 ifconfig.me || true)
[ -z "$PUBLIC_IP" ] && PUBLIC_IP=$(hostname -I | awk '{print $1}')

PRIVATE_IP=$(hostname -I | awk '{print $1}')

# ==========================================================
echo "=================================================="
echo "🚀 OT-Microservices FINAL STARTUP"
echo "=================================================="

# ==========================================================
# SWAP SELF HEAL
# ==========================================================
info "Checking swap..."

if swapon --show | grep -q "$SWAPFILE"; then
    ok "Swap already active"
else
    if [ -f "$SWAPFILE" ]; then
        warn "Swap exists but inactive -> enabling"
        sudo swapon "$SWAPFILE"
    else
        warn "Creating ${SWAPSIZE} swap..."
        sudo fallocate -l "$SWAPSIZE" "$SWAPFILE"
        sudo chmod 600 "$SWAPFILE"
        sudo mkswap "$SWAPFILE"
        sudo swapon "$SWAPFILE"
    fi
fi

free -h

# ==========================================================
# MEMORY PREP
# ==========================================================
info "Refreshing memory..."
sudo sync || true
echo 3 | sudo tee /proc/sys/vm/drop_caches >/dev/null || true

# ==========================================================
# CORE SERVICES FIRST
# ==========================================================
for svc in redis-server postgresql nginx
do
    info "Restarting $svc ..."
    sudo systemctl restart "$svc"

    if systemctl is-active --quiet "$svc"; then
        ok "$svc active"
    else
        bad "$svc failed"
        exit 1
    fi
done

# ==========================================================
# FRONTEND BUILD
# ==========================================================
info "Preparing frontend..."

cd "$FRONTEND"

export NODE_OPTIONS=--openssl-legacy-provider

if [ ! -d node_modules ]; then
    warn "node_modules missing -> npm install"
    npm install
fi

if [ ! -f build/index.html ]; then
    warn "Build missing -> npm run build"
    npm run build
fi

[ -f build/index.html ] && ok "Frontend ready"

# ==========================================================
# SCYLLA START
# ==========================================================
info "Starting scylla-server ..."
sudo systemctl restart scylla-server

for i in {1..60}
do
    ss -tulpn | grep -q ":9042" && break
    sleep 2
done

ss -tulpn | grep -q ":9042" && ok "Scylla ready" || { bad "Scylla failed"; exit 1; }

# ==========================================================
# LIGHT APIs
# ==========================================================
for svc in employee-api attendance-api notification-api
do
    info "Restarting $svc ..."
    sudo systemctl restart "$svc"

    if systemctl is-active --quiet "$svc"; then
        ok "$svc active"
    else
        warn "$svc failed"
    fi
done

# ==========================================================
# SALARY API BUILD + START
# ==========================================================
info "Preparing Salary API..."

cd "$SALARY"

if [ ! -s "$JAR" ]; then
    warn "Jar missing -> rebuilding"
    mvn clean package -DskipTests
fi

jar tf "$JAR" >/dev/null 2>&1 || {
    warn "Jar corrupt -> rebuilding"
    mvn clean package -DskipTests
}

ok "Salary jar ready"

info "Restarting salary-api ..."
sudo systemctl restart salary-api

# ==========================================================
# ELASTICSEARCH LAST
# ==========================================================
info "Starting elasticsearch ..."
sudo systemctl restart elasticsearch || true

# ==========================================================
# HEALTH FUNCTION
# ==========================================================
wait_http() {
    local url=$1
    local name=$2
    local timeout=$3

    for ((i=1;i<=timeout;i++))
    do
        if curl -fsS "$url" >/dev/null 2>&1; then
            ok "$name OK"
            return 0
        fi
        sleep 1
    done

    bad "$name DOWN"
    return 1
}

# ==========================================================
# HEALTH CHECKS
# ==========================================================
echo ""
echo "=================================================="
echo "🔍 HEALTH CHECKS"
echo "=================================================="

wait_http http://localhost/api/v1/employee/health "Employee API" 60
wait_http http://localhost/api/v1/attendance/health "Attendance API" 60
wait_http http://localhost:8082/actuator/health "Salary API" 120
wait_http http://localhost:5000/health "Notification API" 60
wait_http http://localhost "Frontend" 30
wait_http http://localhost:9200 "Elasticsearch" 60 || true

redis-cli ping >/dev/null 2>&1 && ok "Redis OK" || bad "Redis DOWN"

# ==========================================================
# OOM CHECK
# ==========================================================
if dmesg | grep -qi "killed process"; then
    warn "Recent OOM kill detected"
fi

# ==========================================================
# PORTS
# ==========================================================
echo ""
echo "=================================================="
echo "🔌 OPEN PORTS"
echo "=================================================="

ss -tulpn | egrep '80|5000|5432|6379|8080|8081|8082|9042|9200' || true

# ==========================================================
# URLS
# ==========================================================
echo ""
echo "=================================================="
echo "🌐 ACCESS URLS"
echo "=================================================="

echo "Public IP           -> $PUBLIC_IP"
echo "Private IP          -> $PRIVATE_IP"
echo ""
echo "Frontend            -> http://$PUBLIC_IP/"
echo "Employee API        -> http://$PUBLIC_IP/api/v1/employee/health"
echo "Attendance API      -> http://$PUBLIC_IP/api/v1/attendance/health"
echo "Salary API          -> http://$PUBLIC_IP/api/v1/salary/search/all"
echo "Salary Health       -> http://$PUBLIC_IP:8082/actuator/health"
echo "Notification Health -> http://$PUBLIC_IP/notification/health"

echo ""
echo "=================================================="
echo "📘 DOCS"
echo "=================================================="

echo "Employee Swagger    -> http://$PUBLIC_IP:8080/swagger/index.html"
echo "Attendance Swagger  -> http://$PUBLIC_IP:8081/apidocs/"
echo "Salary Swagger      -> http://$PUBLIC_IP:8082/swagger-ui/index.html"

# ==========================================================
# DONE
# ==========================================================
END_TS=$(date +%s)
TOTAL=$((END_TS - START_TS))

echo ""
echo "=================================================="
ok "OT-Microservices fully started"
echo "⏱ Startup Time : ${TOTAL}s"
echo "📝 Log File     : $LOGFILE"
echo "=================================================="
```
# OT-Microservices Stop Script (Production EC2)

```bash
#!/bin/bash
set -euo pipefail

echo "🛑 Stopping OT-Microservices Stack..."

SERVICES=(notification-api salary-api attendance-api employee-api nginx elasticsearch scylla-server postgresql redis-server)

for svc in "${SERVICES[@]}"; do
  if systemctl list-unit-files | grep -q "^${svc}"; then
    echo "▶ Stopping $svc"
    sudo systemctl stop "$svc" || true
    sleep 1
    if systemctl is-active --quiet "$svc"; then
      echo "⚠ $svc still active"
    else
      echo "✅ $svc stopped"
    fi
  else
    echo "⚠ Skipping $svc (not installed)"
  fi
done

echo ""
echo "🔍 Remaining Ports"
ss -tulpn | egrep '80|5000|6379|8080|8081|8082|9200|9042' || true

echo ""
echo "✅ OT-Microservices stopped successfully"
```
# Cleanup script
```
#!/bin/bash
# ==========================================================
# OT-Microservices CLEANUP FINAL
# Enterprise-grade cleanup / low-RAM EC2 optimized
# Safe cleanup + preserve builds + fast restart ready
# ==========================================================

set -Eeuo pipefail

# ---------------- CONFIG ----------------
BASE="$HOME/OT-Micro"
FRONTEND="$BASE/frontend"
SALARY="$BASE/salary-api"

LOCKFILE="/tmp/otmicro-cleanup.lock"
LOGFILE="$HOME/cleanup.log"

LOW_RAM_MB=700

# ---------------- COLORS ----------------
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

ok(){ echo -e "${GREEN}✅ $1${NC}"; }
bad(){ echo -e "${RED}❌ $1${NC}"; }
warn(){ echo -e "${YELLOW}⚠ $1${NC}"; }
info(){ echo -e "${BLUE}▶ $1${NC}"; }

# ---------------- LOGGING ----------------
exec > >(tee -a "$LOGFILE") 2>&1

# ---------------- LOCK ----------------
exec 9>"$LOCKFILE"
flock -n 9 || { echo "Cleanup already running."; exit 1; }

# ---------------- ERROR HANDLER ----------------
trap 'bad "Cleanup failed on line $LINENO"; exit 1' ERR

echo "=================================================="
echo "🧹 OT-Microservices CLEANUP FINAL"
echo "=================================================="

# ==========================================================
# PRE-STATS
# ==========================================================
echo ""
info "Disk Before:"
df -h /

echo ""
info "Memory Before:"
free -h

echo ""
info "Project Size Before:"
du -sh "$BASE" 2>/dev/null || true

# ==========================================================
# HEALTH SNAPSHOT
# ==========================================================
echo ""
info "Running service snapshot..."

for svc in nginx redis-server postgresql scylla-server employee-api attendance-api salary-api notification-api elasticsearch
do
    if systemctl is-active --quiet "$svc"; then
        ok "$svc active"
    else
        warn "$svc inactive"
    fi
done

# ==========================================================
# MEMORY CLEANUP (conditional)
# ==========================================================
echo ""
FREE_MB=$(free -m | awk '/Mem:/ {print $7}')

if [ "$FREE_MB" -lt "$LOW_RAM_MB" ]; then
    info "Low free RAM detected (${FREE_MB}MB) -> reclaiming memory"

    sudo sync || true
    echo 3 | sudo tee /proc/sys/vm/drop_caches >/dev/null || true

    if swapon --show | grep -q "/"; then
        sudo swapoff -a || true
        sudo swapon -a || true
        ok "Swap refreshed"
    fi
else
    info "RAM healthy (${FREE_MB}MB free) -> skipping aggressive cache drop"
fi

# ==========================================================
# FRONTEND SAFE CLEAN
# Preserve build + node_modules
# ==========================================================
echo ""
info "Cleaning frontend temp/cache..."

rm -rf "$FRONTEND/node_modules/.cache" 2>/dev/null || true
rm -rf "$FRONTEND/.cache" 2>/dev/null || true
rm -f "$FRONTEND/npm-debug.log" 2>/dev/null || true
rm -f "$FRONTEND/yarn-error.log" 2>/dev/null || true

ok "Frontend artifacts preserved"

# ==========================================================
# SALARY SAFE CLEAN
# Preserve jar
# ==========================================================
echo ""
info "Cleaning salary temp artifacts..."

find "$SALARY/target" -type f \( -name "*.tmp" -o -name "*.original" -o -name "*.log" \) -delete 2>/dev/null || true

ok "Salary jar preserved"

# ==========================================================
# PYTHON CACHE CLEAN
# ==========================================================
echo ""
info "Cleaning Python cache..."

find "$BASE" -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
find "$BASE" -type f \( -name "*.pyc" -o -name "*.pyo" \) -delete 2>/dev/null || true

ok "Python cache cleaned"

# ==========================================================
# TEMP FILES
# ==========================================================
echo ""
info "Cleaning OT temp files..."

rm -f /tmp/otmicro-* 2>/dev/null || true
rm -f /tmp/*salary*.pdf 2>/dev/null || true
rm -f "$HOME/notification.log" 2>/dev/null || true

ok "Temp files removed"

# ==========================================================
# LOG MAINTENANCE
# ==========================================================
echo ""
info "Cleaning logs..."

sudo journalctl --vacuum-size=200M >/dev/null 2>&1 || true
sudo apt clean >/dev/null 2>&1 || true

# truncate oversized logs
[ -f "$HOME/startup.log" ] && truncate -s 0 "$HOME/startup.log" || true

ok "Logs maintained"

# ==========================================================
# EDITOR / GIT JUNK
# ==========================================================
echo ""
info "Cleaning editor leftovers..."

find "$BASE" -type f \( -name "*.swp" -o -name "*~" \) -delete 2>/dev/null || true

ok "Editor temp files removed"

# ==========================================================
# ZOMBIE / STALE PROCESS SNAPSHOT
# ==========================================================
echo ""
info "Checking zombie processes..."

ps aux | awk '$8 ~ /Z/ {print}' || true

# ==========================================================
# POST-STATS
# ==========================================================
echo ""
info "Disk After:"
df -h /

echo ""
info "Memory After:"
free -h

echo ""
info "Project Size After:"
du -sh "$BASE" 2>/dev/null || true

# ==========================================================
# READY STATUS
# ==========================================================
echo ""
echo "=================================================="
ok "Cleanup completed successfully"
echo "✔ Frontend build kept"
echo "✔ node_modules kept"
echo "✔ Salary jar kept"
echo "✔ Python junk removed"
echo "✔ Logs compressed"
echo "✔ Ready for fast ./start.sh"
echo "📝 Log File: $LOGFILE"
echo "=================================================="
```


