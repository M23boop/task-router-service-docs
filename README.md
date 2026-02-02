# task-router-service-docs
Sample product &amp; technical documentation
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
- support / operations teams,
- engineers integrating with the service,
- technical writers working on external documentation.

---

## Key Concepts

- **Task** — a unit of work created by a client (e.g. “Translate 10-page document”, “Design a landing page”).
- **Freelancer** — a verified contractor in the marketplace with a profile, skills, and rates.
- **Routing rules** — a set of conditions that determine which freelancers are eligible for a task.
- **Routing attempt** — a single try to assign a task to a freelancer or a small group of freelancers.
- **Routing session** — the full process of assigning one task from creation to final assignment or timeout.

---

## How It Works (High-Level)

1. A new task is created in the marketplace.
2. The marketplace backend sends a request to Task Router Service with task details (scope, budget, deadline, required skills, language, etc.).
3. Task Router:
   - filters out ineligible freelancers (missing skills, wrong timezone, rate outside budget, blocked users, etc.),
   - ranks the remaining freelancers (relevance, response rate, recent activity),
   - selects a small set of candidates.
4. The service returns:
   - the selected freelancer(s),
   - routing metadata (why they were selected),
   - a routing ID for logging and debugging.
5. The marketplace assigns the task or retries with updated parameters if needed.

---

## API (Simplified)

> This section describes a simplified version of the API used to interact with the service.

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
}{
  "routing_id": "route_98765",
  "status": "assigned",
  "freelancer_id": "freelancer_456",
  "reason": "matched_skills_and_rate",
  "details": {
    "matched_skills": ["copywriting", "landing-pages"],
    "rate_per_project": 140,
    "timezone_match": true
  }
}{
  "routing_id": "route_98766",
  "status": "no_match",
  "reason": "no_freelancers_within_budget",
  "details": {
    "min_rate_found": 220,
    "number_of_candidates_checked": 35
  }
}
⸻

Configuration

Common configuration options:
	•	MAX_CANDIDATES_PER_REQUEST — maximum number of freelancers to consider in a single routing attempt.
	•	ROUTING_TIMEOUT_MS — maximum time allowed for a routing session.
	•	DEFAULT_PRIORITY — default priority (e.g. normal, high).
	•	EXCLUDED_STATUSES — list of freelancer statuses that should never receive tasks (e.g. blocked, suspended).

Configuration is typically managed via environment variables or a shared config service.

⸻

Verification (How to Check It Works)

When you make a test request to Task Router, verify:
	1.	Request is accepted
	•	HTTP status is 200 or 202.
	•	routing_id is present in the response.
	2.	Routing logic is applied
	•	returned freelancer has required skills,
	•	rate is within the budget,
	•	timezone / availability rules are respected (if applicable).
	3.	Logs contain routing details
	•	search by routing_id,
	•	check which filters were applied,
	•	confirm why specific freelancers were excluded.
	4.	No silent failures
	•	status is either assigned or no_match,
	•	no “empty” response without a reason.

⸻

Logs & Debugging

Each routing session is logged with the following fields:
	•	routing_id
	•	task_id
	•	status (assigned, no_match, error)
	•	selected_freelancer_id (if any)
	•	filters_applied
	•	number_of_candidates_checked
	•	timestamp

To debug an issue:
	1.	Reproduce the routing request (using the same task_id and parameters).
	2.	Find the corresponding log entry by routing_id or task_id.
	3.	Review:
	•	which filters excluded most candidates,
	•	whether the budget or skills were too strict,
	•	whether there were any internal errors.

⸻

Limitations
	•	The service does not:
	•	create tasks,
	•	change freelancer profiles,
	•	send notifications.
	•	Task Router relies on:
	•	up-to-date freelancer data,
	•	correct task parameters from the marketplace backend.
	•	If the underlying data is outdated or incomplete, routing results will be suboptimal.

⸻

Typical Questions

“Why was no freelancer assigned?”
Check the reason and details fields — often the main cause is budget limits, very narrow skill requirements, or conflicting filters.

“Can we force-assign a specific freelancer?”
This should be done through the marketplace backend, not directly through Task Router. The service is designed for automated, rule-based decisions.

“Can non-technical team members use this?”
Yes. Product, support, and operations teams can:
	•	send test requests via internal tools or API clients,
	•	check logs with predefined filters,
	•	use this document as a reference for interpreting results.
