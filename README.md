# Task Router Service

> Sample internal documentation for a microservice that routes incoming tasks to the right freelancers in a marketplace.

---

## Overview

Task Router Service is a backend service responsible for assigning incoming tasks to suitable freelancers based on a set of rules (skills, rates, availability, timezone, and custom business logic).

It is designed to:
- reduce manual work for operations and support teams,
- ensure fast and fair distribution of tasks,
- provide clear logic for why a task was routed to a specific freelancer.

This document is intended for:
- product managers,
- support and operations teams,
- engineers integrating with the service,
- technical writers preparing user-facing documentation.

---

## Key Concepts

- **Task** — a unit of work created by a client (e.g. “Translate a 10-page document”, “Design a landing page”).
- **Freelancer** — a verified contractor with a profile, skills, and rates.
- **Routing rules** — conditions that determine which freelancers are eligible.
- **Routing attempt** — a single attempt to assign a task.
- **Routing session** — the full process from task creation to assignment or timeout.

---

## How It Works (High-Level)

1. A new task is created in the marketplace.
2. The marketplace backend sends task details to Task Router Service.
3. Task Router:
   - filters out ineligible freelancers,
   - ranks remaining candidates,
   - selects the most suitable ones.
4. The service returns:
   - selected freelancer(s),
   - routing metadata explaining the decision,
   - a routing ID for logging and debugging.
5. The marketplace assigns the task or retries if needed.

---

## API (Simplified)

> **Note:** The examples below are illustrative and do not represent a live API.

### Request

`POST /v1/tasks/route`

```json
{
  "task_id": "task_12345",
  "title": "Landing page copy",
  "description": "Need 3 sections of website copy in English",
  "budget": 150,
  "currency": "USD",
  "deadline": "2025-06-01T18:00:00Z",
  "required_skills": ["copywriting", "landing-pages"],
  "language": "en",
  "priority": "normal"
}
# Responce (Success)
{
  "routing_id": "route_98765",
  "status": "assigned",
  "freelancer_id": "freelancer_456",
  "reason": "matched_skills_and_rate",
  "details": {
    "matched_skills": ["copywriting", "landing-pages"],
    "rate_per_project": 140,
    "timezone_match": true
  }
# Response (No Match) 
{
  "routing_id": "route_98766",
  "status": "no_match",
  "reason": "no_freelancers_within_budget",
  "details": {
    "min_rate_found": 220,
    "number_of_candidates_checked": 35
  }
}Configuration

Common configuration options:
	•	MAX_CANDIDATES_PER_REQUEST — maximum freelancers per routing attempt.
	•	ROUTING_TIMEOUT_MS — maximum allowed routing time.
	•	DEFAULT_PRIORITY — default task priority.
	•	EXCLUDED_STATUSES — freelancer statuses excluded from routing.

Configuration is typically managed via environment variables or a shared config service.


Verification (How to Check It Works)

To confirm the service works as expected:
	1.	Send a routing request with valid parameters.
	2.	Confirm the response status is assigned or no_match.
	3.	Verify the selected freelancer matches required skills.
	4.	Review routing metadata and applied filters.
	5.	Check logs using the corresponding routing_id.


Logs & Debugging

Each routing session is logged with:
	•	routing_id
	•	task_id
	•	status
	•	selected_freelancer_id
	•	filters_applied
	•	number_of_candidates_checked
	•	timestamp

To debug issues:
	1.	Reproduce the routing request.
	2.	Locate logs by routing_id or task_id.
	3.	Review filtering logic and error messages.


Limitations
	•	The service does not:
	•	create tasks,
	•	modify freelancer profiles,
	•	send notifications.
	•	Routing quality depends on:
	•	accurate freelancer data,
	•	correct task parameters.
	•	Outdated data may lead to suboptimal results.


Typical Questions

Why was no freelancer assigned?
Check the reason and details fields — common causes include strict budgets or narrow skill requirements.

Can a freelancer be force-assigned?
No. Manual assignment should be handled by the marketplace backend.

Can non-technical teams use this service?
Yes. Product, support, and operations teams can send test requests, review logs, and use this document as a reference.
