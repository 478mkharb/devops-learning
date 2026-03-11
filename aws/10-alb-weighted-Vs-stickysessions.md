# Difference Between Weighted Routing and Sticky Sessions in AWS Application Load Balancer (ALB)

---

# 1. Weighted Routing

Weighted routing distributes incoming traffic across **multiple target groups** based on configured weights.

The load balancer forwards requests **proportionally** according to the assigned weight values.

## Example

| Target Group | Weight | Traffic Approximation |
| ------------ | ------ | --------------------- |
| TG-A         | 80     | ~80%                  |
| TG-B         | 20     | ~20%                  |

This means most requests go to **TG-A**, while fewer go to **TG-B**.

---

## Clear Architecture Diagram

```
              Users
                │
                ▼
        Application Load Balancer
                │
                │  Listener Rule
                │  Forward Action
                ▼
        ┌───────────────┐
        │               │               
        ▼               ▼               
   Target Group A   Target Group B
      Weight 80        Weight 20
        │                 │
   ┌────┴────┐       ┌────┴────┐
   ▼         ▼       ▼         ▼
EC2-A1    EC2-A2   EC2-B1    EC2-B2
```

Traffic flow example:

```
100 Requests
     │
     ├── 80 → Target Group A
     └── 20 → Target Group B
```

---

## Typical Use Cases

### Canary Deployment

Gradually shift traffic to new version.

```
Old Version → 90%
New Version → 10%
```

### Blue-Green Deployment

```
Blue Environment → 100%
Green Environment → 0%
```

Switch later:

```
Blue → 0%
Green → 100%
```

---

# 2. Sticky Sessions (Session Affinity)

Sticky sessions ensure that **a user continues to interact with the same backend server** for the duration of a session.

ALB achieves this using **cookies**.

When a request first arrives, ALB selects a backend server and sends a cookie to the client. Future requests with that cookie are routed to the same server.

---

## Clear Architecture Diagram

```
                User Browser
                     │
                     ▼
            Application Load Balancer
                     │
                     │  Selects backend server
                     ▼
                 Server A
                     │
           Cookie returned to user
                     │
                     ▼
           Future Requests From User
                     │
                     ▼
            Application Load Balancer
                     │
          Cookie identifies Server A
                     │
                     ▼
                 Server A
```

All requests from that user continue to go to **Server A** while the cookie is valid.

---

# Sticky Session Inside Target Group

```
                Users
                  │
                  ▼
        Application Load Balancer
                  │
              Target Group
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Server1    Server2    Server3

User1 → Server1 (sticky)
User1 → Server1 (again)
User1 → Server1 (again)
```

---

# 3. Key Differences

| Feature         | Weighted Routing                | Sticky Sessions                     |
| --------------- | ------------------------------- | ----------------------------------- |
| Purpose         | Distribute traffic              | Maintain session persistence        |
| Level           | Between target groups           | Between instances in a target group |
| Mechanism       | Weight configuration            | Cookies                             |
| Use Case        | Canary / Blue-Green deployments | Stateful applications               |
| Request Pattern | Requests spread across backends | Same user goes to same backend      |

---

# 4. When Both Are Used Together

Weighted routing decides **which target group gets the traffic first**.

Sticky sessions then ensure **that the user sticks to a specific instance inside that target group**.

```
                 Users
                   │
                   ▼
           Application Load Balancer
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
   Target Group A         Target Group B
       (80%)                  (20%)
        │                     │
   ┌────┴────┐           ┌────┴────┐
   ▼         ▼           ▼         ▼
Instance1 Instance2   Instance3 Instance4

User sticks to one instance once selected
```

---

# 5. Interview Answer (Concise)

Weighted routing distributes traffic across multiple target groups based on configured weights and is commonly used for canary or blue‑green deployments. Sticky sessions ensure that requests from the same client are consistently routed to the same backend server using cookies, which is useful for stateful applications that require session persistence.
