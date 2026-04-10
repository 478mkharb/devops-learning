# 📘 Frontend File Modifications – OT Microservices

This document provides a **complete, production-level record** of all changes made in the frontend (`~/frontend/src`) during system setup and debugging.

It includes:

* Files modified
* Exact changes
* Reasons (WHY)
* Full updated files (where required)

---

# 📁 1. AttendanceForm.js

## 🔧 Changes

* Fixed API endpoint
* Fixed payload structure
* Ensured backend compatibility

## ✅ Final Working Code (Relevant Section)

```javascript
import React from "react";
import { Page, Grid, Form, Button, Card } from "tabler-react";
import SiteWrapper from "./SiteWrapper.react";

class AttendanceForm extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      id: "",
      status: "",
      date: ""
    };
  }

  handleChange = (e) => {
    this.setState({
      [e.target.name]: e.target.value
    });
  };

  handleSubmit = (e) => {
    e.preventDefault();

    const payload = {
      id: this.state.id,
      status: this.state.status,
      date: this.state.date
    };

    fetch("/api/v1/attendance/create", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify(payload)
    })
      .then(res => res.json())
      .then(() => {
        alert("Attendance added successfully!");

        this.setState({
          id: "",
          status: "",
          date: ""
        });
      })
      .catch(err => {
        console.error(err);
        alert("Error adding attendance");
      });
  };

  render() {
    return (
      <SiteWrapper>
        <Page.Content title="Add Attendance">

          <Grid.Row>
            <Grid.Col md={6}>
              <Card>
                <Card.Header>
                  <Card.Title>Add Attendance</Card.Title>
                </Card.Header>

                <Card.Body>
                  <Form onSubmit={this.handleSubmit}>

                    <Form.Group label="Employee ID">
                      <Form.Input
                        name="id"
                        value={this.state.id}
                        onChange={this.handleChange}
                        placeholder="Enter Employee ID"
                        required
                      />
                    </Form.Group>

                    <Form.Group label="Status">
                      <Form.Select
                        name="status"
                        value={this.state.status}
                        onChange={this.handleChange}
                        required
                      >
                        <option value="">Select Status</option>
                        <option value="Present">Present</option>
                        <option value="Absent">Absent</option>
                      </Form.Select>
                    </Form.Group>

                    <Form.Group label="Date">
                      <Form.Input
                        type="date"
                        name="date"
                        value={this.state.date}
                        onChange={this.handleChange}
                        required
                      />
                    </Form.Group>

                    <Button color="primary" type="submit">
                      Submit
                    </Button>

                  </Form>
                </Card.Body>
              </Card>
            </Grid.Col>
          </Grid.Row>

        </Page.Content>
      </SiteWrapper>
    );
  }
}

export default AttendanceForm;
```

## 🧠 WHY

* Backend requires strict schema
* Nginx routing requires `/api/v1/...`

---

# 📁 2. AttendanceList.js (FULL FILE REPLACED)

## 🔧 Changes

* Added API integration
* Fixed endpoint
* Fixed table mapping

## ✅ FULL FILE

```javascript
import React, { useEffect, useState } from "react";
import { Page, Grid, Table, Form, Card } from "tabler-react";
import SiteWrapper from "./SiteWrapper.react";

function AttendanceList() {
  const [data, setData] = useState([]);
  const [search, setSearch] = useState("");

  useEffect(() => {
    loadData();
  }, []);

  const loadData = () => {
    Promise.all([
      fetch("/api/v1/attendance/search/all")
        .then(res => res.json())
        .catch(() => []),

      fetch("/api/v1/employee/search/all")
        .then(res => res.json())
        .catch(() => [])
    ])
    .then(([attendance, employees]) => {

      // Create ID → Name map
      const employeeMap = {};
      employees.forEach(emp => {
        employeeMap[emp.id] = emp.name;
      });

      // Merge attendance + employee
      let merged = attendance.map(a => ({
        id: a.id,
        name: employeeMap[a.id] || "Unknown",
        status: a.status,
        rawDate: a.date,
        date: new Date(a.date).toLocaleDateString()
      }));

      // Sort latest first
      merged.sort((a, b) => new Date(b.rawDate) - new Date(a.rawDate));

      setData(merged);
    })
    .catch(() => setData([]));
  };

  // 🔍 Search filter
  const filteredData = data.filter(item =>
    item.id.toLowerCase().includes(search.toLowerCase()) ||
    item.name.toLowerCase().includes(search.toLowerCase()) ||
    item.status.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <SiteWrapper>
      <Page.Content title="Attendance List">

        <Grid.Row>
          <Grid.Col md={12}>

            <Card>
              <Card.Header>
                <Card.Title>Attendance List</Card.Title>
              </Card.Header>

              <Card.Body>

                {/* 🔍 Search */}
                <Form.Group>
                  <Form.Input
                    placeholder="Search by ID, Name, Status..."
                    value={search}
                    onChange={e => setSearch(e.target.value)}
                  />
                </Form.Group>

                {/* 📊 Table */}
                <Table>
                  <Table.Header>
                    <Table.ColHeader>Employee ID</Table.ColHeader>
                    <Table.ColHeader>Name</Table.ColHeader>
                    <Table.ColHeader>Status</Table.ColHeader>
                    <Table.ColHeader>Date</Table.ColHeader>
                  </Table.Header>

                  <Table.Body>
                    {filteredData.map(item => (
                      <Table.Row key={item.id + item.rawDate}>
                        <Table.Col>{item.id}</Table.Col>
                        <Table.Col>{item.name}</Table.Col>
                        <Table.Col>{item.status}</Table.Col>
                        <Table.Col>{item.date}</Table.Col>
                      </Table.Row>
                    ))}
                  </Table.Body>

                </Table>

              </Card.Body>
            </Card>

          </Grid.Col>
        </Grid.Row>

      </Page.Content>
    </SiteWrapper>
  );
}

export default AttendanceList;
```

## 🧠 WHY

* Original file had NO API call
* Data was never loaded

---

# 📁 3. ListSalary.js (FULL LOGIC CHANGED)

## 🔧 Changes

* Implemented API aggregation
* Merged Employee + Salary data

## ✅ CORE LOGIC

```javascript
import React from "react";
import { Page, Grid, Table } from "tabler-react";
import SiteWrapper from "./SiteWrapper.react";

class ListSalary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { data: [] };
  }

  loadData() {
  Promise.all([
    fetch("/api/v1/employee/search/all").then(res => res.json()),
    fetch("/api/v1/salary/search/all").then(res => res.json())
  ])
  .then(([employees, salaries]) => {
    const salaryMap = {};
    salaries.forEach(s => {
      salaryMap[s.id] = s;
    });

    const merged = employees.map(emp => {
      return {
        ...emp,
        salary: salaryMap[emp.id] ? salaryMap[emp.id].salary : "N/A",
        processDate: salaryMap[emp.id] ? salaryMap[emp.id].processDate : "N/A",
        status: salaryMap[emp.id] ? salaryMap[emp.id].status : "N/A"
      };
    });

    console.log("Merged Data:", merged);
    this.setState({ data: merged });
  })
  .catch(err => console.error("Error:", err));
}

  componentDidMount() {
    this.loadData();
  }

  render() {
    return (
      <SiteWrapper>
        <Page.Card title="Salary List"></Page.Card>
        <Grid.Col md={6} lg={10} className="align-self-center">
          <Table>
            <Table.Header>
              <Table.ColHeader>ID</Table.ColHeader>
              <Table.ColHeader>Name</Table.ColHeader>
              <Table.ColHeader>Salary</Table.ColHeader>
              <Table.ColHeader>Process Date</Table.ColHeader>
              <Table.ColHeader>Status</Table.ColHeader>
            </Table.Header>
            <Table.Body>
              {this.state.data.map((item, i) => (
                <Table.Row key={i}>
                  <Table.Col>{item.id}</Table.Col>
                  <Table.Col>{item.name}</Table.Col>
                  <Table.Col>{item.salary}</Table.Col>
                  <Table.Col>{item.processDate}</Table.Col>
                  <Table.Col>{item.status}</Table.Col>
                </Table.Row>
              ))}
            </Table.Body>
          </Table>
        </Grid.Col>
      </SiteWrapper>
    );
  }
}

export default ListSalary;
```

## 🧠 WHY

* Salary service is independent
* Needed frontend aggregation

---

# 📁 4. EmployeeData.js (FULL FILE REPLACED)

## 🔧 Changes

* Removed invalid APIs
* Added analytics logic

## ✅ FULL FILE

```javascript
import React, { useEffect, useState } from "react";
import { StatsCard, Card } from "tabler-react";
import C3Chart from "react-c3js";

// ─────────────────────────────────────────────
// Utility
// ─────────────────────────────────────────────
function countByKey(data, key) {
  const result = {};
  data.forEach(item => {
    const value = item[key] || "Unknown";
    result[value] = (result[value] || 0) + 1;
  });
  return result;
}

// ─────────────────────────────────────────────
// EMPLOYEE CARDS
// ─────────────────────────────────────────────
export function ListAllEmployees() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch("/api/v1/employee/search/all")
      .then(res => res.json())
      .then(setData)
      .catch(() => setData([]));
  }, []);

  return (
    <StatsCard layout={1} total={data.length} label="Total Employees" />
  );
}

export function ListEmployeeActiveEmployee() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch("/api/v1/employee/search/all")
      .then(res => res.json())
      .then(setData)
      .catch(() => setData([]));
  }, []);

  const active = data.filter(e => e.status === "Current Employee").length;

  return (
    <StatsCard layout={1} total={active} label="Active Employees" />
  );
}

export function ListEmployeeInActiveEmployee() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch("/api/v1/employee/search/all")
      .then(res => res.json())
      .then(setData)
      .catch(() => setData([]));
  }, []);

  const inactive = data.filter(e => e.status === "Ex-Employee").length;

  return (
    <StatsCard layout={1} total={inactive} label="Ex Employees" />
  );
}

// ───────────────────────────
// ATTENDANCE DISTRIBUTION 
// ───────────────────────────
export function AttendanceDistribution() {
  const [data, setData] = useState({});
  const [presentRate, setPresentRate] = useState(0);

  useEffect(() => {
    fetch("/api/v1/attendance/search/all")
      .then(res => res.json())
      .then(res => {
        const result = { Present: 0, Absent: 0 };

        res.forEach(item => {
          if (item.status === "Present") result.Present++;
          else if (item.status === "Absent") result.Absent++;
        });

        const total = result.Present + result.Absent;
        const percentage = total ? ((result.Present / total) * 100).toFixed(1) : 0;

        setData(result);
        setPresentRate(percentage);
      })
      .catch(() => {
        setData({ Present: 0, Absent: 0 });
        setPresentRate(0);
      });
  }, []);

  return (
    <Card>
      <Card.Header>
        <Card.Title>Attendance Distribution</Card.Title>
      </Card.Header>

      <Card.Body>
        <C3Chart
          data={{
            columns: Object.entries(data),
            type: "donut",
            colors: {
              Present: "#28a745",
              Absent: "#dc3545",
            },
          }}
          size={{ height: 220 }}
        />
      </Card.Body>
    </Card>
  );
}

// ─────────────────────────────────────────────
// TODAY ATTENDANCE WIDGET
// ─────────────────────────────────────────────
export function TodayAttendanceStats() {
  const [todayStats, setTodayStats] = useState({
    present: 0,
    absent: 0,
  });

  useEffect(() => {
    fetch("/api/v1/attendance/search/all")
      .then(res => res.json())
      .then(res => {
        const today = new Date().toISOString().split("T")[0];

        let present = 0;
        let absent = 0;

        res.forEach(item => {
          if (item.date === today) {
            if (item.status === "Present") present++;
            else if (item.status === "Absent") absent++;
          }
        });

        setTodayStats({ present, absent });
      })
      .catch(() => setTodayStats({ present: 0, absent: 0 }));
  }, []);

  return (
    <StatsCard
      layout={1}
      total={`${todayStats.present}/${todayStats.present + todayStats.absent}`}
      label="Today's Attendance (P/A)"
    />
  );
}

// ─────────────────────────────────────────────
// ROLE DISTRIBUTION
// ─────────────────────────────────────────────
export function RoleDistribution() {
  const [data, setData] = useState({});

  useEffect(() => {
    fetch("/api/v1/employee/search/all")
      .then(res => res.json())
      .then(res => setData(countByKey(res, "designation")))
      .catch(() => setData({}));
  }, []);

  return (
    <Card>
      <Card.Header>
        <Card.Title>Role Distribution</Card.Title>
      </Card.Header>
      <Card.Body>
        <C3Chart
          data={{ columns: Object.entries(data), type: "donut" }}
          size={{ height: 220 }}
        />
      </Card.Body>
    </Card>
  );
}

// ─────────────────────────────────────────────
// LOCATION DISTRIBUTION
// ─────────────────────────────────────────────
export function LocationDistribution() {
  const [data, setData] = useState({});

  useEffect(() => {
    fetch("/api/v1/employee/search/all")
      .then(res => res.json())
      .then(res => setData(countByKey(res, "office_location")))
      .catch(() => setData({}));
  }, []);

  return (
    <Card>
      <Card.Header>
        <Card.Title>Location Distribution</Card.Title>
      </Card.Header>
      <Card.Body>
        <C3Chart
          data={{ columns: Object.entries(data), type: "donut" }}
          size={{ height: 220 }}
        />
      </Card.Body>
    </Card>
  );
}
```

## 🧠 WHY

* Backend does not provide analytics
* Needed client-side aggregation

---

# 📁 5. HomePage.react.js

## 🔧 Changes

Removed:

```javascript
StatusDistribution
```

## 🧠 WHY

* Component removed from EmployeeData
* Causing build failure

---

# 📁 6. EmployeeForm.js

## 🔧 Changes

```javascript
fetch('/api/v1/employee/create')
```

## 🧠 WHY

* Align with Nginx routing

---

# 🎯 FINAL SUMMARY

| File              | Type     | Status |
| ----------------- | -------- | ------ |
| AttendanceForm.js | Modified | ✅      |
| AttendanceList.js | Replaced | ✅      |
| ListSalary.js     | Modified | ✅      |
| EmployeeData.js   | Replaced | ✅      |
| HomePage.react.js | Modified | ✅      |
| EmployeeForm.js   | Modified | ✅      |

---

# 🧠 ARCHITECTURE NOTE

Frontend acts as:

👉 API Aggregation Layer

```text
Employee API + Salary API → Merged in UI
```

---

# 🚀 RESULT

✔ Fully working frontend
✔ All APIs integrated
✔ Dashboard functional
✔ Microservices unified via UI
