## Company Overview

TaskFlow is a SaaS platform that helps teams manage projects, tasks, and collaboration. The company needs a database to support their web application.

### Core Requirements

1. User Management
   Users can register with email, name, and password
   Each user has a profile with optional fields: job title, department, phone number, profile picture
   Users belong to organizations (companies/teams)
   Users can have different roles within an organization: Admin, Manager, Member
   Users can be part of multiple organizations

2. Project Management
   Organizations can create multiple projects
   Each project has: name, description, start date, end date (optional), status (Planning, Active, On Hold, Completed, Cancelled)
   Projects can have multiple team members assigned
   One user acts as the project manager
   Projects can be organized into folders/categories

3. Task Management
   Each project contains multiple tasks
   Task attributes: title, description, priority (Low, Medium, High, Critical), status (To Do, In Progress, In Review, Done)
   Tasks have creation date, due date, and completion date
   Tasks can be assigned to one or more users
   Tasks can have subtasks (parent-child relationship)
   Tasks can have dependencies (a task cannot start until another is completed)

4. Time Tracking
   Users can log time spent on tasks
   Time entries include: date, duration, description of work done
   Users can start/stop timers for real-time tracking
   Weekly time reports should be available

5. Comments and Attachments
   Users can comment on tasks and projects
   Comments support @mentions to notify other users
   Files can be attached to tasks (images, documents, etc.)
   Track who uploaded what and when

6. Notifications
   System tracks notifications for users
   Notification types: task assigned, @mention, deadline approaching, task completed
   Notifications can be read/unread
   Users can have notification preferences (email, in-app, both)

7. Reporting Requirements
   Project progress reports (completed vs pending tasks)
   Team workload reports
   Time tracking summaries by user, project, or date range
   Overdue tasks reports

### Business Rules

- A user must belong to at least one organization
- Projects must have at least one team member
- Tasks cannot be created without a parent project
- Task dependencies cannot create circular references
- Only organization admins can delete projects
- Completed tasks cannot be edited (except for comments)
- Users can only see projects and tasks from their organizations

### Sample Queries the System Needs to Support

- Get all tasks assigned to a specific user that are overdue
- Calculate the percentage of completed tasks in a project
- Find the most productive users based on tasks completed in the last 30 days
- Get all projects with their task counts (grouped by status)
- Find tasks that are blocked by incomplete dependencies
- Generate a timeline of all project activities

---

## Your Challenge

Design a database schema that satisfies these requirements. Consider:

- Tables and relationships (primary keys, foreign keys)
- Data types for each field
- Indexes for query performance
- Constraints to enforce business rules
- Normalization (avoiding redundancy while maintaining query efficiency)

#### Bonus Questions:

- How would you handle soft deletes vs hard deletes?
- What auditing requirements might exist?
- How would you design for scalability if the system grows to millions of records?
- What security considerations should be implemented at the database level?

---

## Solutions - Creating databsae schema

As you can see the Step 1 (Gather Business Requirements), Step 2 (Analyze and Define the Scope), Step 3 (Identify Entities and Attributes) are done.

### Entity indentification

I found these are the nouns in business requirements:

- Organization (department / teams)
- Project
- User
- Task
- Time Tacker
- Notification

### Entity attributes indentification

- Organization
  No attributes are give, let's add possible/required attributes:

- Project
  Attributes:
  - name
  - desription
  - state date
  - end date
  - status
  - members
  - manager
  - categoriy
  - comments

- User
  Attributes:
  - email
  - name
  - password
  - job title
  - phone
  - porifle picture
  - role

- Task
  Attributes:
  - title
  - description
  - priority
  - status
  - creation date
  - due date
  - completion date
  - assigned to member
  - sub tasks
  - depend on tasks
  - comments

- Tracker
  Attributes:
  - duration (time spent on a task)
  - time spent per week

- Notification
  Attributes:
  - types
  - read status
  - preferences

- Reporting requirements
  This could be a featuer of the app that collect the data from different entity and sent it user.

### Determining the keys

- Organization
  id - PK
  namae - unique

  Candidate Keys: id, namae
  Super Keys: Any set containing at least one candidate key. Examples:
  (id), (namae), (id, namae), (id, namae, any other attribute)

- Project
  id - PK
  organization_id - FK
  manager_id - FK
  category_id - FK
  name - unique

  Candidate Keys: id, name
  Super Keys: Any set containing id or name. Examples:
  (id), (name), (id, organization_id), (name, manager_id), (id, name, category_id)

- User
  id - PK
  project_id - FK
  email - unique

  Candidate Keys: id, email
  Super Keys: Any set containing id or email. Examples:
  (id), (email), (id, project_id), (email, project_id)

- Task
  id - PK
  user_id - FK
  project_id - FK
  created_by_user - FK

  Candidate Keys: id only
  (user_id, project_id) could be a candidate if each user has only one task per project, but requirement does not specify. From given constraints, only id is explicitly unique.
  Super Keys: Any set containing id. Examples:
  (id), (id, user_id), (id, project_id), (id, created_by_user)

- Tracker
  id - PK
  user_id - FK
  task_id - FK
  project_id - FK

  Candidate Keys: id only
  (No other unique constraint provided)
  Super Keys: Any set containing id. Examples:
  (id), (id, user_id), (id, task_id), (id, project_id)

- User and Organnizatin (user_organization)
  user_id - FK
  organzation_id - FK

  Candidate Keys: (user_id, organization_id) – assuming a user cannot belong to the same organization twice
  Super Keys: Any set containing both user_id and organization_id. Examples:
  (user_id, organization_id), (user_id, organization_id, any other attribute)

- User and Organnizatin (task_assigned_user)
  user_id - FK
  task_id - FK

  Candidate Keys: (user_id, task_id) – assuming a user cannot be assigned to the same task twice

  Super Keys: Any set containing both user_id and task_id. Examples:
  (user_id, task_id), (user_id, task_id, any other attribute)

### Creating the relations between entities

Based solely on the entity definitions and keys you provided, here are the **relationships** between the entities.

#### Organization ↔ User (via user_organization)

- **Relationship:** Many-to-Many
- **Description:** An organization can have many users. A user can belong to many organizations.
- **Foreign Keys:** `user_organization.user_id` references `User.id`, `user_organization.organization_id` references `Organization.id`

#### Organization → Project

- **Relationship:** One-to-Many
- **Description:** An organization can have many projects. A project belongs to one organization.
- **Foreign Key:** `Project.organization_id` references `Organization.id`

#### User (as manager) → Project

- **Relationship:** One-to-Many
- **Description:** A user (acting as manager) can manage many projects. A project has one manager.
- **Foreign Key:** `Project.manager_id` references `User.id`

#### Project → Task

- **Relationship:** One-to-Many
- **Description:** A project can have many tasks. A task belongs to one project.
- **Foreign Key:** `Task.project_id` references `Project.id`

#### User (as creator) → Task

- **Relationship:** One-to-Many
- **Description:** A user can create many tasks. A task is created by one user.
- **Foreign Key:** `Task.created_by_user` references `User.id`

#### User (as assignee) ↔ Task (via task_assigned_user)

- **Relationship:** Many-to-Many
- **Description:** A user can be assigned to many tasks. A task can be assigned to many users.
- **Foreign Keys:** `task_assigned_user.user_id` references `User.id`, `task_assigned_user.task_id` references `Task.id`

#### Project ↔ User (via Project.member? Not directly in schema)

- Based on your provided schema, there is no direct `project_member` table. However, `User` has a `project_id` FK, which implies:
- **Relationship:** One-to-Many (from Project to User)
- **Description:** A project can have many users (members). A user belongs to one project.
- **Foreign Key:** `User.project_id` references `Project.id`

#### Tracker ↔ User

- **Relationship:** Many-to-One
- **Description:** A tracker entry belongs to one user. A user can have many tracker entries.
- **Foreign Key:** `Tracker.user_id` references `User.id`

#### Tracker ↔ Task

- **Relationship:** Many-to-One
- **Description:** A tracker entry belongs to one task. A task can have many tracker entries.
- **Foreign Key:** `Tracker.task_id` references `Task.id`

#### Tracker ↔ Project

- **Relationship:** Many-to-One
- **Description:** A tracker entry belongs to one project. A project can have many tracker entries.
- **Foreign Key:** `Tracker.project_id` references `Project.id`

#### Summary Table of Relationships

| From Entity  | To Entity | Relationship Type       | Via Foreign Key           |
| ------------ | --------- | ----------------------- | ------------------------- |
| Organization | User      | Many-to-Many            | `user_organization`       |
| Organization | Project   | One-to-Many             | `Project.organization_id` |
| User         | Project   | One-to-Many (manager)   | `Project.manager_id`      |
| Project      | Task      | One-to-Many             | `Task.project_id`         |
| User         | Task      | One-to-Many (creator)   | `Task.created_by_user`    |
| User         | Task      | Many-to-Many (assignee) | `task_assigned_user`      |
| Project      | User      | One-to-Many (member)    | `User.project_id`         |
| User         | Tracker   | One-to-Many             | `Tracker.user_id`         |
| Task         | Tracker   | One-to-Many             | `Tracker.task_id`         |
| Project      | Tracker   | One-to-Many             | `Tracker.project_id`      |

### Normalization

#### 1NF

First normalizatio nform requires below:

- Atomic values (no repeating groups or arrays)
- A primary key defined for each relation

**Organization**

| Attribute | Type   | Constraints   |
| --------- | ------ | ------------- |
| id        | PK     | Unique        |
| namae     | Unique | Atomic string |

**Project**

| Attribute       | Type                          | Constraints   |
| --------------- | ----------------------------- | ------------- |
| id              | PK                            | Unique        |
| organization_id | FK → Organization.id          | Atomic        |
| manager_id      | FK → User.id                  | Atomic        |
| category_id     | FK → (assumed Category table) | Atomic        |
| name            | Unique                        | Atomic string |

**User**

| Attribute  | Type            | Constraints   |
| ---------- | --------------- | ------------- |
| id         | PK              | Unique        |
| project_id | FK → Project.id | Atomic        |
| email      | Unique          | Atomic string |

**Task**

| Attribute       | Type            | Constraints |
| --------------- | --------------- | ----------- |
| id              | PK              | Unique      |
| user_id         | FK → User.id    | Atomic      |
| project_id      | FK → Project.id | Atomic      |
| created_by_user | FK → User.id    | Atomic      |

**Tracker**

| Attribute  | Type            | Constraints |
| ---------- | --------------- | ----------- |
| id         | PK              | Unique      |
| user_id    | FK → User.id    | Atomic      |
| task_id    | FK → Task.id    | Atomic      |
| project_id | FK → Project.id | Atomic      |

**user_organization**

| Attribute       | Type                 | Constraints          |
| --------------- | -------------------- | -------------------- |
| user_id         | FK → User.id         | Part of composite PK |
| organization_id | FK → Organization.id | Part of composite PK |

**Primary Key:** (`user_id`, `organization_id`)

**task_assigned_user**

| Attribute | Type         | Constraints          |
| --------- | ------------ | -------------------- |
| user_id   | FK → User.id | Part of composite PK |
| task_id   | FK → Task.id | Part of composite PK |

**Primary Key:** (`user_id`, `task_id`)

**Summary of 1NF Compliance**

| Relation           | Repeating Groups Removed? | Atomic Values? | Primary Key Defined?               | In 1NF? |
| ------------------ | ------------------------- | -------------- | ---------------------------------- | ------- |
| Organization       | Yes                       | Yes            | Yes (`id`)                         | ✓       |
| Project            | Yes                       | Yes            | Yes (`id`)                         | ✓       |
| User               | Yes                       | Yes            | Yes (`id`)                         | ✓       |
| Task               | Yes                       | Yes            | Yes (`id`)                         | ✓       |
| Tracker            | Yes                       | Yes            | Yes (`id`)                         | ✓       |
| user_organization  | Yes                       | Yes            | Yes (`user_id`, `organization_id`) | ✓       |
| task_assigned_user | Yes                       | Yes            | Yes (`user_id`, `task_id`)         | ✓       |

#### 2NF

For 2NF we need to make sure below rules meets.

- The relation must already be in 1NF
- No partial dependencies (no non-prime attribute depends on only part of a composite primary key)

##### Analysis of Partial Dependencies

**Organization**

- **Primary Key:** `id` (single attribute)
- **No partial dependencies possible** (single-attribute PK)
- **Status:** Already in 2NF

**Project**

- **Primary Key:** `id` (single attribute)
- **No partial dependencies possible**
- **Status:** Already in 2NF

**User**

- **Primary Key:** `id` (single attribute)
- **No partial dependencies possible**
- **Status:** Already in 2NF

**Task**

- **Primary Key:** `id` (single attribute)
- **No partial dependencies possible**
- **Status:** Already in 2NF

**Tracker**

- **Primary Key:** `id` (single attribute)
- **No partial dependencies possible**
- **Status:** Already in 2NF

**user_organization**

- **Primary Key:** (`user_id`, `organization_id`) - composite key
- **Non-prime attributes:** None
- **No partial dependencies possible** (no non-prime attributes to depend on part of the key)
- **Status:** Already in 2NF

**task_assigned_user**

- **Primary Key:** (`user_id`, `task_id`) - composite key
- **Non-prime attributes:** None
- **No partial dependencies possible** (no non-prime attributes to depend on part of the key)
- **Status:** Already in 2NF

#### 2NF Result

**No decompositions are required.** All relations already satisfy 2NF because:

| Relation           | Primary Key Type                         | Non-prime Attributes                                   | Partial Dependencies?        | In 2NF? |
| ------------------ | ---------------------------------------- | ------------------------------------------------------ | ---------------------------- | ------- |
| Organization       | Single attribute (`id`)                  | None (only PK and unique constraint)                   | No                           | ✓       |
| Project            | Single attribute (`id`)                  | `organization_id`, `manager_id`, `category_id`, `name` | No (single-attribute PK)     | ✓       |
| User               | Single attribute (`id`)                  | `project_id`, `email`                                  | No (single-attribute PK)     | ✓       |
| Task               | Single attribute (`id`)                  | `user_id`, `project_id`, `created_by_user`             | No (single-attribute PK)     | ✓       |
| Tracker            | Single attribute (`id`)                  | `user_id`, `task_id`, `project_id`                     | No (single-attribute PK)     | ✓       |
| user_organization  | Composite (`user_id`, `organization_id`) | None                                                   | No (no non-prime attributes) | ✓       |
| task_assigned_user | Composite (`user_id`, `task_id`)         | None                                                   | No (no non-prime attributes) | ✓       |

#### Final 2NF Relations (Identical to 1NF)

**Organization**

- `id` (PK)
- `namae`

**Project**

- `id` (PK)
- `organization_id` (FK)
- `manager_id` (FK)
- `category_id` (FK)
- `name`

**User**

- `id` (PK)
- `project_id` (FK)
- `email`

**Task**

- `id` (PK)
- `user_id` (FK)
- `project_id` (FK)
- `created_by_user` (FK)

**Tracker**

- `id` (PK)
- `user_id` (FK)
- `task_id` (FK)
- `project_id` (FK)

**user_organization**

- `user_id` (FK) - part of composite PK
- `organization_id` (FK) - part of composite PK

**task_assigned_user**

- `user_id` (FK) - part of composite PK
- `task_id` (FK) - part of composite PK

#### 3NF

We need to ensure below rules meet for 3NF:

- The relation is in 2NF
- No transitive dependencies (no non-prime attribute depends on another non-prime attribute)

##### Analysis of Each Relation

**Organization**

- **Primary Key:** `id`
- **Candidate Keys:** `id`, `namae`
- **Non-prime attributes:** None (only key attributes)
- **Transitive dependencies:** None
- **Status:** Already in 3NF

**Project**

- **Primary Key:** `id`
- **Candidate Keys:** `id`, `name`
- **Non-prime attributes:** `organization_id`, `manager_id`, `category_id`
- **Functional dependencies assumed:** `id` → all attributes; `name` → all attributes
- **Transitive dependencies:** None, because no non-prime attribute determines another non-prime attribute (no FDs like `organization_id` → `manager_id` are given)
- **Status:** Already in 3NF

**User**

- **Primary Key:** `id`
- **Candidate Keys:** `id`, `email`
- **Non-prime attributes:** `project_id`
- **Transitive dependencies:** None (`project_id` does not determine any other non-prime attribute)
- **Status:** Already in 3NF

**Task**

- **Primary Key:** `id`
- **Candidate Keys:** `id` (assuming no other unique constraints given)
- **Non-prime attributes:** `user_id`, `project_id`, `created_by_user`
- **Transitive dependencies:** None (no FD among non-prime attributes)
- **Status:** Already in 3NF

**Tracker**

- **Primary Key:** `id`
- **Candidate Keys:** `id`
- **Non-prime attributes:** `user_id`, `task_id`, `project_id`
- **Transitive dependencies:** None
- **Status:** Already in 3NF

**user_organization**

- **Primary Key:** (`user_id`, `organization_id`)
- **Non-prime attributes:** None
- **Transitive dependencies:** None
- **Status:** Already in 3NF

**task_assigned_user**

- **Primary Key:** (`user_id`, `task_id`)
- **Non-prime attributes:** None
- **Transitive dependencies:** None
- **Status:** Already in 3NF

##### Final 3NF Schema

| Relation           | Attributes                                         | Primary Key                |
| ------------------ | -------------------------------------------------- | -------------------------- |
| Organization       | id, namae                                          | id                         |
| Project            | id, organization_id, manager_id, category_id, name | id                         |
| User               | id, project_id, email                              | id                         |
| Task               | id, user_id, project_id, created_by_user           | id                         |
| Tracker            | id, user_id, task_id, project_id                   | id                         |
| user_organization  | user_id, organization_id                           | (user_id, organization_id) |
| task_assigned_user | user_id, task_id                                   | (user_id, task_id)         |

**No decomposition was necessary** because the schema already satisfies 3NF based on the given constraints and absence of transitive dependencies.

### Consceptual data model diagram

I used dbdiagram.io to generate the diagram
LINK: https://dbdiagram.io/d/TaskFlow-data-model-69e438f20aa78f6bc109ea68

![TaskFlow database caceptual data model](/RDBMS-problems/diagrams/TaskFlow-Database-Data-mdel.png)

```dbml
// Conceptual Data Model for TaskFlow
// Crow's foot notation – readable by business stakeholders

Table Organization {
  id integer [pk, unique]           // unique identifier
  namae string [unique]             // organization name
}

Table User {
  id integer [pk, unique]
  email string [unique]
  name string
  password string
  job_title string [null]           // optional
  department string [null]          // optional
  phone_number string [null]        // optional
  profile_picture string [null]     // optional
  project_id integer [ref: > Project.id]  // each user belongs to one project (simplified)
}

Table Project {
  id integer [pk, unique]
  name string [unique]
  description string
  start_date date
  end_date date [null]
  status string                    // Planning, Active, On Hold, Completed, Cancelled
  organization_id integer [ref: > Organization.id]
  manager_id integer [ref: > User.id]
  category_id integer [ref: > Category.id]   // folder/category
}

Table Category {
  id integer [pk]
  name string
  project_id integer [ref: > Project.id]
}

Table Task {
  id integer [pk, unique]
  title string
  description string
  priority string                  // Low, Medium, High, Critical
  status string                    // To Do, In Progress, In Review, Done
  creation_date date
  due_date date
  completion_date date [null]
  user_id integer [ref: > User.id]           // assigned user (one)
  project_id integer [ref: > Project.id]
  created_by_user integer [ref: > User.id]
}

Table Tracker {
  id integer [pk, unique]
  date date
  duration integer                  // in minutes or hours
  description_of_work_done string
  user_id integer [ref: > User.id]
  task_id integer [ref: > Task.id]
  project_id integer [ref: > Project.id]
}

// Junction: User <-> Organization (many-to-many)
Table UserOrganization {
  user_id integer [ref: > User.id]
  organization_id integer [ref: > Organization.id]
  role string                      // Admin, Manager, Member
  indexes {
    (user_id, organization_id) [pk]
  }
}

// Junction: User <-> Task (many-to-many for assignments)
Table TaskAssignedUser {
  user_id integer [ref: > User.id]
  task_id integer [ref: > Task.id]
  indexes {
    (user_id, task_id) [pk]
  }
}

// Additional entities from business requirement

Table Comment {
  id integer [pk]
  content text
  created_at datetime
  user_id integer [ref: > User.id]
  task_id integer [ref: > Task.id]      // comment on a task
  project_id integer [ref: > Project.id] // comment on a project (optional)
}

Table Attachment {
  id integer [pk]
  file_url string
  uploaded_by integer [ref: > User.id]
  uploaded_at datetime
  task_id integer [ref: > Task.id]
}

Table Notification {
  id integer [pk]
  type string                       // task assigned, @mention, deadline approaching, task completed
  is_read boolean
  user_id integer [ref: > User.id]
}

Table NotificationPreference {
  user_id integer [pk, ref: > User.id]
  email boolean
  in_app boolean
  both boolean
}

// Relationships (crow's foot notation is expressed via ref arrows above)
// For visual clarity, the following relationships are already captured by foreign keys:

// Organization 1---* Project
// User (manager) 1---* Project
// Project 1---* Task
// User (creator) 1---* Task
// User *---* Organization (via UserOrganization)
// User *---* Task (via TaskAssignedUser)
// Project 1---* Tracker
// Task 1---* Tracker
// Task 1---* Comment
// Task 1---* Attachment
// User 1---* Notification
// User 1---1 NotificationPreference
```

### Logical ER diagram

I used dbdiagram.io to generate the diagram
LINK: https://dbdiagram.io/d/TaskFlow-Logical-ER-diagram-69e440c3a5db712fe58924ae

![TaskFlow database Logical ER diagram](/RDBMS-problems/diagrams/TaskFlow-Database-logicl-ER-diagram.png)

dbdiagram.io quryies

```dbml
// Logical Data Model for TaskFlow
// Crow's Foot notation – ready for implementation

// ---------- Enumerated Types (as comments for clarity) ----------
// Project.status: 'Planning', 'Active', 'On Hold', 'Completed', 'Cancelled'
// Task.priority: 'Low', 'Medium', 'High', 'Critical'
// Task.status: 'To Do', 'In Progress', 'In Review', 'Done'
// UserOrganization.role: 'Admin', 'Manager', 'Member'
// Notification.type: 'task assigned', '@mention', 'deadline approaching', 'task completed'

// ---------- Entities ----------

Table Organization {
  id integer [pk, unique]
  name string [unique]                       // organization name
}

Table User {
  id integer [pk, unique]
  email string [unique]
  name string
  password string
  job_title string [null]                    // optional
  department string [null]                   // optional
  phone_number string [null]                 // optional
  profile_picture string [null]              // optional
}

Table UserOrganization {
  user_id integer [ref: > User.id]
  organization_id integer [ref: > Organization.id]
  role string                                 // Admin, Manager, Member
  indexes {
    (user_id, organization_id) [pk]          // composite primary key
  }
}

Table Project {
  id integer [pk, unique]
  name string [unique]
  description string
  start_date date
  end_date date [null]
  status string                              // Planning, Active, On Hold, Completed, Cancelled
  organization_id integer [ref: > Organization.id, not null]
  manager_id integer [ref: > User.id, not null]   // project manager
  category_id integer [ref: > Category.id, null]  // folder/category
}

Table Category {
  id integer [pk]
  name string
  organization_id integer [ref: > Organization.id, not null]  // categories belong to an org
}

Table Task {
  id integer [pk, unique]
  title string
  description string
  priority string                            // Low, Medium, High, Critical
  status string                              // To Do, In Progress, In Review, Done
  creation_date date
  due_date date
  completion_date date [null]
  project_id integer [ref: > Project.id, not null]
  parent_task_id integer [ref: > Task.id, null]   // for subtasks (self-reference)
}

Table TaskAssignment {
  user_id integer [ref: > User.id]
  task_id integer [ref: > Task.id]
  indexes {
    (user_id, task_id) [pk]                  // composite primary key
  }
}

Table TaskDependency {
  id integer [pk]
  task_id integer [ref: > Task.id, not null]
  depends_on_task_id integer [ref: > Task.id, not null]
  // business rule: no circular references (enforced by application)
}

Table TimeEntry {
  id integer [pk]
  date date
  duration decimal                             // hours or minutes
  description_of_work text
  user_id integer [ref: > User.id, not null]
  task_id integer [ref: > Task.id, not null]
}

Table Timer {
  id integer [pk]
  start_datetime datetime
  end_datetime datetime [null]                 // null = timer still running
  user_id integer [ref: > User.id, not null]
  task_id integer [ref: > Task.id, not null]
}

Table Comment {
  id integer [pk]
  content text
  created_at datetime
  user_id integer [ref: > User.id, not null]
  task_id integer [ref: > Task.id, null]       // comment on a task
  project_id integer [ref: > Project.id, null] // comment on a project
  // at least one of task_id or project_id must be non‑null (app rule)
}

Table Mention {
  id integer [pk]
  comment_id integer [ref: > Comment.id, not null]
  mentioned_user_id integer [ref: > User.id, not null]
}

Table Attachment {
  id integer [pk]
  file_url string
  uploaded_by integer [ref: > User.id, not null]
  uploaded_at datetime
  task_id integer [ref: > Task.id, not null]
}

Table Notification {
  id integer [pk]
  type string                                 // task assigned, @mention, deadline approaching, task completed
  is_read boolean
  created_at datetime
  user_id integer [ref: > User.id, not null]
  task_id integer [ref: > Task.id, null]      // for task‑related notifications
  comment_id integer [ref: > Comment.id, null] // for @mention notifications
}

Table NotificationPreference {
  user_id integer [pk, ref: > User.id]
  email_notifications boolean
  in_app_notifications boolean
}

// ---------- Relationship Summary (Crow's Foot cardinalities are implied by foreign keys above) ----------
// Organization 1---* Project
// Organization 1---* Category
// User 1---* Project (as manager)
// Project 1---* Task
// Task 1---* Task (as subtask, self-reference)
// User *---* Task (via TaskAssignment)
// Task 1---* TaskDependency (as dependent) and 1---* TaskDependency (as prerequisite)
// User 1---* TimeEntry
// Task 1---* TimeEntry
// User 1---* Timer
// Task 1---* Timer
// User 1---* Comment
// Task 0---* Comment
// Project 0---* Comment
// Comment 1---* Mention
// User 1---* Mention (as mentioned user)
// User 1---* Attachment (as uploader)
// Task 1---* Attachment
// User 1---* Notification
// Task 0---* Notification (optional)
// Comment 0---* Notification (optional)
// User 1---1 NotificationPreference
```

### PosgreSQL Queries

I purposefully added the FK constraint after tables creation by altering the tables. But we can definitely put the at table creaton time.

```sql
CREATE TABLE "Organization" (
  "id" integer UNIQUE PRIMARY KEY,
  "namae" string UNIQUE
);

CREATE TABLE "User" (
  "id" integer UNIQUE PRIMARY KEY,
  "email" string UNIQUE,
  "name" string,
  "password" string,
  "job_title" string,
  "department" string,
  "phone_number" string,
  "profile_picture" string,
  "project_id" integer
);

CREATE TABLE "Project" (
  "id" integer UNIQUE PRIMARY KEY,
  "name" string UNIQUE,
  "description" string,
  "start_date" date,
  "end_date" date,
  "status" string,
  "organization_id" integer,
  "manager_id" integer,
  "category_id" integer
);

CREATE TABLE "Category" (
  "id" integer PRIMARY KEY,
  "name" string,
  "project_id" integer
);

CREATE TABLE "Task" (
  "id" integer UNIQUE PRIMARY KEY,
  "title" string,
  "description" string,
  "priority" string,
  "status" string,
  "creation_date" date,
  "due_date" date,
  "completion_date" date,
  "user_id" integer,
  "project_id" integer,
  "created_by_user" integer
);

CREATE TABLE "Tracker" (
  "id" integer UNIQUE PRIMARY KEY,
  "date" date,
  "duration" integer,
  "description_of_work_done" string,
  "user_id" integer,
  "task_id" integer,
  "project_id" integer
);

CREATE TABLE "UserOrganization" (
  "user_id" integer,
  "organization_id" integer,
  "role" string,
  PRIMARY KEY ("user_id", "organization_id")
);

CREATE TABLE "TaskAssignedUser" (
  "user_id" integer,
  "task_id" integer,
  PRIMARY KEY ("user_id", "task_id")
);

CREATE TABLE "Comment" (
  "id" integer PRIMARY KEY,
  "content" text,
  "created_at" datetime,
  "user_id" integer,
  "task_id" integer,
  "project_id" integer
);

CREATE TABLE "Attachment" (
  "id" integer PRIMARY KEY,
  "file_url" string,
  "uploaded_by" integer,
  "uploaded_at" datetime,
  "task_id" integer
);

CREATE TABLE "Notification" (
  "id" integer PRIMARY KEY,
  "type" string,
  "is_read" boolean,
  "user_id" integer
);

CREATE TABLE "NotificationPreference" (
  "user_id" integer PRIMARY KEY,
  "email" boolean,
  "in_app" boolean,
  "both" boolean
);

ALTER TABLE "User" ADD FOREIGN KEY ("project_id") REFERENCES "Project" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Project" ADD FOREIGN KEY ("organization_id") REFERENCES "Organization" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Project" ADD FOREIGN KEY ("manager_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Project" ADD FOREIGN KEY ("category_id") REFERENCES "Category" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Category" ADD FOREIGN KEY ("project_id") REFERENCES "Project" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Task" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Task" ADD FOREIGN KEY ("project_id") REFERENCES "Project" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Task" ADD FOREIGN KEY ("created_by_user") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Tracker" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Tracker" ADD FOREIGN KEY ("task_id") REFERENCES "Task" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Tracker" ADD FOREIGN KEY ("project_id") REFERENCES "Project" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "UserOrganization" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "UserOrganization" ADD FOREIGN KEY ("organization_id") REFERENCES "Organization" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "TaskAssignedUser" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "TaskAssignedUser" ADD FOREIGN KEY ("task_id") REFERENCES "Task" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Comment" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Comment" ADD FOREIGN KEY ("task_id") REFERENCES "Task" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Comment" ADD FOREIGN KEY ("project_id") REFERENCES "Project" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Attachment" ADD FOREIGN KEY ("uploaded_by") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Attachment" ADD FOREIGN KEY ("task_id") REFERENCES "Task" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "Notification" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
ALTER TABLE "NotificationPreference" ADD FOREIGN KEY ("user_id") REFERENCES "User" ("id") DEFERRABLE INITIALLY IMMEDIATE;
```
