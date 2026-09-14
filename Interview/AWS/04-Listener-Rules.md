# Application Load Balancer Listeners & Listener Rules — Interview Notes

---

## 1. ALB Listeners

### Q1. What is an ALB listener?

**Answer:** An ALB listener checks incoming connections on a configured protocol and port, such as `HTTP:80` or `HTTPS:443`. It evaluates listener rules to determine how matching requests are handled.

### Q2. What protocols can an ALB listener use?

**Answer:** ALB listeners support:

- **HTTP** — commonly configured on port `80`
- **HTTPS** — commonly configured on port `443` and protected by TLS

An HTTPS listener requires a certificate, commonly provided through AWS Certificate Manager (ACM).

### Q3. What is the difference between HTTP and HTTPS listeners?

| HTTP listener | HTTPS listener |
|---|---|
| Uses HTTP | Uses HTTP over TLS |
| Commonly uses port `80` | Commonly uses port `443` |
| Client traffic is unencrypted | Client traffic is encrypted in transit |
| Does not require a TLS certificate | Requires a certificate |
| Commonly redirects clients to HTTPS | Can terminate TLS at the ALB |

### Q4. What is a listener port?

**Answer:** The listener port is the port on which the ALB accepts client connections.

Examples:

```text
HTTP  → 80
HTTPS → 443
```

### Q5. Can one ALB have multiple listeners?

**Answer:** Yes. An ALB can have multiple listeners, commonly one HTTP listener and one HTTPS listener. Each listener has its own protocol, port, default action, and listener rules.

### Q6. Why is HTTPS commonly terminated at the ALB?

**Answer:** TLS termination at the ALB:

- Centralizes certificate management
- Simplifies certificate renewal
- Removes TLS processing from backend applications
- Allows listener rules to inspect HTTP requests after TLS termination

The ALB can forward traffic to targets using HTTP or HTTPS, depending on the configured backend protocol.

---

## 2. Listener Actions

### Q7. What is a listener action?

**Answer:** A listener action defines what the ALB does when a request matches a listener rule.

Common actions include:

| Action | Meaning | Example |
|---|---|---|
| **Forward** | Sends the request to a target group | `/api/*` → API target group |
| **Redirect** | Sends the client to another URL | HTTP → HTTPS |
| **Fixed response** | Returns a response directly from the ALB | Return `503` during maintenance |

### Q8. What is a default listener action?

**Answer:** The default listener action is the fallback action for a request that does not match any custom rule. It commonly forwards the request to a target group, but it can also redirect or return a fixed response.

### Q9. What is a forward action?

**Answer:** A forward action sends a request to one or more target groups. The target group receives the request according to the forwarding configuration.

Example:

```text
/api/orders → API target group
```

### Q10. What is a redirect action?

**Answer:** A redirect action instructs the client to make another request to a specified URL.

Common example:

```text
HTTP :80 → HTTPS :443
```

Example redirect configuration:

```text
Protocol: HTTPS
Port: 443
Status code: HTTP_301
```

### Q11. What is a fixed-response action?

**Answer:** A fixed-response action returns a configured HTTP response directly from the ALB without forwarding the request to a target.

Example:

```text
Status code: 503
Content type: text/plain
Message body: Service temporarily unavailable
```

---

## 3. Listener Rules

### Q12. What is a listener rule?

**Answer:** An ALB listener rule contains:

1. Conditions
2. A priority
3. One or more actions

When a request satisfies the conditions, the ALB executes the rule's action.

### Q13. What is rule priority?

**Answer:** Rule priority determines the order in which custom listener rules are evaluated.

- Lower numerical priority is evaluated first.
- The first matching rule is used.
- The default rule is evaluated if no custom rule matches.

Example:

| Priority | Condition | Action |
|---:|---|---|
| `1` | Host is `api.example.com` | Forward to API target group |
| `10` | Path is `/admin/*` | Forward to admin target group |
| `20` | Path is `/web/*` | Forward to web target group |
| Default | No custom rule matches | Forward to default target group |

### Q14. What is the default rule?

**Answer:** The default rule is the final fallback rule for a listener. It is evaluated when no custom listener rule matches.

The default rule does not have a normal custom priority.

### Q15. What conditions can listener rules evaluate?

Supported listener-rule conditions include:

- Host-header
- Path-pattern
- HTTP-header
- HTTP-request-method
- Query-string
- Source-IP

The ALB evaluates the request against the conditions and executes the action when the rule matches.

### Q16. Can a rule have multiple conditions?

**Answer:** Yes. A rule can combine conditions, such as:

```text
Host header: api.example.com
AND
Path pattern: /v1/*
```

The request must satisfy the rule's conditions for the rule to match.

### Q17. Can a rule contain multiple actions?

**Answer:** Yes. A rule can contain supported actions in an ordered sequence. The final action must be a routing action such as forwarding, redirecting, or returning a fixed response.

---

## 4. Host-Based Routing

### Q18. Explain host-header routing.

**Answer:** Host-header routing examines the HTTP `Host` header and routes different domain names to different target groups.

Example:

| Host header | Action |
|---|---|
| `api.example.com` | Forward to API target group |
| `app.example.com` | Forward to web target group |
| `admin.example.com` | Forward to admin target group |

### Q19. How does host-based routing support multiple applications?

**Answer:** One ALB can serve multiple applications by using different host-header conditions.

Example:

```text
api.example.com   → API application
app.example.com   → Frontend application
admin.example.com → Admin application
```

This avoids requiring a separate ALB for every application.

---

## 5. Path-Based Routing

### Q20. Explain path-pattern routing.

**Answer:** Path-pattern routing examines the URL path and forwards requests to different target groups.

Example:

| Path pattern | Action |
|---|---|
| `/api/*` | Forward to API target group |
| `/web/*` | Forward to web target group |
| `/admin/*` | Forward to admin target group |

### Q21. How would you route `/api/*` to one target group and `/web/*` to another?

**Answer:** Create two listener rules:

```text
Priority 10:
  Condition: Path is /api/*
  Action: Forward to API target group

Priority 20:
  Condition: Path is /web/*
  Action: Forward to web target group
```

The default rule handles all requests that match neither pattern.

### Q22. Can host-based and path-based routing be combined?

**Answer:** Yes.

Example:

```text
Host: api.example.com
Path: /v1/*
Action: Forward to API v1 target group
```

This allows routing based on both the requested domain and URL path.

---

## 6. Listener Rule Examples

### Example 1: HTTP to HTTPS redirect

```text
Listener: HTTP :80
Default action: Redirect to HTTPS :443
Status code: HTTP_301
```

Flow:

```text
Client
  |
  v
HTTP :80 listener
  |
  v
Redirect
  |
  v
HTTPS :443 listener
```

### Example 2: Host-based routing

```text
Rule 1:
  Host: api.example.com
  Action: Forward to API target group

Rule 2:
  Host: app.example.com
  Action: Forward to frontend target group
```

### Example 3: Path-based routing

```text
Rule 1:
  Path: /api/*
  Action: Forward to API target group

Rule 2:
  Path: /web/*
  Action: Forward to frontend target group
```

### Example 4: Maintenance response

```text
Condition: Host is maintenance.example.com
Action: Fixed response 503
```

---

## 7. Listener Rule Evaluation Flow

```text
Incoming request
      |
      v
Listener protocol and port match?
      |
      v
Evaluate custom rules by priority
      |
      v
Does a rule match?
   /          \
 Yes          No
  |            |
  v            v
Execute      Execute
rule action  default action
```

### Q23. What happens if no custom rule matches?

**Answer:** The ALB executes the default listener rule and its configured default action.

### Q24. What happens if two rules could match the same request?

**Answer:** The rule with the lowest numerical priority is evaluated first. If it matches, its action is executed and later rules are not evaluated for that request.

### Q25. Why should listener-rule priorities be designed carefully?

**Answer:** Overlapping conditions can cause an earlier rule to capture traffic before a more specific rule is reached. Specific rules should generally be assigned higher precedence than broad fallback rules.

---

## 8. Listener Rules vs Other AWS Features

### Q26. How do listener rules differ from Route 53 routing?

| Listener rules | Route 53 routing |
|---|---|
| Operate at the ALB request layer | Operates at the DNS layer |
| Inspect HTTP request attributes | Returns DNS answers |
| Match hosts, paths, headers, methods, and other conditions | Uses DNS routing policies |
| Forward, redirect, or return responses | Directs clients toward DNS endpoints |

### Q27. How do listener rules differ from security groups?

| Listener rules | Security groups |
|---|---|
| Decide how accepted requests are handled | Control allowed network traffic |
| Use HTTP request conditions | Use traffic rules such as protocol, port, and source |
| Forward, redirect, or return responses | Allow or deny traffic at the network interface level |
| Do not replace network access controls | Do not perform host/path routing |

---

## 9. Common Troubleshooting Scenarios

### Q28. A request reaches the ALB but goes to the wrong application. What should you check?

Check:

- Listener protocol and port
- Host-header condition
- Path-pattern condition
- Rule priority
- Whether an earlier broad rule matches first
- Default listener action
- Correct target group associated with the action

### Q29. HTTP requests are not redirected to HTTPS. What should you check?

Check:

- An HTTP listener exists on port `80`
- Its default action is a redirect
- Redirect protocol is `HTTPS`
- Redirect port is `443`
- HTTPS listener exists
- The HTTPS listener has a valid certificate

### Q30. A listener rule returns a fixed response instead of forwarding traffic. Why?

Possible causes include:

- The request matched a fixed-response rule
- A higher-priority rule matched before the intended forwarding rule
- The default action is configured as a fixed response
- The rule conditions are broader than expected

---

## 10. Quick Revision

| Question | Short answer |
|---|---|
| What is a listener? | Accepts connections on a protocol and port |
| Common listener protocols? | HTTP and HTTPS |
| Common listener ports? | `80` and `443` |
| What is a listener rule? | Conditions plus priority and action |
| How are rules evaluated? | Lowest numerical priority first |
| What is the default rule? | Final fallback rule |
| Main routing conditions? | Host, path, headers, method, query string, source IP |
| Main actions? | Forward, redirect, fixed response |
| Host-based routing? | Routes by HTTP Host header |
| Path-based routing? | Routes by URL path |
| HTTP to HTTPS? | Redirect from HTTP listener to HTTPS listener |
| Do listener rules replace security groups? | No |

---

## 11. Interview Checkpoints

- Define an ALB listener.
- Compare HTTP and HTTPS listeners.
- Explain listener ports.
- Explain TLS termination at the ALB.
- Define a listener rule.
- Explain rule priority.
- Explain the default listener rule.
- List supported rule conditions.
- Explain host-based routing.
- Explain path-based routing.
- Combine host and path conditions.
- Compare forward, redirect, and fixed-response actions.
- Explain HTTP-to-HTTPS redirection.
- Explain listener rules versus Route 53 routing.
- Explain listener rules versus security groups.
- Troubleshoot incorrect rule matching.
