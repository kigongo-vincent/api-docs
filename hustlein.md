# Hustle IN API Documentation

## Base URL
```
http://localhost:8080/api
```

## Authentication
Most endpoints require JWT authentication via the `Authorization` header:
```
Authorization: Bearer <token>
```

Public endpoints (no auth required):
- `POST /auth/login`
- `POST /auth/signup`
- `POST /auth/google`
- `GET /health`
- `GET /ws/chat` (WebSocket)

## Error Response Format
All errors return JSON with an `error` field:
```json
{
  "error": "error message"
}
```

Common HTTP status codes:
- `400` - Bad Request (invalid parameters/body)
- `401` - Unauthorized (missing/invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `409` - Conflict (duplicate resource)
- `413` - Payload Too Large (storage quota exceeded)
- `500` - Internal Server Error

---

## Authentication

### POST /auth/login
Authenticate user with email and password.

**Authentication:** None (public)

**Request Body:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response (200):**
```json
{
  "user": {
    "id": "string (uuid)",
    "email": "string",
    "name": "string",
    "role": "string (freelancer|consultant|company_admin|super_admin)",
    "companyId": "string (uuid)",
    "avatarUrl": "string (optional)",
    "departmentId": "string (uuid, optional)",
    "status": "string (active|inactive)",
    "lastSeen": "string (iso8601, optional)"
  },
  "token": "string (jwt)"
}
```

### POST /auth/signup
Register a new user.

**Authentication:** None (public)

**Request Body:**
```json
{
  "email": "string",
  "name": "string",
  "password": "string (min 8 chars)",
  "role": "string (optional, default: consultant)",
  "companyId": "string (uuid, optional)",
  "companyName": "string (optional, required for company_admin signup)"
}
```

**Response (201):** Same as `/auth/login`

### POST /auth/google
Authenticate with Google OAuth token.

**Authentication:** None (public)

**Request Body:**
```json
{
  "access_token": "string"
}
```

**Response (200):** Same as `/auth/login`

---

## Companies

### GET /companies
List all companies.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin` (when `requireAdmin=true`)
- **Roles:** All authenticated (when `requireAdmin=false`)

**Query Parameters:** None

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "name": "string",
    "subscription": "string (free|paid)",
    "logoUrl": "string (optional)",
    "createdAt": "string (iso8601)",
    "taxRate": "integer (optional)",
    "storageLimitMb": "integer (optional)",
    "storageUsedMb": "integer (optional)",
    "address": "string (optional)",
    "phone": "string (optional)",
    "email": "string (optional)"
  }
]
```

### GET /companies/:id
Get a specific company.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Super admin or member of the company

**Path Parameters:**
- `id` (string, uuid, required) - Company ID

**Response (200):** Single company object (same structure as list)

### POST /companies
Create a new company.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Request Body:**
```json
{
  "name": "string",
  "subscription": "string (optional, default: free)",
  "taxRate": "integer (optional)",
  "storageLimitMb": "integer (optional)",
  "address": "string (optional)",
  "phone": "string (optional)",
  "email": "string (optional)"
}
```

**Response (201):** Single company object

### PUT /companies/:id
Update a company.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `id` (string, uuid, required) - Company ID

**Request Body:** Partial update, any of:
```json
{
  "name": "string",
  "subscription": "string",
  "address": "string",
  "phone": "string",
  "email": "string",
  "taxRate": "integer",
  "storageLimitMb": "integer",
  "storageUsedMb": "integer",
  "logoUrl": "string"
}
```

**Response (200):** Single company object

### DELETE /companies/:id
Delete a company.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `id` (string, uuid, required) - Company ID

**Response (204):** No content

### POST /companies/:id/logo
Upload company logo.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `id` (string, uuid, required) - Company ID

**Request:** Multipart form data
- `file` (file, required) - Logo image file

**Response (200):**
```json
{
  "logoUrl": "string"
}
```

---

## Users

### GET /me
Get current authenticated user.

**Authentication:** Required
- **Roles:** All authenticated

**Response (200):**
```json
{
  "id": "string (uuid)",
  "email": "string",
  "name": "string",
  "role": "string (freelancer|consultant|company_admin|super_admin)",
  "companyId": "string (uuid)",
  "avatarUrl": "string (optional)",
  "departmentId": "string (uuid, optional)",
  "status": "string (active|inactive)",
  "lastSeen": "string (iso8601, optional)",
  "createdAt": "string (iso8601)",
  "updatedAt": "string (iso8601)"
}
```

### GET /users
List all users.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin` (when `requireAdmin=true`)
- **Roles:** All authenticated (when `requireAdmin=false`)

**Response (200):** Array of user objects (same structure as `/me`)

### GET /users/:id
Get a specific user.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `id` (string, uuid, required) - User ID

**Response (200):** Single user object

### GET /users/company/:companyId
Get users by company.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `companyId` (string, uuid, required) - Company ID

**Response (200):** Array of user objects

### GET /users/email/:email
Get user by email.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `email` (string, required) - User email

**Response (200):** Single user object

### GET /users/:id/profile
Get user profile (alias for GET /users/:id).

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `id` (string, uuid, required) - User ID

**Response (200):** Single user object

### POST /users
Create a new user.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Request Body:**
```json
{
  "email": "string",
  "name": "string",
  "role": "string (optional, default: consultant)",
  "companyId": "string (uuid)",
  "avatarUrl": "string (optional)",
  "departmentId": "string (uuid, optional)",
  "status": "string (optional, default: active)"
}
```

**Response (201):** Single user object

### PUT /users/:id
Update a user.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `id` (string, uuid, required) - User ID

**Request Body:** Partial update, any of:
```json
{
  "name": "string",
  "role": "string",
  "avatarUrl": "string",
  "departmentId": "string (uuid, optional)",
  "status": "string",
  "lastSeen": "string (iso8601)"
}
```

**Response (200):** Single user object

### DELETE /users/:id
Delete a user.

**Authentication:** Required
- **Roles:** `company_admin`, `super_admin`

**Path Parameters:**
- `id` (string, uuid, required) - User ID

**Response (204):** No content

---

## Projects

### GET /projects
List projects.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Super admin sees all; company roles see company projects; freelancers see assigned projects

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "companyId": "string (uuid)",
    "name": "string",
    "description": "string",
    "projectLeadId": "string (uuid)",
    "workflowId": "string (uuid)",
    "folderId": "string (optional)",
    "status": "string (active|archived)",
    "projectType": "string (internal|external)",
    "dueDate": "string (yyyy-mm-dd, optional)",
    "budgetType": "string (hourly|fixed|hybrid, optional)",
    "hourlyRate": "number (optional)",
    "fixedBudget": "number (optional)",
    "currency": "string (optional)",
    "createdAt": "string (iso8601)"
  }
]
```

### GET /projects/:id
Get a specific project.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Must have project access via company or assignment

**Path Parameters:**
- `id` (string, uuid, required) - Project ID

**Response (200):** Single project object

### GET /projects/company/:companyId
List projects by company.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Super admin or same company

**Path Parameters:**
- `companyId` (string, uuid, required) - Company ID

**Response (200):** Array of project objects

### GET /projects/lead/:leadId
List projects by lead.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `leadId` (string, uuid, required) - User ID of project lead

**Response (200):** Array of project objects

### POST /projects
Create a new project.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Request Body:**
```json
{
  "companyId": "string (uuid)",
  "name": "string",
  "description": "string (optional)",
  "projectLeadId": "string (uuid)",
  "folderId": "string (optional)",
  "projectType": "string (optional, default: internal)",
  "dueDate": "string (yyyy-mm-dd, optional)",
  "budgetType": "string (optional)",
  "hourlyRate": "number (optional)",
  "fixedBudget": "number (optional)",
  "currency": "string (optional)"
}
```

**Response (201):** Single project object

### PUT /projects/:id
Update a project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - Project ID

**Request Body:** Partial update, any of:
```json
{
  "name": "string",
  "description": "string",
  "projectLeadId": "string (uuid)",
  "folderId": "string",
  "status": "string",
  "dueDate": "string (yyyy-mm-dd)",
  "budgetType": "string",
  "hourlyRate": "number",
  "fixedBudget": "number",
  "currency": "string"
}
```

**Response (200):** Single project object

### DELETE /projects/:id
Delete a project.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Project ID

**Response (204):** No content

---

## Tasks

### GET /tasks
List tasks.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Super admin sees all; company roles see company tasks; freelancers see assigned tasks

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "projectId": "string (uuid)",
    "milestoneId": "string (uuid, optional)",
    "title": "string",
    "description": "string",
    "workflowStateId": "string (uuid)",
    "ownerId": "string (uuid)",
    "duration": "integer (minutes)",
    "priority": "string (low|medium|high)",
    "dueDate": "string (optional)",
    "dependencyIds": ["string (uuid)"],
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)"
  }
]
```

### GET /tasks/:id
Get a specific task.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - Task ID

**Response (200):** Single task object

### GET /projects/:projectId/tasks
List tasks by project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Response (200):** Array of task objects

### GET /milestones/:milestoneId/tasks
List tasks by milestone.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `milestoneId` (string, uuid, required) - Milestone ID

**Response (200):** Array of task objects

### GET /tasks/owner/:ownerId
List tasks by owner.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Freelancers/consultants can only query their own tasks

**Path Parameters:**
- `ownerId` (string, uuid, required) - User ID

**Response (200):** Array of task objects

### POST /tasks
Create a new task.

**Authentication:** Required
- **Roles:** All authenticated with project access
- **Access:** Freelancers/consultants can only create tasks assigned to themselves

**Request Body:**
```json
{
  "projectId": "string (uuid)",
  "milestoneId": "string (uuid, optional)",
  "title": "string",
  "description": "string (optional)",
  "workflowStateId": "string (uuid)",
  "ownerId": "string (uuid)",
  "duration": "integer (minutes)",
  "priority": "string (optional, default: medium)",
  "dueDate": "string (optional)",
  "dependencyIds": ["string (uuid)"]
}
```

**Response (201):** Single task object

### PUT /tasks/:id
Update a task.

**Authentication:** Required
- **Roles:** All authenticated with project access
- **Access:** Freelancers/consultants cannot reassign tasks to others

**Path Parameters:**
- `id` (string, uuid, required) - Task ID

**Request Body:** Partial update, any of:
```json
{
  "title": "string",
  "description": "string",
  "workflowStateId": "string (uuid)",
  "ownerId": "string (uuid)",
  "milestoneId": "string (uuid, optional)",
  "priority": "string",
  "dueDate": "string",
  "dependencyIds": ["string (uuid)"]
}
```

**Response (200):** Single task object

### DELETE /tasks/:id
Delete a task.

**Authentication:** Required
- **Roles:** All authenticated with project access
- **Access:** Freelancers/consultants can only delete their own tasks

**Path Parameters:**
- `id` (string, uuid, required) - Task ID

**Response (204):** No content

---

## Milestones

### GET /milestones
List milestones.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Super admin sees all; company roles see company milestones; freelancers see assigned project milestones

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "projectId": "string (uuid)",
    "name": "string",
    "priority": "string (low|medium|high)",
    "targetDate": "string",
    "workflowStateId": "string (uuid, optional)",
    "taskIds": ["string (uuid)"],
    "assigneeIds": ["string (uuid, optional)"],
    "summary": [
      {
        "userId": "string (uuid)",
        "totalDuration": "number (hours)"
      }
    ],
    "createdAt": "string (iso8601)"
  }
]
```

### GET /milestones/:id
Get a specific milestone.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - Milestone ID

**Response (200):** Single milestone object

### GET /projects/:projectId/milestones
List milestones by project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Response (200):** Array of milestone objects

### POST /milestones
Create a new milestone.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Request Body:**
```json
{
  "projectId": "string (uuid)",
  "name": "string",
  "priority": "string (optional, default: medium)",
  "targetDate": "string",
  "workflowStateId": "string (uuid, optional)",
  "taskIds": ["string (uuid)"],
  "assigneeIds": ["string (uuid)"]
}
```

**Response (201):** Single milestone object

### PUT /milestones/:id
Update a milestone.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - Milestone ID

**Request Body:** Partial update, any of:
```json
{
  "name": "string",
  "priority": "string",
  "targetDate": "string",
  "workflowStateId": "string (uuid, optional)",
  "taskIds": ["string (uuid)"],
  "assigneeIds": ["string (uuid)"]
}
```

**Response (200):** Single milestone object

### DELETE /milestones/:id
Delete a milestone.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - Milestone ID

**Response (204):** No content

---

## Comments

### GET /comments
List comments by entity.

**Authentication:** Required
- **Roles:** All authenticated with entity access

**Query Parameters:**
- `entityType` (string, required) - "task" or "doc"
- `entityId` (string, uuid, required) - Entity ID

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "entityType": "string (task|doc)",
    "entityId": "string (uuid)",
    "authorId": "string (uuid)",
    "body": "string",
    "attachmentUrl": "string (optional)",
    "attachmentType": "string (optional)",
    "attachmentSize": "string (optional)",
    "createdAt": "string (iso8601)"
  }
]
```

### POST /comments
Create a new comment.

**Authentication:** Required
- **Roles:** All authenticated with entity access

**Request Body:**
```json
{
  "entityType": "string (task|doc)",
  "entityId": "string (uuid)",
  "authorId": "string (uuid, must match current user)",
  "body": "string",
  "attachmentUrl": "string (optional)",
  "attachmentType": "string (optional)",
  "attachmentSize": "string (optional)"
}
```

**Response (201):** Single comment object

### PUT /comments/:id
Update a comment.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only comment author

**Path Parameters:**
- `id` (string, uuid, required) - Comment ID

**Request Body:** Partial update, any of:
```json
{
  "body": "string",
  "attachmentUrl": "string",
  "attachmentType": "string",
  "attachmentSize": "string"
}
```

**Response (200):** Single comment object

### DELETE /comments/:id
Delete a comment.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only comment author

**Path Parameters:**
- `id` (string, uuid, required) - Comment ID

**Response (204):** No content

---

## Notes

### GET /notes
List current user's notes.

**Authentication:** Required
- **Roles:** All authenticated

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "title": "string",
    "content": "string",
    "color": "string",
    "projectFileNodeId": "string (optional)",
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)"
  }
]
```

### GET /notes/:id
Get a specific note.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only note owner

**Path Parameters:**
- `id` (string, uuid, required) - Note ID

**Response (200):** Single note object

### POST /notes
Create a new note.

**Authentication:** Required
- **Roles:** All authenticated

**Request Body:**
```json
{
  "title": "string",
  "content": "string (optional)",
  "color": "string (optional, default: #e5e7eb)",
  "projectFileNodeId": "string (optional)"
}
```

**Response (201):** Single note object

### PUT /notes/:id
Update a note.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only note owner

**Path Parameters:**
- `id` (string, uuid, required) - Note ID

**Request Body:** Partial update, any of:
```json
{
  "title": "string",
  "content": "string",
  "color": "string"
}
```

**Response (200):** Single note object

### DELETE /notes/:id
Delete a note.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only note owner

**Path Parameters:**
- `id` (string, uuid, required) - Note ID

**Response (204):** No content

---

## Calendar

### GET /calendar
List current user's calendar events.

**Authentication:** Required
- **Roles:** All authenticated

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "projectId": "string (uuid, optional)",
    "taskId": "string (uuid, optional)",
    "title": "string",
    "start": "string",
    "end": "string",
    "type": "string"
  }
]
```

### POST /calendar
Create a new calendar event.

**Authentication:** Required
- **Roles:** All authenticated

**Request Body:**
```json
{
  "projectId": "string (uuid, optional)",
  "taskId": "string (uuid, optional)",
  "title": "string",
  "start": "string",
  "end": "string",
  "type": "string"
}
```

**Response (201):** Single calendar event object

### PUT /calendar/:id
Update a calendar event.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only event owner

**Path Parameters:**
- `id` (string, uuid, required) - Event ID

**Request Body:** Partial update, any of:
```json
{
  "title": "string",
  "start": "string",
  "end": "string",
  "type": "string",
  "projectId": "string (uuid, optional)",
  "taskId": "string (uuid, optional)"
}
```

**Response (200):** Single calendar event object

### DELETE /calendar/:id
Delete a calendar event.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Only event owner

**Path Parameters:**
- `id` (string, uuid, required) - Event ID

**Response (204):** No content

---

## Invoices

### GET /companies/:companyId/invoices
List invoices by company.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `companyId` (string, uuid, required) - Company ID

**Query Parameters:**
- `status` (string, optional) - Filter by status

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "companyId": "string (uuid)",
    "number": "string",
    "clientName": "string",
    "amount": "number",
    "currency": "string",
    "dueDate": "string (yyyy-mm-dd)",
    "issuedDate": "string (yyyy-mm-dd, optional)",
    "status": "string (unpaid|paid)",
    "description": "string",
    "paidAt": "string (iso8601, optional)",
    "consultantId": "string (uuid, optional)",
    "consultantName": "string (optional)",
    "logoUrl": "string (optional)",
    "issuer": "object (optional)",
    "bank": "object (optional)",
    "lineItems": "array (optional)",
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)"
  }
]
```

### GET /invoices/:id
Get a specific invoice.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Invoice ID

**Response (200):** Single invoice object

### POST /invoices
Create a new invoice.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Request Body:**
```json
{
  "companyId": "string (uuid)",
  "number": "string",
  "clientName": "string",
  "amount": "number",
  "currency": "string (optional)",
  "dueDate": "string (yyyy-mm-dd)",
  "issuedDate": "string (yyyy-mm-dd, optional)",
  "status": "string (optional, default: unpaid)",
  "description": "string (optional)",
  "consultantId": "string (uuid, optional)",
  "consultantName": "string (optional)",
  "logoUrl": "string (optional)",
  "issuer": "object (optional)",
  "bank": "object (optional)",
  "lineItems": "array (optional)"
}
```

**Response (201):** Single invoice object

### PUT /invoices/:id
Update an invoice.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Invoice ID

**Request Body:** Partial update, any of:
```json
{
  "number": "string",
  "clientName": "string",
  "amount": "number",
  "currency": "string",
  "dueDate": "string (yyyy-mm-dd)",
  "issuedDate": "string (yyyy-mm-dd)",
  "status": "string",
  "description": "string",
  "paidAt": "string (iso8601)",
  "consultantId": "string (uuid, optional)",
  "consultantName": "string",
  "logoUrl": "string"
}
```

**Response (200):** Single invoice object

### PATCH /invoices/:id/paid
Mark invoice as paid.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Invoice ID

**Response (200):** Single invoice object with status="paid"

### DELETE /invoices/:id
Delete an invoice.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Invoice ID

**Response (204):** No content

---

## Departments

### GET /companies/:companyId/departments
List departments by company.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `companyId` (string, uuid, required) - Company ID

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "companyId": "string (uuid)",
    "name": "string",
    "description": "string",
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)"
  }
]
```

### POST /companies/:companyId/departments
Create a new department.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `companyId` (string, uuid, required) - Company ID

**Request Body:**
```json
{
  "name": "string",
  "description": "string (optional)"
}
```

**Response (201):** Single department object

### PUT /departments/:id
Update a department.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `id` (string, uuid, required) - Department ID

**Request Body:** Partial update, any of:
```json
{
  "name": "string",
  "description": "string"
}
```

**Response (200):** Single department object

### DELETE /departments/:id
Delete a department.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `id` (string, uuid, required) - Department ID

**Response (204):** No content

---

## Project Files

### GET /files/:id
Get a specific project file or folder.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - File ID

**Response (200):**
```json
{
  "id": "string (uuid)",
  "projectId": "string (uuid)",
  "parentId": "string (uuid, optional)",
  "name": "string",
  "type": "string (file|folder|link)",
  "url": "string",
  "sizeBytes": "integer (optional)",
  "contentType": "string (optional)",
  "uploadedById": "string (uuid)",
  "createdAt": "string (iso8601)"
}
```

### GET /projects/:projectId/files
List files by project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Query Parameters:**
- `parentId` (string, uuid, optional) - Filter by parent folder (empty for root, "*" for all)

**Response (200):** Array of file objects

### GET /projects/:projectId/files/tree
List all files in tree structure by project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Response (200):** Array of all file objects (flat list)

### GET /projects/:projectId/files/storage
Get storage summary for project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Response (200):**
```json
{
  "projectId": "string (uuid)",
  "fileCount": "integer",
  "folderCount": "integer",
  "projectBytes": "integer",
  "companyId": "string (uuid)",
  "usedBytes": "integer",
  "limitBytes": "integer",
  "limitMb": "integer",
  "usagePercent": "number",
  "remainingBytes": "integer"
}
```

### POST /projects/:projectId/files
Create a new file/folder/link.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Request Body:**
```json
{
  "name": "string",
  "type": "string (file|folder|link)",
  "url": "string (required for file/link)",
  "parentId": "string (uuid, optional)"
}
```

**Response (201):** Single file object

### POST /projects/:projectId/files/upload
Upload a file to project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Request:** Multipart form data
- `file` (file, required) - File to upload
- `name` (string, optional) - Custom filename
- `parentId` (string, uuid, optional) - Parent folder ID

**Response (201):** Single file object

**Error (413):** Storage quota exceeded

### PUT /files/:id
Update a file.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `id` (string, uuid, required) - File ID

**Request Body:** Partial update, any of:
```json
{
  "name": "string",
  "url": "string",
  "sizeBytes": "integer",
  "contentType": "string"
}
```

**Response (200):** Single file object

### PATCH /files/:id/move
Move a file to different parent folder.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - File ID

**Request Body:**
```json
{
  "parentId": "string (uuid, optional)"
}
```

**Response (200):** Single file object

### DELETE /files/:id
Delete a file or folder.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - File ID

**Response (204):** No content

---

## Reports

### GET /reports/stat-cards
Get stat cards for dashboard.

**Authentication:** Required
- **Roles:** All authenticated

**Query Parameters:**
- `projectId` (string, uuid, optional) - Filter by project

**Response (200):**
```json
[
  {
    "label": "string",
    "value": "integer",
    "trend": "string (up|down|neutral, optional)",
    "trendPercent": "integer (optional)"
  }
]
```

### GET /reports/completion-by-owner
Get task completion by owner.

**Authentication:** Required
- **Roles:** All authenticated

**Query Parameters:**
- `projectId` (string, uuid, optional) - Filter by project

**Response (200):**
```json
[
  {
    "ownerId": "string (uuid)",
    "ownerName": "string",
    "completed": "integer",
    "total": "integer"
  }
]
```

### GET /reports/progress-over-time
Get progress over time.

**Authentication:** Required
- **Roles:** All authenticated

**Query Parameters:**
- `projectId` (string, uuid, optional) - Filter by project

**Response (200):**
```json
[
  {
    "date": "string (yyyy-mm-dd)",
    "completed": "integer"
  }
]
```

---

## Workflow

### GET /projects/:projectId/workflow
Get workflow for a project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Response (200):**
```json
{
  "id": "string (uuid)",
  "projectId": "string (uuid)",
  "name": "string",
  "states": [
    {
      "id": "string (uuid)",
      "name": "string",
      "order": "integer"
    }
  ]
}
```

### POST /workflows
Create a new workflow.

**Authentication:** Required
- **Roles:** All authenticated

**Request Body:**
```json
{
  "projectId": "string (uuid)",
  "name": "string"
}
```

**Response (201):** Single workflow object

### PUT /workflows/:id/states
Update workflow states.

**Authentication:** Required
- **Roles:** All authenticated

**Path Parameters:**
- `id` (string, uuid, required) - Workflow ID

**Request Body:**
```json
{
  "states": [
    {
      "id": "string (uuid, optional)",
      "name": "string",
      "order": "integer"
    }
  ]
}
```

**Response (200):** Single workflow object

---

## Marketplace

### GET /marketplace/projects
List marketplace project postings.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Freelancers see open postings; company roles see their company's postings

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "companyId": "string (uuid)",
    "createdById": "string (uuid)",
    "title": "string",
    "description": "string",
    "budgetType": "string (hourly|fixed|hybrid)",
    "hourlyMin": "number (optional)",
    "hourlyMax": "number (optional)",
    "fixedMin": "number (optional)",
    "fixedMax": "number (optional)",
    "currency": "string",
    "requiredSkills": ["string"],
    "status": "string (open|closed)",
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)",
    "companyName": "string (optional)",
    "companyLogoUrl": "string (optional)"
  }
]
```

### GET /marketplace/projects/:id
Get a specific marketplace posting.

**Authentication:** Required
- **Roles:** All authenticated with appropriate access

**Path Parameters:**
- `id` (string, uuid, required) - Posting ID

**Response (200):** Single posting object

### POST /marketplace/projects
Create a new marketplace posting.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Request Body:**
```json
{
  "title": "string",
  "description": "string (optional)",
  "budgetType": "string (optional, default: hybrid)",
  "hourlyMin": "number (optional)",
  "hourlyMax": "number (optional)",
  "fixedMin": "number (optional)",
  "fixedMax": "number (optional)",
  "currency": "string (optional, default: UGX)",
  "requiredSkills": ["string"],
  "status": "string (optional, default: open)",
  "companyId": "string (uuid, optional, super_admin only)"
}
```

**Response (201):** Single posting object

### PUT /marketplace/projects/:id
Update a marketplace posting.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Posting ID

**Request Body:** Partial update, any of:
```json
{
  "title": "string",
  "description": "string",
  "budgetType": "string",
  "hourlyMin": "number",
  "hourlyMax": "number",
  "fixedMin": "number",
  "fixedMax": "number",
  "currency": "string",
  "requiredSkills": ["string"],
  "status": "string"
}
```

**Response (200):** Single posting object

### DELETE /marketplace/projects/:id
Delete a marketplace posting.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Posting ID

**Response (204):** No content

### POST /marketplace/projects/:id/apply
Apply to a marketplace posting.

**Authentication:** Required
- **Roles:** Freelancers only

**Path Parameters:**
- `id` (string, uuid, required) - Posting ID

**Request Body:**
```json
{
  "coverLetter": "string",
  "proposedHourlyRate": "number (optional)",
  "proposedFixed": "number (optional)",
  "currency": "string (optional)"
}
```

**Response (201):**
```json
{
  "id": "string (uuid)",
  "postingId": "string (uuid)",
  "companyId": "string (uuid)",
  "freelancerId": "string (uuid)",
  "coverLetter": "string",
  "attachments": [
    {
      "id": "string (uuid)",
      "applicationId": "string (uuid)",
      "name": "string",
      "url": "string"
    }
  ],
  "proposedHourlyRate": "number (optional)",
  "proposedFixed": "number (optional)",
  "currency": "string",
  "status": "string (applied|shortlisted|rejected|withdrawn|hired)",
  "createdAt": "string (iso8601)",
  "updatedAt": "string (iso8601)",
  "linkedProjectId": "string (uuid, optional)"
}
```

### GET /marketplace/projects/:id/applications
List applications for a posting.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Posting ID

**Response (200):** Array of application objects

### GET /marketplace/applications/mine
List current freelancer's applications.

**Authentication:** Required
- **Roles:** Freelancers only

**Response (200):** Array of application objects

### PATCH /marketplace/applications/:id
Update application status.

**Authentication:** Required
- **Roles:** All authenticated
- **Access:** Freelancers can only withdraw; company roles can shortlist/reject

**Path Parameters:**
- `id` (string, uuid, required) - Application ID

**Request Body:**
```json
{
  "status": "string (withdrawn|shortlisted|rejected)"
}
```

**Response (200):** Single application object

### POST /marketplace/applications/:id/hire
Hire a freelancer from application (creates project and assignment).

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Application ID

**Request Body:**
```json
{
  "projectLeadId": "string (uuid)",
  "billingType": "string (hourly|fixed|hybrid)",
  "hourlyRate": "number (optional)",
  "fixedBudget": "number (optional)",
  "currency": "string (optional)",
  "startDate": "string (yyyy-mm-dd, optional)"
}
```

**Response (200):**
```json
{
  "projectId": "string (uuid)",
  "assignmentId": "string (uuid)",
  "freelancerId": "string (uuid)",
  "companyId": "string (uuid)",
  "billingType": "string",
  "currency": "string",
  "hourlyRate": "number (optional)",
  "fixedBudget": "number (optional)"
}
```

### GET /marketplace/applications/:id/files
List application files.

**Authentication:** Required
- **Roles:** All authenticated with appropriate access

**Path Parameters:**
- `id` (string, uuid, required) - Application ID

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "applicationId": "string (uuid)",
    "name": "string",
    "url": "string"
  }
]
```

### POST /marketplace/applications/:id/files/upload
Upload file to application.

**Authentication:** Required
- **Roles:** All authenticated with appropriate access

**Path Parameters:**
- `id` (string, uuid, required) - Application ID

**Request:** Multipart form data
- `file` (file, required) - File to upload
- `name` (string, optional) - Custom filename

**Response (201):** Single application file object

---

## Assignments

### GET /assignments/mine
List current freelancer's assignments.

**Authentication:** Required
- **Roles:** Freelancers, super_admin

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "projectId": "string (uuid)",
    "projectName": "string",
    "companyId": "string (uuid)",
    "companyName": "string",
    "billingType": "string (hourly|fixed|hybrid)",
    "hourlyRate": "number (optional)",
    "fixedBudget": "number (optional)",
    "currency": "string",
    "status": "string (active|completed|terminated)"
  }
]
```

### GET /projects/:projectId/assignments
List assignments by project.

**Authentication:** Required
- **Roles:** All authenticated with project access

**Path Parameters:**
- `projectId` (string, uuid, required) - Project ID

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "projectId": "string (uuid)",
    "companyId": "string (uuid)",
    "freelancerId": "string (uuid)",
    "freelancerName": "string (optional)",
    "billingType": "string (hourly|fixed|hybrid)",
    "hourlyRate": "number (optional)",
    "fixedBudget": "number (optional)",
    "currency": "string",
    "status": "string (active|completed|terminated)"
  }
]
```

---

## Billing

### GET /assignments/:assignmentId/timesheets
List timesheets for an assignment.

**Authentication:** Required
- **Roles:** All authenticated with appropriate access

**Path Parameters:**
- `assignmentId` (string, uuid, required) - Assignment ID

**Query Parameters:**
- `from` (string, yyyy-mm-dd, optional) - Filter by start date
- `to` (string, yyyy-mm-dd, optional) - Filter by end date
- `status` (string, optional) - Filter by status

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "assignmentId": "string (uuid)",
    "companyId": "string (uuid)",
    "freelancerId": "string (uuid)",
    "workDate": "string (yyyy-mm-dd)",
    "minutes": "integer",
    "notes": "string",
    "status": "string (submitted|approved|rejected)",
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)"
  }
]
```

### POST /assignments/:assignmentId/timesheets
Create a timesheet entry.

**Authentication:** Required
- **Roles:** Freelancers only

**Path Parameters:**
- `assignmentId` (string, uuid, required) - Assignment ID

**Request Body:**
```json
{
  "workDate": "string (yyyy-mm-dd)",
  "minutes": "integer",
  "notes": "string"
}
```

**Response (201):** Single timesheet object

### PATCH /timesheets/:id/approve
Approve a timesheet entry.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Timesheet ID

**Response (200):** Single timesheet object with status="approved"

### GET /assignments/:assignmentId/billing-milestones
List billing milestones for an assignment.

**Authentication:** Required
- **Roles:** All authenticated with appropriate access

**Path Parameters:**
- `assignmentId` (string, uuid, required) - Assignment ID

**Response (200):**
```json
[
  {
    "id": "string (uuid)",
    "assignmentId": "string (uuid)",
    "companyId": "string (uuid)",
    "freelancerId": "string (uuid)",
    "title": "string",
    "amount": "number",
    "currency": "string",
    "dueDate": "string (yyyy-mm-dd, optional)",
    "status": "string (pending|approved|invoiced|paid)",
    "approvedAt": "string (iso8601, optional)",
    "createdAt": "string (iso8601)",
    "updatedAt": "string (iso8601)"
  }
]
```

### POST /assignments/:assignmentId/billing-milestones
Create a billing milestone.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `assignmentId` (string, uuid, required) - Assignment ID

**Request Body:**
```json
{
  "title": "string",
  "amount": "number",
  "currency": "string (optional)",
  "dueDate": "string (yyyy-mm-dd, optional)"
}
```

**Response (201):** Single billing milestone object

### PATCH /billing-milestones/:id/approve
Approve a billing milestone.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `id` (string, uuid, required) - Milestone ID

**Response (200):** Single billing milestone object with status="approved"

### POST /assignments/:assignmentId/invoices/generate
Generate invoice for an assignment.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `assignmentId` (string, uuid, required) - Assignment ID

**Request Body:**
```json
{
  "from": "string (yyyy-mm-dd, optional)",
  "to": "string (yyyy-mm-dd, optional)",
  "milestoneIds": ["string (uuid)"]
}
```

**Response (201):** Single invoice object

### POST /companies/:companyId/invoices/generate-bulk
Generate bulk invoices for a company.

**Authentication:** Required
- **Roles:** Company roles only (not freelancers)

**Path Parameters:**
- `companyId` (string, uuid, required) - Company ID

**Request Body:**
```json
{
  "consultants": ["string (uuid)"],
  "from": "string (yyyy-mm-dd)",
  "to": "string (yyyy-mm-dd)",
  "hybridChoice": "string (hourly|fixed)"
}
```

**Response (201):** Array of invoice objects

---

## Health

### GET /health
Health check endpoint.

**Authentication:** None (public)

**Response (200):**
```json
{
  "status": "ok"
}
```

---

## WebSocket

### GET /ws/chat
WebSocket chat hub.

**Authentication:** None (no WS auth for now)

**Connection:** Upgrade to WebSocket protocol

**Message Format:**
```json
{
  "type": "string",
  "roomId": "string",
  "userId": "string"
}
```

