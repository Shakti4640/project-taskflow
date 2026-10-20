# Build a Single-Page Task Manager with Spring Boot and PostgreSQL

## 1. Project overview

Build a web application called **TaskFlow**. It will display tasks in a
browser and allow users to create, view, edit, update, and delete them
without reloading the entire page.

This project teaches:

-   HTTP request methods and REST APIs
-   Spring Boot and Spring MVC
-   Layered backend architecture
-   Spring Data JPA and Hibernate
-   PostgreSQL persistence
-   HTML, CSS, and JavaScript
-   Request validation and error handling
-   Authentication and authorization (in later milestones)
-   Automated testing and deployment (in later milestones)

The initial implementation is a learning prototype. Later sections
identify improvements needed before production use.

## 2. What the application will do

Example dashboard:

  ID   Task                Status        Priority   Actions
  ---- ------------------- ------------- ---------- ---------------
  1    Learn Spring Boot   IN_PROGRESS   HIGH       Edit / Delete
  2    Learn PostgreSQL    PENDING       MEDIUM     Edit / Delete
  3    Build REST API      COMPLETED     HIGH       Edit / Delete

The user can:

-   Create a task.
-   View all tasks.
-   View one task.
-   Edit a task.
-   Partially update a task.
-   Delete a task.
-   Later, search, filter, sort, and paginate tasks.

The browser updates the dashboard through JavaScript requests. It does
not need to reload the entire HTML page for every operation.

## 3. Architecture

``` text
                  BROWSER
             +------------------+
             |   TaskFlow SPA   |
             | HTML + CSS + JS  |
             +--------+---------+
                      |
                      | HTTP requests
                      | GET / POST / PUT
                      | PATCH / DELETE
                      v
             +------------------+
             | TaskController   |
             | Spring MVC        |
             +--------+---------+
                      |
                      v
             +------------------+
             | TaskService      |
             | Business logic   |
             +--------+---------+
                      |
                      v
             +------------------+
             | TaskRepository   |
             | Spring Data JPA  |
             +--------+---------+
                      |
                      v
             +------------------+
             | PostgreSQL       |
             | tasks table      |
             +------------------+
```

### Frontend

HTML defines the page structure, CSS styles it, and JavaScript handles
user interactions. JavaScript uses `fetch()` to send HTTP requests to
Spring Boot.

### Controller

The controller maps HTTP requests to Java methods. For example,
`@GetMapping` handles GET requests and `@PostMapping` handles POST
requests.

### Service

The service contains business rules, such as validating task state and
ensuring that an operation is allowed.

### Repository

The repository accesses persistent data through Spring Data JPA.
Hibernate is the JPA implementation used to translate entity operations
into database operations.

### Database

PostgreSQL stores tasks persistently. Tasks remain in the database after
the application stops and restarts.

## 4. HTTP methods and REST endpoints

The API will use `/api/tasks` as its base path.

### GET --- read data

Retrieve all tasks:

``` http
GET /api/tasks
```

Retrieve one task:

``` http
GET /api/tasks/1
```

Example response:

``` json
[
  {
    "id": 1,
    "title": "Learn Spring Boot",
    "status": "PENDING"
  },
  {
    "id": 2,
    "title": "Learn PostgreSQL",
    "status": "IN_PROGRESS"
  }
]
```

GET retrieves information and should not change the stored resource.

### POST --- create a resource

``` http
POST /api/tasks
Content-Type: application/json
```

Request body:

``` json
{
  "title": "Learn Spring Boot",
  "description": "Build a REST API",
  "status": "PENDING",
  "priority": "HIGH"
}
```

A successful creation should return `201 Created` and the created
resource.

### PUT --- replace a resource

``` http
PUT /api/tasks/1
Content-Type: application/json
```

Request body:

``` json
{
  "title": "Master Spring Boot",
  "description": "Build a production-ready REST API",
  "status": "IN_PROGRESS",
  "priority": "MEDIUM"
}
```

PUT is conventionally used to replace a resource's represented state.
The API should define which fields are required for a full update and
should not silently turn omitted fields into null values.

### PATCH --- partially update a resource

``` http
PATCH /api/tasks/1
Content-Type: application/json
```

Request body:

``` json
{
  "status": "COMPLETED"
}
```

PATCH updates selected fields. The API must define and validate its
patch format.

### DELETE --- delete a resource

``` http
DELETE /api/tasks/1
```

A successful deletion can return `204 No Content`.

### HEAD and OPTIONS

  -----------------------------------------------------------------------
  Method                  Purpose                 Example
  ----------------------- ----------------------- -----------------------
  GET                     Retrieve data           `GET /api/tasks`

  POST                    Create a task           `POST /api/tasks`

  PUT                     Replace a task's state  `PUT /api/tasks/1`

  PATCH                   Update selected fields  `PATCH /api/tasks/1`

  DELETE                  Delete a task           `DELETE /api/tasks/1`

  HEAD                    Retrieve response       `HEAD /api/tasks/1`
                          headers without the     
                          response body           

  OPTIONS                 Discover communication  `OPTIONS /api/tasks`
                          options supported by a  
                          resource                
  -----------------------------------------------------------------------

Focus on GET, POST, PUT, PATCH, and DELETE for the initial CRUD
implementation. Spring MVC and the HTTP infrastructure handle some HEAD
and OPTIONS behavior automatically.

## 5. Create the Spring Boot project

Use Spring Initializr to generate a Maven project.

Suggested configuration:

  Setting       Value
  ------------- ---------------------------------------------------------
  Project       Maven
  Language      Java
  Spring Boot   A stable version offered by Initializr
  Group         `com.taskflow`
  Artifact      `taskflow`
  Packaging     Jar
  Java          A version supported by the selected Spring Boot release

Add these dependencies:

1.  **Spring Web** --- build HTTP endpoints and REST APIs.
2.  **Spring Data JPA** --- interact with relational databases using
    JPA.
3.  **PostgreSQL Driver** --- connect to PostgreSQL.
4.  **Validation** --- validate incoming request data.
5.  **Spring Boot DevTools** --- optional development convenience.

Spring Security can be added in a later milestone when authentication
and authorization are implemented.

## 6. Project structure

The project should eventually have a structure similar to this:

``` text
taskflow/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── taskflow/
        │           ├── TaskflowApplication.java
        │           ├── controller/
        │           │   └── TaskController.java
        │           ├── service/
        │           │   └── TaskService.java
        │           ├── repository/
        │           │   └── TaskRepository.java
        │           ├── entity/
        │           │   └── Task.java
        │           ├── dto/
        │           │   ├── CreateTaskRequest.java
        │           │   ├── UpdateTaskRequest.java
        │           │   └── TaskResponse.java
        │           └── exception/
        └── resources/
            ├── application.properties
            └── static/
                ├── index.html
                ├── css/
                │   └── style.css
                └── js/
                    └── app.js
```

For the initial implementation, Spring Boot will serve the frontend
files from `src/main/resources/static/`. This creates a single
deployable application without a separate frontend server.

## 7. Create the PostgreSQL database

Create the database:

``` sql
CREATE DATABASE taskflow;
```

Configure `src/main/resources/application.properties`:

``` properties
spring.application.name=taskflow

spring.datasource.url=jdbc:postgresql://localhost:5432/taskflow
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=update

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Set the `DB_USERNAME` and `DB_PASSWORD` environment variables before
starting the application. Do not commit real database credentials to
Git.

`spring.jpa.hibernate.ddl-auto=update` is convenient for learning. For
production, use a database migration tool such as Flyway or Liquibase to
manage schema changes explicitly.

## 8. Create the Task entity

Create `src/main/java/com/taskflow/entity/Task.java`:

``` java
package com.taskflow.entity;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "tasks")
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(length = 2000)
    private String description;

    @Column(nullable = false, length = 30)
    private String status = "PENDING";

    @Column(nullable = false, length = 30)
    private String priority = "MEDIUM";

    private LocalDateTime createdAt;

    @PrePersist
    public void onCreate() {
        createdAt = LocalDateTime.now();
    }

    public Task() {
    }

    public Long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public String getPriority() {
        return priority;
    }

    public void setPriority(String priority) {
        this.priority = priority;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }
}
```

This is a simple starting point. Later, replace status and priority
strings with Java enums, add update timestamps and due dates, and
introduce task ownership.

Hibernate inspects the entity and, with the development configuration
above, creates or updates the corresponding database schema. The
generated schema may differ slightly from a hand-written SQL definition.

Conceptually, the table will contain columns for:

``` sql
id BIGINT PRIMARY KEY,
title VARCHAR(200) NOT NULL,
description VARCHAR(2000),
status VARCHAR(30) NOT NULL,
priority VARCHAR(30) NOT NULL,
created_at TIMESTAMP
```

## 9. Create the repository

Create `src/main/java/com/taskflow/repository/TaskRepository.java`:

``` java
package com.taskflow.repository;

import com.taskflow.entity.Task;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TaskRepository
        extends JpaRepository<Task, Long> {
}
```

Spring Data JPA provides common operations, including:

``` java
findAll();
findById(id);
save(task);
delete(task);
deleteById(id);
existsById(id);
```

You do not need to implement the basic repository operations yourself.
Spring Data creates the repository implementation.

For example:

``` java
taskRepository.findAll();
```

retrieves tasks from PostgreSQL through Hibernate and the configured
database connection.

## 10. Create the service layer

Create `src/main/java/com/taskflow/service/TaskService.java`:

``` java
package com.taskflow.service;

import com.taskflow.entity.Task;
import com.taskflow.repository.TaskRepository;

import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

import java.util.List;

@Service
public class TaskService {

    private final TaskRepository repository;

    public TaskService(TaskRepository repository) {
        this.repository = repository;
    }

    public List<Task> getAllTasks() {
        return repository.findAll();
    }

    public Task getTaskById(Long id) {
        return repository.findById(id)
                .orElseThrow(() ->
                        new ResponseStatusException(
                                HttpStatus.NOT_FOUND,
                                "Task not found"
                        )
                );
    }

    public Task createTask(Task task) {
        task.setStatus(
                task.getStatus() == null
                        ? "PENDING"
                        : task.getStatus()
        );

        task.setPriority(
                task.getPriority() == null
                        ? "MEDIUM"
                        : task.getPriority()
        );

        return repository.save(task);
    }

    public Task replaceTask(Long id, Task updatedTask) {
        Task existingTask = getTaskById(id);

        existingTask.setTitle(updatedTask.getTitle());
        existingTask.setDescription(
                updatedTask.getDescription()
        );
        existingTask.setStatus(updatedTask.getStatus());
        existingTask.setPriority(updatedTask.getPriority());

        return repository.save(existingTask);
    }

    public Task updateTaskStatus(Long id, String status) {
        Task existingTask = getTaskById(id);

        existingTask.setStatus(status);

        return repository.save(existingTask);
    }

    public void deleteTask(Long id) {
        Task task = getTaskById(id);
        repository.delete(task);
    }
}
```

This is a learning-oriented implementation, not a production-ready
service. Incoming data should be validated through dedicated DTOs, and
PATCH must validate the requested status before saving it.

Keep responsibilities separate:

-   **Controller:** HTTP request handling.
-   **Service:** application behavior and business rules.
-   **Repository:** persistence.

## 11. Create the REST controller

Create `src/main/java/com/taskflow/controller/TaskController.java`:

``` java
package com.taskflow.controller;

import com.taskflow.entity.Task;
import com.taskflow.service.TaskService;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    private final TaskService service;

    public TaskController(TaskService service) {
        this.service = service;
    }

    // GET: Retrieve all tasks
    @GetMapping
    public List<Task> getAllTasks() {
        return service.getAllTasks();
    }

    // GET: Retrieve one task
    @GetMapping("/{id}")
    public Task getTask(@PathVariable Long id) {
        return service.getTaskById(id);
    }

    // POST: Create a task
    @PostMapping
    public ResponseEntity<Task> createTask(
            @RequestBody Task task) {

        Task createdTask = service.createTask(task);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(createdTask);
    }

    // PUT: Replace a task
    @PutMapping("/{id}")
    public Task replaceTask(
            @PathVariable Long id,
            @RequestBody Task task) {

        return service.replaceTask(id, task);
    }

    // PATCH: Update selected fields
    @PatchMapping("/{id}")
    public Task patchTask(
            @PathVariable Long id,
            @RequestBody Map<String, String> updates) {

        if (updates.size() != 1
                || !updates.containsKey("status")) {
            throw new IllegalArgumentException(
                    "Only the status field can be updated"
            );
        }

        return service.updateTaskStatus(
                id,
                updates.get("status")
        );
    }

    // DELETE: Delete a task
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(
            @PathVariable Long id) {

        service.deleteTask(id);

        return ResponseEntity.noContent().build();
    }
}
```

This demonstrates the five main CRUD operations.

Before treating it as a complete application, improve validation, error
responses, DTO usage, and PATCH semantics. For example, an invalid
status should return a clear `400 Bad Request` rather than being stored
in the database.

Avoid binding arbitrary client JSON directly to a database entity in a
production application. Dedicated request DTOs let you control which
fields clients can change.

## 12. Build the single-page frontend

Create these files under `src/main/resources/static/`.

### A. `index.html`

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>TaskFlow</title>

    <link rel="stylesheet" href="/css/style.css">
</head>

<body>

    <h1>TaskFlow</h1>

    <form id="taskForm">

        <input
            type="text"
            id="title"
            placeholder="Task title"
            required
        >

        <input
            type="text"
            id="description"
            placeholder="Description"
        >

        <select id="priority">
            <option value="LOW">Low</option>
            <option value="MEDIUM" selected>Medium</option>
            <option value="HIGH">High</option>
        </select>

        <button type="submit">Create Task</button>

    </form>

    <p id="message" role="status"></p>

    <h2>My Tasks</h2>

    <div id="taskList"></div>

    <script src="/js/app.js"></script>

</body>
</html>
```

### B. `css/style.css`

``` css
body {
    font-family: Arial, sans-serif;
    max-width: 900px;
    margin: 40px auto;
    padding: 0 20px;
}

form {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-bottom: 25px;
}

input,
select,
button {
    padding: 10px;
}

#taskList {
    display: grid;
    gap: 12px;
}

.task {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 15px;
}

.task button {
    margin-right: 8px;
    cursor: pointer;
}
```

### C. `js/app.js`

``` javascript
const API = "/api/tasks";

const taskForm = document.getElementById("taskForm");
const taskList = document.getElementById("taskList");
const message = document.getElementById("message");

async function request(url, options = {}) {
    const response = await fetch(url, options);

    if (!response.ok) {
        throw new Error(`Request failed: ${response.status}`);
    }

    if (response.status === 204) {
        return null;
    }

    return response.json();
}

async function loadTasks() {
    try {
        const tasks = await request(API);

        taskList.replaceChildren();

        tasks.forEach(task => {
            const card = document.createElement("article");
            card.className = "task";

            const title = document.createElement("h3");
            title.textContent = task.title;

            const description = document.createElement("p");
            description.textContent = task.description || "";

            const status = document.createElement("p");
            status.textContent = `Status: ${task.status}`;

            const priority = document.createElement("p");
            priority.textContent = `Priority: ${task.priority}`;

            const completeButton = document.createElement("button");
            completeButton.textContent = "Complete";

            completeButton.addEventListener("click", async () => {
                try {
                    await request(`${API}/${task.id}`, {
                        method: "PATCH",
                        headers: {
                            "Content-Type": "application/json"
                        },
                        body: JSON.stringify({
                            status: "COMPLETED"
                        })
                    });

                    await loadTasks();
                } catch (error) {
                    message.textContent = error.message;
                }
            });

            const editButton = document.createElement("button");
            editButton.textContent = "Edit";

            editButton.addEventListener("click", async () => {
                const newTitle = prompt(
                    "Enter the new task title:",
                    task.title
                );

                if (newTitle === null || !newTitle.trim()) {
                    return;
                }

                try {
                    await request(`${API}/${task.id}`, {
                        method: "PUT",
                        headers: {
                            "Content-Type": "application/json"
                        },
                        body: JSON.stringify({
                            title: newTitle.trim(),
                            description: task.description || "",
                            status: task.status,
                            priority: task.priority
                        })
                    });

                    await loadTasks();
                } catch (error) {
                    message.textContent = error.message;
                }
            });

            const deleteButton = document.createElement("button");
            deleteButton.textContent = "Delete";

            deleteButton.addEventListener("click", async () => {
                if (!confirm(`Delete "${task.title}"?`)) {
                    return;
                }

                try {
                    await request(`${API}/${task.id}`, {
                        method: "DELETE"
                    });

                    await loadTasks();
                } catch (error) {
                    message.textContent = error.message;
                }
            });

            card.append(
                title,
                description,
                status,
                priority,
                completeButton,
                editButton,
                deleteButton
            );

            taskList.appendChild(card);
        });

        message.textContent = "";
    } catch (error) {
        message.textContent = error.message;
    }
}

taskForm.addEventListener("submit", async event => {
    event.preventDefault();

    const task = {
        title: document.getElementById("title").value.trim(),
        description: document.getElementById("description").value.trim(),
        priority: document.getElementById("priority").value,
        status: "PENDING"
    };

    try {
        await request(API, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(task)
        });

        taskForm.reset();
        document.getElementById("priority").value = "MEDIUM";

        await loadTasks();
    } catch (error) {
        message.textContent = error.message;
    }
});

loadTasks();
```

This frontend supports:

-   GET: load tasks.
-   POST: create tasks.
-   PUT: edit tasks.
-   PATCH: mark tasks completed.
-   DELETE: delete tasks.

The example uses browser prompts for editing to keep the code short.
Replace them with a proper edit form or modal later.

## 13. Why is this a single-page application?

Open the application at:

``` text
http://localhost:8080/
```

Spring Boot serves `index.html`, and the browser loads the CSS and
JavaScript files.

When you create a task, JavaScript sends a request such as:

``` http
POST /api/tasks
```

Spring Boot saves the task in PostgreSQL and returns a response.
JavaScript then calls:

``` javascript
await loadTasks();
```

That function fetches the latest task data and updates the dashboard.

**The browser does not need to request a new HTML page for every
operation.** That is the key SPA behavior.

An SPA can still make many HTTP requests. "Single-page" describes how
the frontend is delivered and updated, not the number of HTTP requests
it makes.

Because the frontend and API are served from the same origin, this
initial version does not need CORS configuration.

## 14. The full request-to-database lifecycle

When you enter a task title and click Create Task:

1.  JavaScript collects the form data into a JavaScript object.
2.  JavaScript serializes the object as JSON.
3.  The browser sends `POST /api/tasks`.
4.  Spring MVC routes the request to the controller.
5.  Spring converts the JSON request body into a Java object.
6.  The controller calls the service.
7.  The service calls the repository.
8.  Spring Data JPA and Hibernate persist the task.
9.  PostgreSQL stores the record and generates its ID.
10. Spring Boot returns `201 Created` and the created task as JSON.
11. JavaScript fetches the latest tasks.
12. The dashboard updates without a full page reload.

``` text
User clicks Save
       |
       v
JavaScript creates JSON
       |
       v
POST /api/tasks
       |
       v
TaskController
       |
       v
TaskService
       |
       v
TaskRepository
       |
       v
PostgreSQL INSERT
       |
       v
201 Created + JSON
       |
       v
JavaScript fetches tasks
       |
       v
Dashboard updates
```

## 15. How to test the application

Test the API independently before testing the frontend. Use Postman or
another HTTP client.

  Test                    HTTP method   Endpoint         Expected result
  ----------------------- ------------- ---------------- ------------------
  Retrieve tasks          GET           `/api/tasks`     `200 OK`
  Create task             POST          `/api/tasks`     `201 Created`
  Retrieve one task       GET           `/api/tasks/1`   `200 OK`
  Replace task            PUT           `/api/tasks/1`   `200 OK`
  Update status           PATCH         `/api/tasks/1`   `200 OK`
  Delete task             DELETE        `/api/tasks/1`   `204 No Content`
  Retrieve deleted task   GET           `/api/tasks/1`   `404 Not Found`

These responses assume that the relevant task exists and the operation
succeeds.

Start the application and open:

``` text
http://localhost:8080/
```

To diagnose failures, open your browser's Developer Tools and inspect
the **Network** tab. Check the request method, URL, request body,
response status, and response body.

## 16. Improvements required before production

The sample is intended to teach the request-response lifecycle. It is
not yet production-ready.

Improve it in the following areas:

-   **DTOs:** Do not bind arbitrary client JSON directly to the database
    entity.
-   **Validation:** Use Bean Validation annotations such as `@NotBlank`,
    `@Size`, and `@NotNull`, with `@Valid` on request parameters.
-   **Status and priority:** Use Java enums and reject invalid values.
-   **PATCH semantics:** Define which fields can be patched and how
    missing, null, and invalid values are handled.
-   **Error handling:** Add centralized exception handling with
    `@RestControllerAdvice` and consistent error response bodies.
-   **Transactions:** Apply transaction boundaries where appropriate,
    especially for multi-step business operations.
-   **Database migrations:** Use Flyway or Liquibase for controlled
    schema changes.
-   **Security:** Add Spring Security, secure password storage, and a
    clear authentication strategy.
-   **Authorization:** Verify task ownership or team permissions on the
    server for every protected operation.
-   **Testing:** Add unit tests, repository tests, and API integration
    tests.
-   **Logging and monitoring:** Record useful diagnostic information
    without logging secrets.
-   **Configuration:** Keep credentials and environment-specific
    settings out of source control.

The current PATCH endpoint only accepts a `status` field, but it does
not yet validate the status value. Add that validation before using the
application beyond local learning.

## 17. Development roadmap

Build the application in three milestones rather than trying to
implement everything at once.

### Milestone 1 --- Working CRUD application

-   Spring Boot backend.
-   PostgreSQL database.
-   GET, POST, PUT, PATCH, and DELETE.
-   Single-page HTML/CSS/JavaScript frontend.
-   Complete request-to-database lifecycle.

### Milestone 2 --- Proper backend engineering

-   DTOs and request validation.
-   Service-layer business rules.
-   Centralized exception handling.
-   Search, filtering, pagination, and sorting.
-   Unit and integration tests.

### Milestone 3 --- Production-oriented features

-   Spring Security.
-   User registration and login.
-   Task ownership and team permissions.
-   Docker and PostgreSQL containerization.
-   Logging and deployment.

A future API can retain the same core endpoints:

``` text
GET    /api/tasks
POST   /api/tasks
GET    /api/tasks/{id}
PUT    /api/tasks/{id}
PATCH  /api/tasks/{id}
DELETE /api/tasks/{id}
```

However, every request that accesses private task data must be
authorized against the current user. Do not rely on the frontend to
enforce access restrictions; users can call the API directly.

## 18. Final recommendation

You do not need React or Angular to build a single-page application.
Plain JavaScript with `fetch()` is sufficient to learn how an SPA
communicates with Spring Boot.

Start by getting the CRUD workflow working. Then add DTOs, validation,
error handling, tests, authentication, authorization, and deployment.
Once you understand the request-response lifecycle, you can replace the
frontend with React without fundamentally changing the REST API.
