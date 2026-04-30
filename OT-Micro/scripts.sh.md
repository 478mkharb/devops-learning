# OT-Microservices Start Script (Production EC2)
```bash
# ==========================================================
# PATCHED start.sh  (Production Safe)
# Auto rebuild salary-api jar if missing
# Smart waits
# ==========================================================

#!/bin/bash
set +e

GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

ok(){ echo -e "${GREEN}✅ $1${NC}"; }
bad(){ echo -e "${RED}❌ $1${NC}"; }
warn(){ echo -e "${YELLOW}⚠ $1${NC}"; }

PUBLIC_IP=$(curl -s ifconfig.me 2>/dev/null)
[ -z "$PUBLIC_IP" ] && PUBLIC_IP=$(hostname -I | awk '{print $1}')

echo "=================================================="
echo "🚀 Starting OT-Microservices Stack"
echo "=================================================="

# Infra first
for svc in redis-server postgresql scylla-server elasticsearch nginx
do
  sudo systemctl start $svc
  sleep 2
done

# Employee + Attendance
sudo systemctl start employee-api
sudo systemctl start attendance-api

# ------------------------------------------------
# Salary API jar auto-check
# ------------------------------------------------
JAR=~/OT-Micro/salary-api/target/salary-0.1.0-RELEASE.jar

if [ ! -f "$JAR" ]; then
    warn "Salary JAR missing. Rebuilding..."
    cd ~/OT-Micro/salary-api
    mvn clean package -DskipTests
fi

sudo systemctl start salary-api

# Notification
sudo systemctl start notification-api

echo ""
echo "=================================================="
echo "🔍 HEALTH CHECKS"
echo "=================================================="

curl -fsS http://localhost/api/v1/employee/health >/dev/null \
&& ok "Employee API OK" || bad "Employee API DOWN"

curl -fsS http://localhost/api/v1/attendance/health >/dev/null \
&& ok "Attendance API OK" || bad "Attendance API DOWN"

echo ""
echo "⏳ Waiting for Salary API (max 60 sec)..."

SALARY_OK=false
for i in {1..60}
do
    if curl -fsS http://localhost:8082/actuator/health >/dev/null 2>&1; then
        SALARY_OK=true
        break
    fi
    sleep 1
done

if [ "$SALARY_OK" = true ]; then
    ok "Salary API OK"
else
    bad "Salary API DOWN after 60 sec"
fi

echo ""
echo "⏳ Waiting for Notification API (max 20 sec)..."

NOTIFY_OK=false
for i in {1..20}
do
    if curl -fsS http://localhost:5000/health >/dev/null 2>&1; then
        NOTIFY_OK=true
        break
    fi
    sleep 1
done

if [ "$NOTIFY_OK" = true ]; then
    ok "Notification API OK"
else
    bad "Notification API DOWN"
fi

redis-cli ping >/dev/null && ok "Redis OK" || bad "Redis DOWN"
curl -fsS localhost:9200 >/dev/null && ok "Elasticsearch OK" || bad "Elasticsearch DOWN"

echo ""
echo "=================================================="
echo "🌐 ACCESS URLS"
echo "=================================================="

echo "Frontend            -> http://$PUBLIC_IP/"
echo "Employee API        -> http://$PUBLIC_IP/api/v1/employee/health"
echo "Attendance API      -> http://$PUBLIC_IP/api/v1/attendance/health"
echo "Salary API          -> http://$PUBLIC_IP/api/v1/salary/search/all"
echo "Salary Health       -> http://$PUBLIC_IP:8082/actuator/health"
echo "Notification Health -> http://$PUBLIC_IP/notification/health"
echo ""
echo "=================================================="
echo "📘 SWAGGER / API DOCS"
echo "=================================================="

echo "Employee Swagger    -> http://$PUBLIC_IP:8080/swagger/index.html"
echo "Attendance Swagger  -> http://$PUBLIC_IP:8081/apidocs/"
echo "Salary Swagger      -> http://$PUBLIC_IP:8082/swagger-ui/index.html"
echo ""
ok "Startup completed"
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
# ==========================================================
# PATCHED cleanup.sh  (Production Safe)
# Keeps required salary-api jar
# ==========================================================

#!/bin/bash
set +e

echo "=================================================="
echo "🧹 OT-Microservices Safe Cleanup"
echo "=================================================="

echo "▶ Memory Before:"
free -h
echo ""

echo "▶ Disk Before:"
df -h /
echo ""

sudo sync
echo 3 | sudo tee /proc/sys/vm/drop_caches >/dev/null

# Frontend safe cleanup
rm -rf ~/OT-Micro/frontend/build
rm -rf ~/OT-Micro/frontend/node_modules/.cache

# Salary API safe cleanup (preserve real jar)
find ~/OT-Micro/salary-api/target -type f -name "*.original" -delete 2>/dev/null
find ~/OT-Micro/salary-api/target -type f -name "*.tmp" -delete 2>/dev/null
find ~/OT-Micro/salary-api/target -type f -name "*.log" -delete 2>/dev/null

# Python cache cleanup
find ~/OT-Micro -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find ~/OT-Micro -type f -name "*.pyc" -delete 2>/dev/null

# Temp files
rm -f /tmp/*salary*.pdf
rm -f ~/notification.log

# Logs cleanup
sudo journalctl --vacuum-time=7d >/dev/null 2>&1
sudo apt clean >/dev/null 2>&1

echo ""
echo "▶ Memory After:"
free -h
echo ""

echo "▶ Disk After:"
df -h /
echo ""

echo "▶ Project Size:"
du -sh ~/OT-Micro

echo ""
echo "✅ Cleanup complete"
echo "Now run ./start.sh"
echo "=================================================="
```
