# 📘 React Components
<p align="center">
  <img width="120" src="https://upload.wikimedia.org/wikipedia/commons/a/a7/React-icon.svg" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Concept-Components-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Type-Functional-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Architecture-Component--Based-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Usage-Frontend-purple?style=for-the-badge" />
</p>

---

## 📑 Table of Contents

* [Introduction](#-introduction)
* [What is a Component](#-what-is-a-component)
* [Types of Components](#-types-of-components)
* [Component Structure](#-component-structure)
* [Props and State](#-props-and-state)
* [Component Hierarchy](#-component-hierarchy)
* [Real Project Mapping (OT-Micro)](#-real-project-mapping-ot-micro)
* [DevOps Perspective](#-devops-perspective)
* [Interview Questions](#-interview-questions)
* [References](#-references)

---

## 📌 Introduction

React follows a component-based architecture where the entire UI is divided into small, reusable pieces called components. This makes applications easier to develop, maintain, and scale.

---

## 🔹 What is a Component?

A component is a reusable and independent piece of UI that defines how a part of the application looks and behaves.

### Example:

```javascript
function EmployeeCard() {
  return <h1>Employee Details</h1>;
}
```

---

## 🔹 Types of Components

### 1. Functional Components (Recommended)

```javascript
function App() {
  return <h1>Hello Mukesh</h1>;
}
```

✔ Simple and modern
✔ Uses hooks

---

### 2. Class Components (Legacy)

```javascript
class App extends React.Component {
  render() {
    return <h1>Hello</h1>;
  }
}
```

❌ Rarely used in modern apps

---

## 🔹 Component Structure

Each component typically contains:

* UI (JSX)
* Logic (JavaScript)
* Styling (CSS)

```javascript
function Employee() {
  return (
    <div>
      <h2>Employee</h2>
    </div>
  );
}
```

---

## 🔹 Props and State

### Props (Input Data)

```javascript
<Employee name="Mukesh" />
```

* Passed from parent
* Read-only

---

### State (Internal Data)

```javascript
const [data, setData] = useState([]);
```

* Managed inside component
* Changes UI dynamically

---

## 🔹 Component Hierarchy

React UI is built as a tree:

```
App
 ├── Navbar
 ├── EmployeeList
 └── Footer
```

---

## 🔹 Real Project Mapping (OT-Micro)

In your project:

* EmployeeForm.js → Handles employee creation
* EmployeeList.js → Displays employee data
* AttendanceForm.js → Handles attendance
* ListSalary.js → Displays salary data

### API Example:

```javascript
fetch('/api/v1/employee/search/all')
```

👉 Components interact with backend APIs

---

## 🔹 Component → Backend API Interaction (OT-Micro Flow)

### 🔥 High-Level Flow

```text
User Action → React Component → API Call → Backend Service → Database
        ↓
   Response ← JSON Data ← Backend ← DB
        ↓
   UI Update (React Re-render)
```

---

### 🔹 Step-by-Step Flow

#### 1. User Interaction

* User clicks button (e.g., Create Employee)
* Triggers component function

#### 2. Component Logic (Example: EmployeeForm.js)

```javascript
const handleSubmit = () => {
  fetch('/api/v1/employee/create', {
    method: 'POST',
    body: JSON.stringify(employeeData),
  });
};
```

* Collects state data
* Sends API request

---

#### 3. API Request

```javascript
fetch('/api/v1/employee/create')
```

* HTTP request sent to backend

---

#### 4. Routing (NGINX Layer)

```text
Browser → NGINX → Backend Service
```

* Routes requests:

  * employee → Employee API
  * attendance → Attendance API
  * salary → Salary API

---

#### 5. Backend Processing

* Receives request
* Validates data
* Executes logic
* Interacts with DB

---

#### 6. Database Layer

```text
Backend → DB → Query/Insert
```

* Stores or fetches data

---

#### 7. Response Sent

```json
[
  { "id": 1, "name": "Mukesh" }
]
```

* Backend returns JSON

---

#### 8. React Receives Response (Example: EmployeeList.js)

```javascript
useEffect(() => {
  fetch('/api/v1/employee/search/all')
    .then(res => res.json())
    .then(data => setEmployees(data));
}, []);
```

* Stores response in state

---

#### 9. UI Re-render

```javascript
employees.map(emp => <p>{emp.name}</p>)
```

* UI updates automatically

---

### 🔹 Full OT-Micro Mapping

| Layer     | Example                 |
| --------- | ----------------------- |
| Component | EmployeeForm.js         |
| API Call  | /api/v1/employee/create |
| Gateway   | NGINX                   |
| Service   | Employee API            |
| Database  | employee_info           |
| Response  | JSON                    |
| UI Update | EmployeeList.js         |

---

### 🔹 Key Concepts

* State → stores API data
* useEffect → triggers API calls
* Async → non-blocking calls
* Re-render → UI updates on state change

---

### 🔹 End-to-End Example

```text
1. User opens UI
2. React loads component
3. API call triggered
4. NGINX routes request
5. Backend processes request
6. DB returns data
7. JSON response sent
8. React updates state
9. UI re-renders
```

---

## 🔹 DevOps Perspective

* Components improve modularity
* Easier CI/CD updates
* Faster debugging
* Scalable frontend architecture

---

## 🎯 Interview Questions

🔥 Interview Question 1: What is a component in React?

A component in React is a reusable and independent piece of UI that encapsulates both structure and behavior. It allows developers to break down complex user interfaces into smaller, manageable parts.

---

🔥 Interview Question 2: What are props in React?

Props are inputs passed from one component to another. They are used to pass data and are read-only in nature, meaning a component cannot modify its props.

---

🔥 Interview Question 3: What is state in React?

State is a built-in object used to store dynamic data within a component. When state changes, the component re-renders automatically.

---

🔥 Interview Question 4: Difference between props and state?

Props are external inputs passed to a component, whereas state is managed internally within the component. Props are read-only, while state can be updated.

---

🔥 Interview Question 5: What is component hierarchy?

Component hierarchy refers to the parent-child relationship between components in a React application, where UI is structured as a tree.

---

## 🔗 References

* [https://react.dev/learn](https://react.dev/learn)
* [https://react.dev/learn/describing-the-ui](https://react.dev/learn/describing-the-ui)

---
