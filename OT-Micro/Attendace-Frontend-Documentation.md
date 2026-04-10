# Attendance Module Reference Files

This document contains the **final corrected versions** of:

* AttendanceForm.js
* AttendanceList.js

These are production-ready and aligned with microservices architecture.

---

# 1. AttendanceForm.js

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

---

# 2. AttendanceList.js

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

      const employeeMap = {};
      employees.forEach(emp => {
        employeeMap[emp.id] = emp.name;
      });

      let merged = attendance.map(a => ({
        id: a.id,
        name: employeeMap[a.id] || "Unknown",
        status: a.status,
        rawDate: a.date,
        date: new Date(a.date).toLocaleDateString()
      }));

      merged.sort((a, b) => new Date(b.rawDate) - new Date(a.rawDate));

      setData(merged);
    })
    .catch(() => setData([]));
  };

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

                <Form.Group>
                  <Form.Input
                    placeholder="Search by ID, Name, Status..."
                    value={search}
                    onChange={e => setSearch(e.target.value)}
                  />
                </Form.Group>

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

---

# End of Reference
