# 📘 ReactJS Guidebook (Complete Beginner → DevOps Guide)

<p align="center">
  <img width="120" src="https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Library-ReactJS-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Package-npm-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Runner-npx-yellow?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Bundler-Webpack-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Transpiler-Babel-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Deploy-NGINX-purple?style=for-the-badge" />
</p>

---

## 📑 Table of Contents

* [Introduction](#-introduction)
* [React Basics](#-what-is-reactjs)
* [Node.js, npm, npx](#-what-is-nodejs)
* [Modern React Setup (Recommended)](#-modern-react-setup-recommended)
* [Create React App (Legacy)](#-create-react-app-legacy)
* [Project Structure](#-project-structure)
* [Core Files](#-core-files-real-example)
* [Commands](#-important-commands)
* [Internal Working](#-how-react-works-internally)
* [Build Process](#-build-process)
* [NGINX Role](#-why-nginx-is-required)
* [Execution Flow](#-full-execution-flow)
* [FAQs](#-faqs-scenario-based)

---

## 📌 Introduction

ReactJS is a JavaScript library used to build modern, dynamic user interfaces. It enables developers to create Single Page Applications (SPA) where content updates without full page reload.

👉 Important Update (Modern React):

* Create React App (CRA) is **no longer recommended for new applications**
* React team encourages:

  * Using frameworks (Next.js, Remix)
  * Or build tools like **Vite, Parcel, RSBuild**

---

## 🔹 What is ReactJS?

ReactJS is a component-based library used to build reusable UI.

```javascript
function App() {
  return <h1>Employee Dashboard</h1>;
}
```

---

## 🔹 What is Node.js?

Node.js is required to run React build tools.

---

## 🔹 What is npm?

```bash
npm install
```

Installs dependencies.

---

## 🔹 What is npx?

```bash
npx create-react-app app
```

Runs packages without global install.

---

# 🚀 Modern React Setup (Recommended)

## 🔹 Why CRA is Deprecated?

* Slow build time
* Uses older Webpack config
* Hard to customize
* Not aligned with modern React architecture

👉 React official docs now recommend using **modern tools**.

---

## 🔹 Option 1: Vite (Recommended ⭐)

### Create App

```bash
npm create vite@latest employee-ui
cd employee-ui
npm install
npm run dev
```

👉 Features:

* Very fast startup
* Instant reload
* Uses modern ES modules

---

## 🔹 Option 2: Parcel

```bash
npm init -y
npm install parcel react react-dom
```

Run:

```bash
npx parcel index.html
```

👉 Zero configuration bundler

---

## 🔹 Option 3: RSBuild

```bash
npx create-rsbuild app
```

👉 High performance build tool (used in enterprise setups)

---

## 🔹 Recommendation Summary

| Tool    | Use Case             |
| ------- | -------------------- |
| Vite    | Best for most apps ⭐ |
| Parcel  | Simple setups        |
| RSBuild | High performance     |

---

# ⚠️ Create React App (Legacy)

## 🔹 Still Used In:

* Existing projects (like OT-Micro)

## 🔹 Create App (Old Way)

```bash
npx create-react-app employee-ui
cd employee-ui
npm start
```

👉 Works, but not recommended for new apps.

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

## 🔹 Core Files (Real Example)

### index.html

```html
<div id="root"></div>
```

### index.js

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
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

## 🔹 Important Commands

```bash
npm install
npm start
npm run build
```

---

## 🔹 How React Works Internally

```
JSX → Babel → JS → Webpack/Vite → Browser
```

---

## 🔹 Build Process

```bash
npm run build
```

---

## 🔹 Why NGINX is Required

* Serves static files
* Routes API

---

## 🔹 Full Execution Flow

1. Create app (Vite recommended)
2. Install dependencies
3. Run dev server
4. Build
5. Deploy
6. Serve using NGINX

---

## 🔹 FAQs (Scenario-Based)

### Should I use CRA for new apps?

No. Use Vite or frameworks.

### Existing CRA app?

Continue or migrate gradually.

### Best modern tool?

Vite.

---

## 🔗 References

* [https://react.dev/learn/build-a-react-app-from-scratch](https://react.dev/learn/build-a-react-app-from-scratch)
* [https://vitejs.dev](https://vitejs.dev)
* [https://parceljs.org](https://parceljs.org)
* [https://rsbuild.dev](https://rsbuild.dev)

---
