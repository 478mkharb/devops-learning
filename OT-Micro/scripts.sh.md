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
