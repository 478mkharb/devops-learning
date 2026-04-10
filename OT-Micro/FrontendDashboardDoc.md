# 📘 Frontend Dashboard Documentation (OT-Microservices)

---

## 🧩 Overview

This document provides a detailed explanation of the dashboard implementation along with the **final working code** for:

* `EmployeeData.js`
* `HomePage.react.js`

---

# 📁 1. EmployeeData.js (FINAL WORKING FILE)

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

---

# 📁 2. HomePage.react.js (FINAL WORKING FILE)

```javascript
// @flow

import * as React from "react";
import { Page, Grid, StatsCard } from "tabler-react";

import SiteWrapper from "./SiteWrapper.react";

import {
  ListAllEmployees,
  ListEmployeeActiveEmployee,
  ListEmployeeInActiveEmployee,
  AttendanceDistribution,
  RoleDistribution,
  LocationDistribution
} from "./EmployeeData";

function Home() {
  return (
    <SiteWrapper>
      <Page.Content title="Dashboard">

        {/* ===== TOP CARDS ===== */}
        <Grid.Row className="mb-4">

          <Grid.Col md={3}>
            <ListAllEmployees />
          </Grid.Col>

          <Grid.Col md={3}>
            <ListEmployeeActiveEmployee />
          </Grid.Col>

          <Grid.Col md={3}>
            <ListEmployeeInActiveEmployee />
          </Grid.Col>

          <Grid.Col md={3}>
            <StatsCard
              layout={1}
              total="4"
              label="Office Locations"
            />
          </Grid.Col>

        </Grid.Row>

        {/* ===== DONUT CHARTS IN ONE ROW ===== */}
        <Grid.Row>

         <Grid.Col md={4}>
           <AttendanceDistribution />
         </Grid.Col>

           <Grid.Col md={4}>
             <RoleDistribution />
           </Grid.Col>

           <Grid.Col md={4}>
             <LocationDistribution />
           </Grid.Col>

        </Grid.Row>

      </Page.Content>
    </SiteWrapper>
  );
}

export default Home;
```

---

# 🎯 Final Notes

* Layout handled only in `HomePage.react.js`
* Data logic handled only in `EmployeeData.js`
* No `movement` used in StatsCard
* All graphs aligned in single row
* Backend field names strictly followed

---

**End of Document**
