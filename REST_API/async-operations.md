# Async Operations (202 Pattern)

Use asynchronous processing for long-running tasks. Return 202 Accepted with a job resource.

## Pattern

```http
POST /api/reports
{
  "type": "sales",
  "range": "2025-01-01:2025-08-27"
}

202 Accepted
Location: /api/jobs/job_abc
{
  "job_id": "job_abc",
  "status": "queued",
  "status_url": "/api/jobs/job_abc",
  "estimated_completion": "2025-08-27T10:45:00Z"
}

# Poll job status
GET /api/jobs/job_abc
200 OK
{
  "job_id": "job_abc",
  "status": "processing",
  "progress": 45
}

# When complete
GET /api/jobs/job_abc
200 OK
{
  "job_id": "job_abc",
  "status": "succeeded",
  "result_url": "/api/reports/rep_123"
}
```

## Features

- Webhooks or SSE for push notifications
- Retry and dead-letter queues
- Idempotent job submission

## Interview Questions

1. When to choose async pattern?
2. How to track and resume jobs?
3. How to secure job status endpoints?
4. How to handle failure and retries?
5. How to estimate progress?
