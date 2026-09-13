# Linux for DevOps — Topic 10: Networking for DevOps

## Scope

This topic covers practical Linux networking for DevOps engineers working with EC2, private/public subnets, NGINX, Jenkins, Ansible, SSM, APIs, exporters, load balancers, security groups, and firewalls.

> **DevOps principle:** Troubleshoot layer by layer: DNS, route, socket, firewall, cloud security controls, TCP, TLS, proxy, and application.

## 1. Core networking terms

| Term | Meaning |
|---|---|
| IP address | Identifies a network endpoint/interface |
| Port | Identifies a service on a host |
| Protocol | Communication rules such as TCP or UDP |
| Socket | Endpoint commonly represented by IP + port + protocol |
| Listening socket | Process waiting for connections |

Example:

```text
10.0.2.15:8080
```

## 2. Essential commands

```bash
ip addr
ip -br addr
ip link
ip route
ip route get 8.8.8.8
sudo ss -lntup
sudo lsof -i -P -n
ping -c 4 10.0.2.15
nc -vz 10.0.2.15 8080
curl -v http://10.0.2.15:8080/health
tracepath 8.8.8.8
```

Useful `ss` flags:

```bash
ss -l   # listening
ss -n   # numeric
ss -t   # TCP
ss -u   # UDP
ss -p   # process
ss -a   # all
```

## 3. TCP vs UDP

| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable and ordered | No built-in delivery guarantee |
| Retransmits lost data | Datagram-based |
| HTTP/HTTPS, SSH, PostgreSQL | DNS and streaming use cases |

## 4. Private and public IPs

Private ranges:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Typical AWS design:

```text
Internet -> Public Load Balancer -> Private EC2 -> Private Database
```

A public IP does not guarantee reachability. Also check:

- Route tables
- Security groups
- Network ACLs
- Host firewall
- Listening address
- Correct port
- Load-balancer target health

## 5. Listening address: localhost vs all interfaces

A service bound to:

```text
127.0.0.1:8080
```

is reachable only from the same host.

A service bound to:

```text
0.0.0.0:8080
```

can accept connections through configured IPv4 interfaces, subject to firewall rules.

Check:

```bash
sudo ss -lntp | grep 8080
```

Common error:

```text
Frontend -> 127.0.0.1:8080
```

If the API is on another EC2 instance, `127.0.0.1` points to the frontend host. Use the API's private DNS name or private IP.

## 6. Routing

Inspect routes:

```bash
ip route
ip route get 10.0.3.25
```

Typical output:

```text
default via 10.0.1.1 dev eth0
10.0.0.0/16 dev eth0 proto kernel scope link src 10.0.1.20
```

Common route failures:

- Missing default route
- Wrong subnet route
- Incorrect AWS route-table association
- Private subnet without NAT for internet access
- Missing VPC peering/TGW route
- Asymmetric routing

## 7. DNS troubleshooting

```bash
getent hosts example.com
resolvectl query example.com
dig example.com
cat /etc/resolv.conf
resolvectl status
dig A example.com
dig AAAA example.com
dig CNAME app.example.com
```

Test a specific resolver:

```bash
dig @8.8.8.8 example.com
```

Distinguish failures:

- DNS failure: hostname cannot resolve.
- TCP failure: IP resolves but port is unreachable.
- TLS failure: TCP works but handshake/certificate fails.
- HTTP failure: application returns an error.

## 8. `/etc/hosts` and DNS

```bash
getent hosts employee-api
cat /etc/hosts
cat /etc/nsswitch.conf
```

Use DNS/service discovery for dynamic infrastructure. Avoid maintaining large manually edited hosts files.

## 9. HTTP troubleshooting with curl

```bash
curl -v http://localhost:8080/health
curl -I http://localhost:8080/health
curl -L https://example.com
curl --connect-timeout 3 --max-time 10 -fsS http://localhost:8080/health
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/health
```

JSON request:

```bash
curl -fsS   -H 'Content-Type: application/json'   -d '{"name":"Mukesh"}'   http://localhost:8080/api/employees
```

## 10. Important HTTP status codes

| Code | Meaning | Investigation |
|---|---|---|
| 200 | Success | Request completed |
| 301/302 | Redirect | URL/proxy rules |
| 400 | Bad request | Payload/headers |
| 401 | Unauthorized | Credentials/token |
| 403 | Forbidden | Authorization/WAF |
| 404 | Not found | Route/context path |
| 429 | Too many requests | Rate limiting |
| 500 | Application error | Application logs |
| 502 | Bad gateway | Upstream unavailable/invalid |
| 503 | Unavailable | Backend unhealthy/overloaded |
| 504 | Gateway timeout | Upstream too slow |

## 11. NGINX reverse proxy

```text
Client -> NGINX:80/443 -> Application:8080
```

Commands:

```bash
sudo nginx -t
sudo systemctl status nginx
sudo ss -lntp | grep nginx
curl -v http://127.0.0.1:8080/health
curl -v http://127.0.0.1/health
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

Common 502 causes:

- Backend stopped
- Wrong upstream port
- Backend bound to localhost on another host
- Firewall blocks proxy-to-backend traffic
- Unix socket permission issue
- Backend crashes during request

## 12. Host firewall

UFW:

```bash
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow from 10.0.0.0/16 to any port 8080 proto tcp
```

Be careful when enabling UFW remotely; allow SSH first.

Other inspection commands:

```bash
sudo nft list ruleset
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v
```

## 13. AWS networking checks

For EC2, verify:

```text
Application
 -> Listening socket
 -> Linux firewall
 -> ENI
 -> Security group
 -> Network ACL
 -> Route table
 -> Gateway/NAT/LB/Peering
 -> Destination
```

Private EC2 outbound internet path:

```text
Private EC2 -> Route Table -> NAT Gateway -> Internet Gateway -> Internet
```

SSM commonly requires:

- Correct instance IAM role
- DNS resolution
- HTTPS access to SSM endpoints
- NAT Gateway or VPC interface endpoints
- Correct AWS region
- Running SSM agent

## 14. Security groups vs NACLs

| Security Group | Network ACL |
|---|---|
| Attached to ENI/instance | Attached to subnet |
| Stateful | Stateless |
| Allow rules | Allow and deny rules |
| Return traffic automatically handled | Return traffic must be allowed |
| Common instance-level check | Subnet-level filtering |

## 15. SSH troubleshooting

```bash
ssh -vvv ubuntu@10.0.2.15
nc -vz 10.0.2.15 22
ip route get 10.0.2.15
```

On the destination:

```bash
sudo ss -lntp | grep ':22'
sudo systemctl status ssh
sudo journalctl -u ssh -n 100 --no-pager
sudo ufw status
```

Common timeout causes:

- Wrong private IP
- Missing route
- Security group rule missing
- NACL blocks traffic or ephemeral return ports
- Host firewall
- SSH daemon stopped
- Bastion cannot reach private subnet

Difference:

- **Timeout:** often filtering, routing, or path problem.
- **Connection refused:** host reached, but no process accepted or connection was actively rejected.

## 16. Ephemeral ports

Example:

```text
10.0.1.20:49152 -> 10.0.2.15:8080
```

The client uses a temporary source port. Return traffic must be allowed, especially with stateless NACLs and firewalls.

## 17. Packet capture

```bash
sudo tcpdump -i eth0 -nn
sudo tcpdump -i eth0 -nn port 8080
sudo tcpdump -i eth0 -nn host 10.0.2.15
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-syn != 0'
sudo tcpdump -i eth0 -nn -w /tmp/capture.pcap port 8080
tcpdump -nn -r /tmp/capture.pcap
```

Interpretation:

- SYN with no response: route/filtering issue likely.
- SYN, SYN-ACK, ACK: TCP handshake completed.
- RST: active rejection/reset.
- Retransmissions: packet loss or filtering.
- TCP works but HTTP fails: investigate TLS/application.

## 18. Monitoring networking

Typical flow:

```text
Prometheus -> Node Exporter:9100
Prometheus -> Blackbox Exporter:9115
Grafana -> Prometheus:9090
```

Check:

```bash
sudo ss -lntp | grep -E '9100|9115|9090'
curl -fsS http://localhost:9100/metrics | head
curl -fsS http://localhost:9115/metrics | head
```

An exporter can work locally but fail remotely because of bind address, security groups, host firewall, wrong target, or missing route.

## 19. Repeatable troubleshooting workflow

```text
1. Confirm hostname and port.
2. Resolve hostname.
3. Check route.
4. Confirm destination listener.
5. Test locally on destination.
6. Test TCP from client.
7. Check host firewall.
8. Check AWS security groups and NACLs.
9. Check load-balancer target health.
10. Inspect proxy/application logs.
11. Capture packets if needed.
```

Useful sequence:

```bash
getent hosts SERVICE
ip route get DESTINATION_IP
nc -vz SERVICE PORT
curl -v http://SERVICE:PORT/health
sudo ss -lntup
sudo ufw status verbose
sudo tcpdump -i any -nn host DESTINATION_IP
```

## 20. Real-world scenarios

### Scenario A: Local port works, remote port fails

```bash
sudo ss -lntp | grep 8080
curl http://127.0.0.1:8080/health
```

If listening on `127.0.0.1`, change the bind address or use a local reverse proxy. Then check firewall, security group, NACL, and routes.

### Scenario B: NGINX returns 502

```bash
sudo nginx -t
curl -v http://127.0.0.1:8080/health
sudo journalctl -u myapp -n 100 --no-pager
sudo tail -n 100 /var/log/nginx/error.log
```

### Scenario C: Bastion SSH timeout

```bash
ip route get 10.0.2.15
nc -vz 10.0.2.15 22
ssh -vvv ubuntu@10.0.2.15
```

Check bastion outbound rules, target inbound rule, routes, NACLs, target IP, and SSH daemon.

### Scenario D: SSM offline

```bash
sudo systemctl status amazon-ssm-agent
sudo journalctl -u amazon-ssm-agent -n 100 --no-pager
curl -I https://ssm.us-east-1.amazonaws.com
```

Verify IAM role, DNS, NAT/VPC endpoints, HTTPS egress, region, and agent status.

### Scenario E: DNS works but API fails

```bash
getent hosts api.internal
nc -vz api.internal 8080
curl -v http://api.internal:8080/health
```

This separates DNS, TCP, and HTTP problems.

## 21. Interview questions

1. What is the difference between IP, port, socket, and protocol?
2. Explain TCP vs UDP.
3. What does `0.0.0.0` mean?
4. Why is `127.0.0.1` unreachable from another host?
5. Difference between timeout and connection refused?
6. How do you identify the process listening on port 8080?
7. How do you test a TCP port?
8. How do you troubleshoot an NGINX 502?
9. What does `ip route get` show?
10. How does a private EC2 instance access the internet?
11. Compare security groups and NACLs.
12. Why are ephemeral ports important?
13. How do you troubleshoot SSH through a bastion?
14. How do you verify DNS?
15. How do you distinguish DNS, TCP, TLS, and HTTP failures?
16. How do you use tcpdump?
17. Why can ping fail while HTTP works?
18. Why can an exporter work locally but fail in Prometheus?
19. What networking does SSM require?
20. How would you troubleshoot an API timeout between EC2 instances?

## 22. Interview checklist

- [ ] Explain IP, port, socket, TCP, and UDP.
- [ ] Use `ip addr`, `ip route`, and `ss`.
- [ ] Test ports with `nc`.
- [ ] Test APIs with `curl`.
- [ ] Troubleshoot DNS with `getent`, `dig`, and `resolvectl`.
- [ ] Explain localhost vs `0.0.0.0`.
- [ ] Understand private/public subnet routing.
- [ ] Explain security groups and NACLs.
- [ ] Troubleshoot SSH through a bastion.
- [ ] Understand ephemeral ports.
- [ ] Use tcpdump.
- [ ] Troubleshoot NGINX upstream failures.
- [ ] Troubleshoot exporter reachability.
- [ ] Explain SSM network dependencies.

## Key DevOps principle

> **Do not say “the network is down” without evidence. Identify the exact failing layer: DNS, route, TCP handshake, firewall, TLS, proxy, or application.**
