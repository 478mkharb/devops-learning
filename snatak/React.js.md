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
* [What is ReactJS](#-what-is-reactjs)
* [Node.js, npm, npx](#-what-is-nodejs)
* [Project Setup](#-step-1-create-react-app)
* [Project Structure](#-project-structure)
* [package.json Explained](#-role-of-packagejson)
* [node_modules Explained](#-what-is-node_modules)
* [Core Files Deep Dive](#-core-files-real-example)
* [Commands & Execution](#-important-commands)
* [Internal Working (Babel + Webpack)](#-how-react-works-internally)
* [Build Process in Detail](#-build-process)
* [Idempotency Explained](#-idempotency-of-build)
* [Dev vs Prod](#-npm-start-vs-npm-run-build)
* [Why NGINX](#-why-nginx-is-required)
* [End-to-End Flow](#-full-execution-flow)
* [FAQs](#-faqs-scenario-based)

---

## 📌 Introduction

ReactJS is a JavaScript library used to build modern, dynamic user interfaces. It enables developers to create Single Page Applications (SPA), where only parts of the page update instead of reloading the entire page.

In real-world DevOps and production environments, React is not just about writing UI code. It involves:

* Managing dependencies using npm
* Using build tools like Webpack and Babel
* Integrating with backend APIs
* Deploying using web servers like NGINX

This guide walks you through the **complete lifecycle** of a React application using a real-world Employee Dashboard example.

---

## 🔹 What is ReactJS?

ReactJS is a component-based library. You build UI by combining small reusable components.

Example:

```javascript
function App() {
  return <h1>Employee Dashboard</h1>;
}
```

👉 Internally, React converts JSX into browser-understandable JavaScript.

---

## 🔹 What is Node.js?

Node.js is a runtime environment that allows JavaScript to run outside the browser.

👉 Required because:

* React build tools run on Node
* npm and npx depend on Node

---

## 🔹 What is npm?

npm (Node Package Manager) installs and manages project dependencies.

```bash
npm install
```

👉 Reads package.json and installs required libraries into node_modules.

---

## 🔹 What is npx?

npx executes packages without installing them globally.

```bash
npx create-react-app employee-ui
```

👉 Downloads and runs package temporarily.

---

## 🔹 Step 1: Create React App

```bash
npx create-react-app employee-ui
cd employee-ui
```

👉 Internally:

* Downloads template
* Creates project structure
* Installs dependencies

---

## 🔹 Project Structure

```
employee-ui/
 ├── package.json
 ├── node_modules/
 ├── public/
 │    └── index.html
 ├── src/
 │    ├── index.js
 │    ├── App.js
 └── build/
```

---

## 🔹 Role of package.json

This is the **brain of the project**.

Contains:

* Dependencies
* Scripts (start/build)
* Project metadata

Example:

```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build"
}
```

👉 Ensures consistency across environments.

---

## 🔹 What is node_modules?

node_modules stores all installed packages.

Key points:

* Auto-generated
* Very large
* Never edited manually
* Can be recreated anytime

---

## 🔹 Core Files (Real Example)

### public/index.html

```html
<div id="root"></div>
```

👉 Root where React mounts.

---

### src/index.js

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

👉 Connects React app to DOM.

---

### src/App.js (API Driven)

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

👉 Fetches data from backend APIs.

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
JSX Code → Babel → JS → Webpack → Bundle → Browser
```

### Babel:

* Converts JSX → JS

### Webpack:

* Bundles files

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

👉 Fully optimized for production.

---

## 🔹 Idempotency of Build

Build is idempotent:

* Same code → same output
* No duplication
* Safe to run multiple times

---

## 🔹 npm start vs npm run build

| Feature | npm start | npm run build |
| ------- | --------- | ------------- |
| Mode    | Dev       | Production    |
| Output  | Memory    | Files         |
| Use     | Testing   | Deployment    |

---

## 🔹 Why NGINX is Required

After build, React becomes static files.

NGINX:

* Serves files
* Routes API calls

```
Browser → NGINX → React → API
```

---

## 🔹 Full Execution Flow

1. Create app
2. Install dependencies
3. Run dev server
4. Build app
5. Deploy build
6. Serve via NGINX
7. API integration

---

## 🔹 FAQs (Scenario-Based)

### App works locally but not on server?

Because npm start is dev only. Use build + NGINX.

### API not loading?

Check backend and routing.

### node_modules deleted?

Run npm install.

### Build slow?

Because full optimization.

### Why npx?

Avoid global install.

---

## 🔗 References

* [https://react.dev](https://react.dev)
* [https://create-react-app.dev](https://create-react-app.dev)
* [https://webpack.js.org](https://webpack.js.org)
* [https://babeljs.io](https://babeljs.io)

---
