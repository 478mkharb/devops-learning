# 🔐 loginctl Command — DevOps Canvas

## 📌 Overview

`loginctl` is a systemd utility used to manage user sessions, seats, and login states.

---

## ⚙️ Key Commands

### 👥 List users

```
loginctl list-users
```

### 🖥️ List sessions

```
loginctl list-sessions
```

### 🔎 Show session details

```
loginctl show-session <session-id>
```

### 👤 Show user details

```
loginctl show-user <username>
```

### ❌ Terminate session

```
loginctl terminate-session <session-id>
```

### 🚫 Terminate all sessions of user

```
loginctl terminate-user <username>
```

### 🔐 Lock / Unlock session

```
loginctl lock-session <session-id>
loginctl unlock-session <session-id>
```

### 🔄 Enable linger (IMPORTANT)

```
loginctl enable-linger <username>
loginctl disable-linger <username>
```

---

## 🧠 DevOps Use Cases

* Monitor active SSH sessions
* Kill suspicious user sessions
* Run background services after logout
* Debug login/session issues

---

## ⚡ Example

```
loginctl list-sessions
loginctl terminate-session 3
loginctl enable-linger jenkins
```

---

## 📊 Summary

| Command           | Purpose                   |
| ----------------- | ------------------------- |
| list-users        | Show users                |
| list-sessions     | Show sessions             |
| terminate-session | Kill session              |
| enable-linger     | Run services after logout |

---

## 💡 Interview Tip

loginctl is used for managing user sessions via systemd and is safer than manually killing processes.
