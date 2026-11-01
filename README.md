# University Management System (CSE 3230 - Lab 4)

Spring Boot 3 / Java 21 web app with three roles: **ADMIN**, **TEACHER**, **STUDENT**.

| Role    | Can manage (REST + web UI)        |
|---------|-----------------------------------|
| ADMIN   | admins, teachers and students     |
| TEACHER | teachers only                     |
| STUDENT | students only                     |

Stack: Spring Web, Spring Data JPA, PostgreSQL, Spring Security (BCrypt, form login + HTTP Basic), Thymeleaf, Lombok, Docker.

## Run with Docker (everything together)

```bash
docker compose up --build
```
Open http://localhost:8080. Stop with `docker compose down` (`-v` also deletes the DB volume).

## Run locally (without Docker for the app)

```bash
docker compose up -d db      # only PostgreSQL
mvn spring-boot:run
```
DB settings come from `DB_URL`, `DB_USER`, `DB_PASSWORD` (defaults match `compose.yaml`).

## Demo accounts (created on first start)

| Role    | Email              | Password   |
|---------|--------------------|------------|
| Admin   | admin@kuet.ac.bd   | admin123   |
| Teacher | teacher@kuet.ac.bd | teacher123 |
| Student | student@kuet.ac.bd | student123 |

## REST API

Authenticate with HTTP Basic (or a session cookie from the login form). All bodies/responses are JSON, passwords are BCrypt-hashed and never returned.

| Method | URL                 | Who                       | Success |
|--------|---------------------|---------------------------|---------|
| GET    | /api/{res}          | see below                 | 200     |
| GET    | /api/{res}/{id}     | see below                 | 200     |
| POST   | /api/{res}          | see below                 | 201 + `Location` |
| PUT    | /api/{res}/{id}     | see below (partial update)| 200     |
| DELETE | /api/{res}/{id}     | see below                 | 204     |
| GET    | /api/me             | any logged-in user        | 200     |

`{res}` = `admins` (ADMIN) · `teachers` (ADMIN, TEACHER) · `students` (ADMIN, STUDENT)

Error codes: `400` invalid data, `401` not logged in, `403` role not allowed, `404` not found, `409` duplicate email / deleting or re-emailing yourself.

### Examples

```bash
# admin creates a student
curl -u admin@kuet.ac.bd:admin123 -H "Content-Type: application/json" \
  -d '{"name":"Rahim","email":"rahim@kuet.ac.bd","password":"secret123","department":"CSE","roll":"2307010"}' \
  http://localhost:8080/api/students

# teacher lists teachers (allowed)  /  teacher lists students (403)
curl -u teacher@kuet.ac.bd:teacher123 http://localhost:8080/api/teachers
curl -u teacher@kuet.ac.bd:teacher123 http://localhost:8080/api/students

# update only some fields
curl -u admin@kuet.ac.bd:admin123 -X PUT -H "Content-Type: application/json" \
  -d '{"department":"EEE"}' http://localhost:8080/api/students/3

# delete
curl -u admin@kuet.ac.bd:admin123 -X DELETE http://localhost:8080/api/students/3
```

## Project structure

```
com.example.ums
├── UmsApplication
├── config        SecurityConfig            (RBAC rules, login/logout, BCrypt)
├── controller    ApiController             (REST, /api/**)
│                 AuthController, AdminController, TeacherController, StudentController (Thymeleaf pages)
└── model
    ├── entity      User, Role
    ├── repository  UserRepository
    ├── service     UserService
    └── component   CustomUserDetailsService, DataSeeder
```
