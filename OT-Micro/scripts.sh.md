# OT-Microservices Start Script (Production EC2)

```bash
#!/bin/bash
set -euo pipefail

echo "🚀 Starting OT-Microservices Stack..."

PUBLIC_IP=$(curl -s ifconfig.me || hostname -I | awk '{print $1}')
LOG_DIR=~/logs
mkdir -p "$LOG_DIR"

SERVICES=(redis-server postgresql scylla-server elasticsearch nginx employee-api attendance-api salary-api notification-api)

wait_service() {
  local svc=$1
  for i in {1..20}; do
    if systemctl is-active --quiet "$svc"; then
      echo "✅ $svc is active"
      return 0
    fi
    sleep 2
  done
  echo "❌ $svc failed to become active"
  return 1
}

for svc in "${SERVICES[@]}"; do
  if systemctl list-unit-files | grep -q "^${svc}"; then
    echo "▶ Starting $svc"
    sudo systemctl start "$svc" || true
    wait_service "$svc" || true
  else
    echo "⚠ Skipping $svc (not installed)"
  fi
done

echo ""
echo "🔍 Quick Health Checks"
curl -fsS http://localhost/api/v1/employee/health && echo "  Employee OK" || echo "  Employee FAIL"
curl -fsS http://localhost/api/v1/attendance/health && echo "  Attendance OK" || echo "  Attendance FAIL"
curl -fsS http://localhost:8082/actuator/health && echo "  Salary OK" || echo "  Salary FAIL"
curl -fsS http://localhost/notification/health && echo "  Notification OK" || echo "  Notification FAIL"
redis-cli ping && echo "  Redis OK" || echo "  Redis FAIL"
curl -fsS localhost:9200 >/dev/null && echo "  Elasticsearch OK" || echo "  Elasticsearch FAIL"

echo ""
echo "🌐 ACCESS URLS"
echo "Frontend            -> http://$PUBLIC_IP/"
echo "Employee API        -> http://$PUBLIC_IP/api/v1/employee/health"
echo "Attendance API      -> http://$PUBLIC_IP/api/v1/attendance/health"
echo "Salary API          -> http://$PUBLIC_IP/api/v1/salary/search/all"
echo "Salary Health       -> http://$PUBLIC_IP:8082/actuator/health"
echo "Notification Health -> http://$PUBLIC_IP/notification/health"

echo ""
echo "📘 Swagger URLs"
echo "Employee Swagger    -> http://$PUBLIC_IP:8080/swagger/index.html"
echo "Attendance Swagger  -> http://$PUBLIC_IP:8081/apidocs/"
echo "Salary Swagger      -> http://$PUBLIC_IP:8082/swagger-ui/index.html"

echo ""
echo "✅ OT-Microservices started successfully"
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
# OT-Microservices Full Cleanup Script
# File: cleanup.sh
# Usage:
#   chmod +x cleanup.sh
#   ./cleanup.sh
#
# Safe cleanup before ./start.sh
# Combines:
#   Memory cleanup
#   Old build cleanup
#   Logs cleanup
#   Temp files cleanup
# ==========================================================

set +e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

ok()   { echo -e "${GREEN}✅ $1${NC}"; }
warn() { echo -e "${YELLOW}⚠ $1${NC}"; }
info() { echo -e "${BLUE}▶ $1${NC}"; }

echo "=================================================="
echo "🧹 OT-Microservices Full Safe Cleanup"
echo "=================================================="

# --------------------------------------------------
# Initial Usage
# --------------------------------------------------
info "Disk usage before cleanup:"
df -h /
echo ""

info "Memory before cleanup:"
free -h
echo ""

# --------------------------------------------------
# Sync + Drop Cache
# --------------------------------------------------
info "Syncing filesystem buffers..."
sudo sync

info "Dropping Linux cache..."
echo 3 | sudo tee /proc/sys/vm/drop_caches >/dev/null
ok "Kernel cache cleared"

# --------------------------------------------------
# Refresh Swap
# --------------------------------------------------
if swapon --show | grep -q "/"; then
    info "Refreshing swap..."
    sudo swapoff -a
    sudo swapon -a
    ok "Swap refreshed"
else
    warn "No swap configured"
fi

# --------------------------------------------------
# Frontend Cleanup
# --------------------------------------------------
info "Cleaning frontend build/cache..."
rm -rf ~/OT-Micro/frontend/build
rm -rf ~/OT-Micro/frontend/node_modules/.cache
ok "Frontend cache removed"

# --------------------------------------------------
# Salary API Cleanup
# --------------------------------------------------
info "Cleaning Salary API old target builds..."
rm -rf ~/OT-Micro/salary-api/target/*
ok "Old JAR/build artifacts removed"

# --------------------------------------------------
# Python Cache Cleanup
# --------------------------------------------------
info "Cleaning Python caches..."
find ~/OT-Micro -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find ~/OT-Micro -type f -name "*.pyc" -delete 2>/dev/null
ok "Python cache removed"

# --------------------------------------------------
# Temp Files
# --------------------------------------------------
info "Cleaning temp PDFs..."
rm -f /tmp/*salary*.pdf
ok "Temp PDFs removed"

# --------------------------------------------------
# Logs Cleanup
# --------------------------------------------------
info "Cleaning old logs..."
rm -f ~/notification.log
sudo journalctl --vacuum-time=7d >/dev/null 2>&1
ok "Logs cleaned"

# --------------------------------------------------
# Apt Cache
# --------------------------------------------------
info "Cleaning apt cache..."
sudo apt clean >/dev/null 2>&1
ok "APT cache cleaned"

# --------------------------------------------------
# Final Usage
# --------------------------------------------------
echo ""
info "Disk usage after cleanup:"
df -h /
echo ""

info "Memory after cleanup:"
free -h
echo ""

info "Project size:"
du -sh ~/OT-Micro 2>/dev/null

echo ""
echo "=================================================="
ok "Cleanup completed successfully"
echo "Now run: ./start.sh"
echo "=================================================="

```
