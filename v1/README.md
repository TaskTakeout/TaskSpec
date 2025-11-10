# TaskTakeout

An open standard, OpenAPI-driven spec and toolkit that frees tasks from proprietary lock-ins.

## Overview

TaskTakeout defines a shared API for task data. Any provider or client can implement it, which gives users true data freedom. Users can switch apps, sync accounts, or export and import their data without pain. A common spec means clients do not need one-off adapters and providers can focus on features instead of bespoke integrations.

## Why TaskTakeout?

- **Data Portability**: Export your tasks from one provider and import them seamlessly into another
- **No Vendor Lock-in**: Switch task management tools without losing your data or structure
- **Standard API**: One API spec that works across all compliant providers
- **Complete Data Preservation**: IDs, timestamps, relationships, and custom metadata all maintained during export/import
- **Developer Friendly**: RESTful API with comprehensive filtering, searching, and sorting

## Features

### Core Task Management
- Create, read, update, and delete tasks
- Hierarchical tasks with parent-child relationships
- Task completion tracking with automatic timestamp management
- Archiving for completed or inactive tasks
- Priority levels (0-99)
- Due dates with overdue detection
- Tags for organization (projects, contexts, etc.)
- Rich text descriptions

### Advanced Capabilities
- **Full-text search** across title and description
- **Flexible filtering** by completion status, priority, tags, due dates, parent tasks
- **Sorting** by created date, updated date, title, priority, due date, or completion status
- **Pagination** for efficient data retrieval
- **Custom metadata** for app-specific fields (assignees, estimates, custom properties)
- **Optimistic concurrency control** with ETags and conditional requests
- **Rate limiting** with retry guidance
- **Idempotency** for safe retries

### Data Portability
- **Complete export** of all tasks in JSON format
- **Bulk import** with ID and timestamp preservation
- **Atomic imports** - all tasks created or none (transactional)
- **Conflict handling** strategies (fail, skip, or upsert)
- **Validation mode** to test imports before committing

## API Endpoints

### Tasks

#### `GET /task/v1/tasks`
List and filter tasks with comprehensive query options.

**Query Parameters:**
- `completed` - Filter by completion status
- `archived` - Filter by archived status
- `priority` - Filter by specific priority level
- `tag` - Filter by tags (repeatable, all must match)
- `parent_id` - Filter by parent task (omit for root tasks)
- `search` - Full-text search across title and description
- `due_before` / `due_after` - Filter by due date range
- `overdue` - Find tasks past their due date
- `sort_by` - Sort field (created_at, updated_at, title, priority, due_date, completed)
- `order` - Sort order (asc or desc)
- `limit` / `offset` - Pagination controls

**Response:** Paginated list with total count

#### `POST /task/v1/tasks`
Create a new task.

**Required:** `title`

**Optional:** `description`, `completed`, `archived`, `priority`, `due_date`, `tags`, `metadata`, `parent_id`

**Returns:** Created task with server-generated ID and timestamps

#### `GET /task/v1/tasks/{id}`
Get a single task by ID.

**Returns:** Task details with ETag header for concurrency control

#### `PATCH /task/v1/tasks/{id}`
Partially update a task.

**Features:**
- Send only the fields you want to change
- Send `null` to clear nullable fields
- Automatic `completion_date` management when `completed` changes
- Optional `If-Match` or `If-Unmodified-Since` headers for optimistic locking

#### `DELETE /task/v1/tasks/{id}`
Delete a task and all its subtasks (cascading delete).

### Export & Import

#### `GET /task/v1/tasks/export`
Export all tasks in portable format.

**Formats:**
- `application/json` - Complete JSON array
- `application/x-ndjson` - Newline-delimited JSON stream

**Features:**
- Includes all fields (IDs, timestamps, relationships)
- Ordered with parents before children
- Ready for direct import to another provider

#### `POST /task/v1/tasks/import`
Import tasks in bulk.

**Parameters:**
- `validate_only` - Validate without writing (dry run)
- `on_conflict` - How to handle existing IDs (fail, skip, or upsert)
- `Idempotency-Key` header - Safe retries

**Features:**
- Preserves IDs, timestamps, and relationships
- Atomic transaction (all or nothing)
- Accepts exact export format

## Data Model

### Task Object

```json
{
  "id": "c9a1cbd1-4e89-4e2a-8a1f-8f9a6f7d6b1b",
  "title": "Draft Q4 plan",
  "description": "Outline scope and owners",
  "completed": false,
  "archived": false,
  "priority": 50,
  "due_date": "2025-12-01T17:00:00Z",
  "tags": ["work", "planning"],
  "metadata": {
    "estimated_hours": 4,
    "assignee": "alice"
  },
  "parent_id": null,
  "completion_date": null,
  "created_at": "2025-11-01T12:00:00Z",
  "updated_at": "2025-11-03T08:30:00Z"
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier (server-generated) |
| `title` | String | Task title (1-500 chars, required) |
| `description` | String | Detailed description (max 5000 chars) |
| `completed` | Boolean | Completion status (default: false) |
| `archived` | Boolean | Archive status (default: false) |
| `priority` | Integer | Priority level 0-99 (default: 0) |
| `due_date` | ISO 8601 | Due date in UTC (nullable) |
| `tags` | Array | Tags for organization (max 100, case-insensitive) |
| `metadata` | Object | Custom key-value data (suggested max 10KB) |
| `parent_id` | UUID | Parent task for subtasks (nullable) |
| `completion_date` | ISO 8601 | When completed (server-managed, read-only) |
| `created_at` | ISO 8601 | Creation timestamp (server-managed) |
| `updated_at` | ISO 8601 | Last update timestamp (server-managed) |

## Authentication

TaskTakeout uses Bearer token authentication (JWT).

```http
Authorization: Bearer <your-token>
```

## Error Handling

### Standard Error Response
```json
{
  "code": "NOT_FOUND",
  "message": "Task with id xyz not found"
}
```

### Validation Error Response
```json
{
  "code": "VALIDATION_ERROR",
  "message": "Invalid input",
  "field_errors": [
    {
      "field": "title",
      "message": "Title is required"
    }
  ]
}
```

### HTTP Status Codes
- `200` - Success
- `201` - Created
- `204` - No Content (successful deletion)
- `400` - Bad Request
- `401` - Unauthorized
- `404` - Not Found
- `409` - Conflict (import ID collision)
- `412` - Precondition Failed (concurrency conflict)
- `422` - Validation Error
- `429` - Rate Limited (see `Retry-After` header)
- `500` - Server Error

## Implementation Guide

### For API Providers

To implement TaskTakeout in your task management service:

1. **Implement the core endpoints** according to the OpenAPI spec
2. **Support JWT authentication** for secure access
3. **Handle timestamps** - All dates must be ISO 8601 in UTC
4. **Implement automatic completion_date management**:
   - Set to current UTC time when `completed` changes to `true`
   - Clear to `null` when `completed` changes to `false`
5. **Support cascading deletes** for parent tasks
6. **Implement export/import** for data portability
7. **Add rate limiting** and return appropriate `Retry-After` headers

### For API Clients

To integrate with TaskTakeout-compliant providers:

1. **Obtain an API token** from your provider
2. **Use the base URL** `/task/v1` with your provider's domain
3. **Handle pagination** when listing tasks
4. **Respect rate limits** and retry using `Retry-After` headers
5. **Use ETags** for optimistic concurrency when updating
6. **Send Idempotency-Key** headers for safe retries on create/import
7. **Leverage export/import** for backups and migrations

## Key Behaviors

### Completion Date Management
When you set `completed: true`, the server automatically sets `completion_date` to the current UTC time. When you set `completed: false`, the server clears `completion_date` to `null`. Clients cannot directly set `completion_date`.

### Parent Filtering Default
When querying tasks without specifying `parent_id`, only root-level tasks are returned (same as `parent_id="null"`). To get subtasks, explicitly provide the parent's UUID.

### Tag Matching
Tags are case-insensitive and trimmed. When filtering with multiple tag parameters (`?tag=work&tag=urgent`), only tasks with ALL specified tags are returned.

### Null Handling
- Send a field as `null` in a PATCH request to clear it
- Nulls sort last when sorting by `due_date` or `title`
- Ties are broken by `created_at` descending

### Optimistic Concurrency
Use `If-Match` (with ETag) or `If-Unmodified-Since` headers on PATCH requests to prevent conflicting updates. The server returns `412 Precondition Failed` if the resource has changed.

## Example Usage

### Create a Task
```bash
curl -X POST https://api.example.com/task/v1/tasks \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Review pull request",
    "priority": 75,
    "tags": ["work", "code-review"],
    "due_date": "2025-11-15T17:00:00Z"
  }'
```

### List Overdue Work Tasks
```bash
curl "https://api.example.com/task/v1/tasks?tag=work&overdue=true" \
  -H "Authorization: Bearer $TOKEN"
```

### Complete a Task
```bash
curl -X PATCH https://api.example.com/task/v1/tasks/$TASK_ID \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'
```

### Export All Tasks
```bash
curl https://api.example.com/task/v1/tasks/export \
  -H "Authorization: Bearer $TOKEN" \
  > tasks-backup.json
```

### Import Tasks
```bash
curl -X POST https://api.example.com/task/v1/tasks/import \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @tasks-backup.json
```

## Specification

The complete OpenAPI 3.1.0 specification is available in [`openapi.yaml`](./openapi.yaml).

## Contributing

We welcome contributions to improve the TaskTakeout specification:

1. Fork the repository
2. Create a feature branch
3. Make your changes to `openapi.yaml`
4. Submit a pull request with a clear description

## License

[Add your license information here]

## Support

For questions, issues, or discussions about TaskTakeout, please [open an issue](../../issues) in this repository.
