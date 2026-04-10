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

function countByKey(data, key) {
  const result = {};
  data.forEach(item => {
    const value = item[key] || "Unknown";
    result[value] = (result[value] || 0) + 1;
  });
  return result;
}

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

export function StatusDistribution() {
  const [data, setData] = useState({});

  useEffect(() => {
    fetch("/api/v1/attendance/search/all")
      .then(res => res.json())
      .then(res => {
        const result = {};
        res.forEach(item => {
          const status = item.status || "Unknown";
          result[status] = (result[status] || 0) + 1;
        });
        setData(result);
      })
      .catch(() => setData({}));
  }, []);

  return (
    <Card>
      <Card.Header>
        <Card.Title>Status Distribution</Card.Title>
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
  StatusDistribution,
  RoleDistribution,
  LocationDistribution
} from "./EmployeeData";

function Home() {
  return (
    <SiteWrapper>
      <Page.Content title="Dashboard">

        <Grid.Row className="mb-4">
          <Grid.Col md={3}><ListAllEmployees /></Grid.Col>
          <Grid.Col md={3}><ListEmployeeActiveEmployee /></Grid.Col>
          <Grid.Col md={3}><ListEmployeeInActiveEmployee /></Grid.Col>
          <Grid.Col md={3}>
            <StatsCard layout={1} total="4" label="Office Locations" />
          </Grid.Col>
        </Grid.Row>

        <Grid.Row>
          <Grid.Col md={4}><StatusDistribution /></Grid.Col>
          <Grid.Col md={4}><RoleDistribution /></Grid.Col>
          <Grid.Col md={4}><LocationDistribution /></Grid.Col>
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
