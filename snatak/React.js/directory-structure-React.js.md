# 📘 React.js Directory Structure

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
* [React Project Structure Overview](#-react-project-structure-overview)
* [Root Level Explained](#-root-level-explained)
* [node_modules Deep Dive](#-node_modules-deep-dive)
* [public Folder Explained](#-public-folder-explained)
* [src Folder Explained](#-src-folder-explained)
* [How src Connects to public](#-how-src-connects-to-public)
* [ReactDOM Explained](#-reactdom-explained)
* [Execution Flow](#-execution-flow)
* [DevOps Perspective](#-devops-perspective)
* [Interview Questions](#-interview-questions)
* [References](#-references)

---

## 📌 Introduction

React applications follow a structured directory layout that separates configuration, dependencies, static files, and application logic. Understanding this structure is critical for development, debugging, and deployment.

---

## 🔹 React Project Structure Overview

```
frontend/
 ├── package.json
 ├── node_modules/
 ├── public/
 ├── src/
 └── build/
```

---

## 🔹 Root Level Explained

### package.json

* Defines dependencies
* Defines scripts (start, build)
* Controls project behavior

👉 OT-Micro Mapping:

* Used to run frontend via `npm start`
* Used to build production files via `npm run build`

---

## 🔹 node_modules Deep Dive

> node_modules contains the complete dependency tree required to run the application.

### Key Points:

* Includes React, Webpack, Babel, and their dependencies
* Auto-generated using `npm install`
* Not committed to Git

👉 OT-Micro Mapping:

* Contains React libraries used in `EmployeeForm.js`, `EmployeeList.js`
* Contains fetch/polyfill utilities used in API calls

---

## 🔹 public Folder Explained

> Contains static files served directly to the browser.

### Key File:

```html
<div id="root"></div>
```

👉 OT-Micro Mapping:

* This is where the entire Employee UI gets rendered
* NGINX serves this file in production

---

## 🔹 src Folder Explained

> Contains all application logic and UI components.

### Structure (OT-Micro Real Example):

```
src/
 ├── index.js
 ├── App.react.js
 ├── EmployeeForm.js
 ├── EmployeeList.js
 ├── AttendanceForm.js
 ├── AttendanceList.js
 ├── ListSalary.js
 ├── SiteWrapper.react.js
 ├── HomePage.react.js
```

### File Roles:

* **index.js** → Entry point
* **App.react.js** → Root component
* **EmployeeForm.js** → Create employee
* **EmployeeList.js** → Display employees
* **AttendanceForm.js** → Attendance input
* **ListSalary.js** → Salary display
* **SiteWrapper.react.js** → Navigation + layout
* **HomePage.react.js** → Dashboard view

👉 OT-Micro API Flow Example:

```javascript
fetch('/api/v1/employee/search/all')
```

---

## 🔹 How src Connects to public

### Flow:

```
public/index.html
        ↓
<div id="root">
        ↓
src/index.js
        ↓
ReactDOM
        ↓
App Component
        ↓
Feature Components
```

---

## 🔹 ReactDOM Explained

> ReactDOM is responsible for rendering React components into the browser DOM.

### Key Role:

```javascript
ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

👉 OT-Micro Mapping:

* Injects full Employee Dashboard UI into browser

---

## 🔹 Execution Flow

```
Browser → index.html → ReactDOM → App → Components → API Calls → Backend Services
```

👉 Backend Services:

* Employee API
* Attendance API
* Salary API

---

## 🔹 DevOps Perspective

| Folder       | Role                |
| ------------ | ------------------- |
| package.json | Build & run control |
| node_modules | Dependencies        |
| public       | Static entry        |
| src          | Application logic   |
| build        | Deployment output   |

---

## 🎯 Interview Questions

🔥 Interview Question 1: What is node_modules?

node_modules is a directory that contains the complete dependency tree of a Node.js project. It includes all the direct dependencies defined in package.json as well as all indirect (transitive) dependencies required by those packages. This folder is automatically generated when we run `npm install` and is used by Node.js during development and build processes to resolve imports and execute tools like Webpack and Babel. It is not committed to version control because it can be recreated using package.json and package-lock.json.

---

🔥 Interview Question 2: How does React connect to HTML?

React connects to the HTML file through the index.js file using ReactDOM. In the public/index.html file, there is a root element (usually a div with id="root") which acts as a container. ReactDOM takes the main App component and renders it inside this root element. This creates a bridge between the static HTML file and the dynamic React application, allowing the UI to be updated dynamically without reloading the page.

---

🔥 Interview Question 3: What is the role of public folder?

The public folder contains static files that are served directly to the browser without being processed by the React build system. It includes the main index.html file, which acts as the entry point of the application. React mounts the entire application inside the root div of this HTML file. Additionally, the public folder may contain assets like images, icons, and manifest files that are required at runtime.

---

🔥 Interview Question 4: What is src folder?

The src folder contains all the core application logic of a React project. This includes components, state management, API calls, and business logic. It is the most important part of the application where developers write the actual code. Files inside src are processed by build tools like Babel and Webpack/Vite, which convert modern JavaScript and JSX into optimized browser-compatible code.

---

🔥 Interview Question 5: What is ReactDOM (and how is it related to DOM - Document Object Model)?

ReactDOM is a library that acts as a bridge between React and the browser's DOM (Document Object Model). Its primary responsibility is to render React components into actual HTML elements inside the browser. It takes the virtual representation of the UI (Virtual DOM) created by React and efficiently updates the real DOM by applying only the necessary changes. This improves performance and ensures that UI updates are fast and optimized.

---

## 🔗 References

* [https://react.dev](https://react.dev)
* [https://nodejs.org](https://nodejs.org)
* [https://vitejs.dev](https://vitejs.dev)
* [https://webpack.js.org](https://webpack.js.org)

---
