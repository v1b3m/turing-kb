### **Agent Dashboard API Specification**

Read-only API for the **agent home dashboard**.  
Provides case lists scoped to the signed-in agent and their team, plus **aggregated case statistics** (priority, recency, assignment, SLA, and task counts).

### **Method Summary**

| Method | Path | Purpose |
| ----- | ----- | ----- |
| GET | `/api/v1/agent-dashboard/my-cases` | Cases assigned to the current user |
| GET | `/api/v1/agent-dashboard/my-team-cases` | Cases visible to the user’s team |
| GET | `/api/v1/agent-dashboard/case-statistics` | Aggregated dashboard counters |

### **Shared: Case List Item**

List endpoints return a consistent **case list structure** compatible with existing case APIs.

**Example:**

{  
  "sys\_id": "case-sys-id",  
  "number": "CS0001001",  
  "short\_description": "Example case",  
  "state": "In Progress",  
  "priority": "2 \- High",  
  "assignment\_group": "Tier 1 Support",  
  "assigned\_to": "Abel Tuter",  
  "opened": "2026-03-25 14:30:00",  
  "updated": "2026-03-28 09:00:00",  
  "channel": "web",  
  "account": "Acme Corp",  
  "contact": "Jane Doe"  
}

**Priority labels:**  
Examples include:

* 1 \- Critical  
* 2 \- High  
* 3 \- Moderate  
* 4 \- Low

### **GET `/my-cases`**

Returns cases assigned to the **current user**.

**Behavior:**

* Matches authenticated user as assignee  
* Default ordering:  
  * `updated` (descending)  
  * then `opened` (descending)  
* Empty results return `200 OK` with empty array

**Response:**

{  
  "status": "success",  
  "data": \[\],  
  "meta": {  
    "total": 0,  
    "limit": 50,  
    "offset": 0  
  }  
}

**Example:**

GET /api/v1/agent-dashboard/my-cases?limit=15

### **GET `/my-team-cases`**

Returns cases visible to the user’s **team scope**.

**Behavior:**

* Includes assignment group / queue scope  
* Not limited to user-only assignments  
* Same ordering as `/my-cases`

**Response:**  
Same structure as `/my-cases`.

**Example:**

GET /api/v1/agent-dashboard/my-team-cases?state=In%20Progress

### **GET `/case-statistics`**

Returns aggregated counts for dashboard cards.

**Default Scope:** team

#### **Query Parameters**

| Parameter | Type | Description |
| ----- | ----- | ----- |
| scope | string | `team` or `mine` |

#### **Response Fields**

| Field | Description |
| ----- | ----- |
| highPriorityCases | Cases with Critical or High priority |
| last3DaysUpdated | Cases updated within last 72 hours |
| unassignedCases | Cases without assignee |
| slaBreached | SLA breaches within last 24 hours |
| caseTasks | Total case tasks (recommended: active only) |

#### **Response Example**

{  
  "status": "success",  
  "data": {  
    "highPriorityCases": 12,  
    "last3DaysUpdated": 48,  
    "unassignedCases": 5,  
    "slaBreached": 3,  
    "caseTasks": 27  
  },  
  "meta": {  
    "asOf": "2026-03-28T12:00:00Z",  
    "scope": "team"  
  }  
}

### **Implementation Notes**

* **High Priority Cases:**  
  Must align with list filtering logic (Critical \+ High)  
* **Last 3 Days Updated:**  
  Based on `updated` timestamp (rolling 72 hours)  
* **SLA Breaches:**  
  Define clearly:  
  * Per case (at least one breach), or  
  * Per SLA record  
* **Case Tasks:**  
  Recommend counting only **active/open tasks**

**Also Tied to this not mentioned in the document**  
[**Agent Interaction Document**](https://docs.google.com/document/d/1Wp7GxzMbzIFkz2F5GOdFetyreFpgRKVySYIjGyiMvaE/edit?usp=sharing)