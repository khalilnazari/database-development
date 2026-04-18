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

# Solutions - Creating databsae schema

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
