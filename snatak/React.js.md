# 📘 ReactJS Guidebook (Beginner → DevOps Perspective)

<p align="center">
  <img width="120" src="https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Library-ReactJS-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Runtime-Node.js-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Package-npm-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Runner-npx-yellow?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Build-Vite-purple?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Deploy-NGINX-orange?style=for-the-badge" />
</p>

---

## 📑 Table of Contents

* [Introduction](#-introduction)
* [React Fundamentals](#-react-fundamentals)
* [Node.js, npm, npx](#-nodejs-npm-npx)
* [Modern React Setup](#-modern-react-setup)
* [Create React App (Legacy)](#-create-react-app-legacy)
* [Project Structure](#-project-structure)
* [Core Files Explained](#-core-files-explained)
* [Commands](#-commands)
* [Internal Working](#-internal-working)
* [Build Process](#-build-process)
* [Idempotency](#-idempotency)
* [NGINX Role](#-nginx-role)
* [Execution Flow](#-execution-flow)
* [References](#-references)

---

## 📌 Introduction

ReactJS is a JavaScript library used to build modern, dynamic user interfaces. It is widely used for Single Page Applications (SPA), where content updates dynamically without full page reload.

In real-world DevOps workflows, React is not just UI—it involves dependency management, build tools, API communication, and deployment.

---

## 🔹 React Fundamentals

React is based on **component-driven architecture**.

```javascript
function App() {
  return <h1>Employee Dashboard</h1>;
}
```

👉 JSX is converted into JavaScript using Babel.

---

## 🔹 Node.js, npm, npx

### Node.js

* Runtime for executing JavaScript outside browser

### npm

```bash
npm install
```

* Installs dependencies from package.json

### npx

```bash
npx create-react-app app
```

* Runs packages without global install

---

# 🚀 Modern React Setup

👉 React recommends using modern tools instead of CRA.

## Vite (Recommended ⭐)

```bash
npm create vite@latest employee-ui
cd employee-ui
npm install
npm run dev
```

✔ Fast startup
✔ Modern architecture
✔ Better performance

---

## Parcel

```bash
npm init -y
npm install parcel react react-dom
npx parcel index.html
```

✔ Zero configuration

---

## RSBuild

```bash
npx create-rsbuild app
```

✔ High performance builds

---

## Recommendation

| Tool    | Use Case       |
| ------- | -------------- |
| Vite    | Best overall ⭐ |
| Parcel  | Simple apps    |
| RSBuild | Enterprise     |

---

# ⚠️ Create React App (Legacy)

```bash
npx create-react-app employee-ui
npm start
```

👉 Not recommended for new apps

---

## 🔹 Project Structure

```
employee-ui/
 ├── package.json
 ├── node_modules/
 ├── public/
 ├── src/
 └── build/
```

---

## 🔹 Core Files Explained

### index.html

```html
<div id="root"></div>
```

### index.js

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

### App.js

```javascript
import { useEffect, useState } from 'react';

function App() {
  const [employees, setEmployees] = useState([]);

  useEffect(() => {
    fetch('/api/v1/employee/search/all')
      .then(res => res.json())
      .then(data => setEmployees(data));
  }, []);

  return (
    <div>
      <h1>Employee List</h1>
      {employees.map((emp, index) => (
        <p key={index}>{emp.name}</p>
      ))}
    </div>
  );
}
```

---

## 🔹 Commands

```bash
npm install
npm start
npm run build
```

---

## 🔹 Internal Working

```
JSX → Babel → JS → Bundler → Browser
```

* Babel → converts JSX
* Bundler (Vite/Webpack) → bundles code

---

## 🔹 Build Process

```bash
npm run build
```

Output:

```
build/
 ├── index.html
 ├── static/js/main.js
 ├── static/css/main.css
```

---

## 🔹 Idempotency

* Same input → same output
* Safe for repeated builds
* Important for CI/CD

---

## 🔹 NGINX Role

After build:

* React becomes static files
* Needs web server

```
Browser → NGINX → React → API
```

---

## 🔹 Execution Flow

1. Create app
2. Install dependencies
3. Run dev server
4. Build app
5. Deploy build
6. Serve using NGINX

---

## 🔗 References

* [https://react.dev/learn/build-a-react-app-from-scratch](https://react.dev/learn/build-a-react-app-from-scratch)
* [https://react.dev](https://react.dev)
* [https://vitejs.dev](https://vitejs.dev)
* [https://parceljs.org](https://parceljs.org)
* [https://rsbuild.dev](https://rsbuild.dev)

---
