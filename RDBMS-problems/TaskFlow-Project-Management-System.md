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
