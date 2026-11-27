# Student Management System

A RESTful backend API built with **Spring Boot 4.1.1**, **Java 21**, **Spring Data JPA**, **Spring Security**, and **MySQL** for managing students and their enrolled courses.

---

### IMPORTANT: 
THIS README FILE CONTAINS THINGS LIKE *PROJECT OVERVIEW*, *PROJECT STRUCTURE*, *TECH STACK*,*HOW TOR RUN THIS APPLICATION*,*ANNOTAION USED IN PROJECT(FOR THEORY PURPOSES)*,*LIST OF API ENDPOINTS IN A SINGLE PLACE*,AND *EXAMPLE REQUESTS* TILL LN 220.
AFTER LN 220, THIS FILE CONTAINS *LAB WISE/EXPERIMENT WISE* WORK DONE IN EACH LAB. LAB WISE IMPLEMENTATION OF PROJECT AND LEARNIG OF THING IN SIMPLE MANNER.

## What This Project Does

This application provides a backend service to manage **Students** and **Courses** with a one-to-many relationship (`One Student -> Many Courses`), input validation, centralized error handling, pagination/sorting, and HTTP Basic authentication:

- **Student CRUD Operations:** Create, read (all or by ID), update, and delete student records stored in the `students` table in MySQL.
- **Student Search, Pagination & Sorting:**
  - Search students by exact name, partial name keyword, or email address.
  - Retrieve paginated lists of students using Spring Data's `Pageable` support.
  - Retrieve all students sorted alphabetically by name.
- **Course Enrollment (`Student` <-> `Course`):**
  - Manage `Course` entities (`courseName`, `credits`) linked to a `Student`.
  - List all courses and assign new courses to an existing student.
  - Uses cascading (`CascadeType.ALL`) and `orphanRemoval = true` so deleting or updating a student manages its associated courses cleanly, while avoiding infinite JSON recursion with Jackson managed/back references.
- **Input Validation & Global Error Handling:**
  - Validates student `name` and `email` on incoming requests.
  - Intercepts validation failures globally and returns a clean `400 Bad Request` JSON map of `{ "fieldName": "errorMessage" }`.
- **Security (HTTP Basic Auth):**
  - `GET /hello` is publicly accessible without authentication.
  - All other endpoints require HTTP Basic Authentication.

---

## Tech Stack

- **Language:** Java 21
- **Framework:** Spring Boot `4.1.1`
  - Spring Web MVC (`spring-boot-starter-webmvc`)
  - Spring Data JPA / Hibernate (`spring-boot-starter-data-jpa`)
  - Spring Validation (`spring-boot-starter-validation`)
  - Spring Security (`spring-boot-starter-security`)
  - Spring Boot DevTools (`spring-boot-devtools`)
- **Database:** MySQL (`mysql-connector-j`)
- **Build Tool:** Maven (with Maven Wrapper `mvnw` / `mvnw.cmd`)
- **Testing:** JUnit & `MockMvc` (`spring-boot-starter-webmvc-test`)

---

## Project Structure

```text
student-management/
├── pom.xml
├── mvnw / mvnw.cmd
└── src/
    ├── main/
    │   ├── java/com/example/ucs067/studentmanagement/
    │   │   ├── StudentManagementApplication.java   # Application entry point
    │   │   ├── Student.java                        # Student JPA entity & validation rules
    │   │   ├── Course.java                         # Course JPA entity
    │   │   ├── config/
    │   │   │   └── SecurityConfig.java             # HTTP Basic auth & endpoint rules
    │   │   ├── controller/
    │   │   │   └── HelloController.java            # REST API endpoints for students & courses
    │   │   ├── exception/
    │   │   │   └── GlobalExceptionHandler.java     # Centralized validation error responses
    │   │   └── repository/
    │   │       ├── StudentRepository.java          # Student JPA & derived query methods
    │   │       └── CourseRepository.java           # Course JPA repository
    │   └── resources/
    │       ├── application.properties              # Base configuration (safe for Git)
    │       └── application-local.properties        # Local secrets override (gitignored)
    └── test/
        └── java/com/example/ucs067/studentmanagement/
            └── StudentManagementApplicationTests.java
```

---

## API Endpoints

Base URL: `http://localhost:8081`

> **Authentication:** `GET /hello` is public. All other endpoints require **HTTP Basic Auth** (username `admin` by default).

### General & Sample

| Method | Endpoint | Auth | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/hello` | Public | Returns `"Hello, Spring Boot!"` |
| `GET` | `/student` | Basic Auth | Returns a sample `Student` JSON object (`Rahul`) |

### Students

| Method | Endpoint | Auth | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/students` | Basic Auth | Get all students |
| `GET` | `/students/{id}` | Basic Auth | Get a student by ID (`200 OK` or `404 Not Found`) |
| `POST` | `/students` | Basic Auth | Create a new student (validates `name` & `email`) |
| `PUT` | `/students/{id}` | Basic Auth | Update a student's `name` and `email` (`200 OK` or `404 Not Found`) |
| `DELETE` | `/students/{id}` | Basic Auth | Delete a student by ID (`200 OK` or `404 Not Found`) |
| `GET` | `/students/search/name/{name}` | Basic Auth | Find students by exact name |
| `GET` | `/students/search/name/contains/{keyword}` | Basic Auth | Find students whose name contains `keyword` |
| `GET` | `/students/search/email/{email}` | Basic Auth | Find students by exact email |
| `GET` | `/students/page` | Basic Auth | Get paginated students (e.g., `?page=0&size=5&sort=name,asc`) |
| `GET` | `/students/sort/name` | Basic Auth | Get all students sorted by `name` |

### Courses

| Method | Endpoint | Auth | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/courses` | Basic Auth | Get all courses |
| `POST` | `/students/{studentId}/courses` | Basic Auth | Add a new course to an existing student |

---

## Annotations Used in This Project

### 1. Spring Boot & Configuration
- **`@SpringBootApplication`** (`StudentManagementApplication`): Entry-point annotation that enables auto-configuration, component scanning, and configuration support.
- **`@Configuration`** (`SecurityConfig`): Declares a configuration class containing Spring bean definitions.
- **`@Bean`** (`SecurityConfig.securityFilterChain`): Exposes the configured `SecurityFilterChain` as a Spring-managed bean.

### 2. Web & REST Controller (`HelloController`)
- **`@RestController`**: Marks the class as a REST controller (`@Controller` + `@ResponseBody`), serializing return values directly to JSON/text responses.
- **`@GetMapping`**: Maps HTTP `GET` requests to handler methods.
- **`@PostMapping`**: Maps HTTP `POST` requests to handler methods.
- **`@PutMapping`**: Maps HTTP `PUT` requests to handler methods.
- **`@DeleteMapping`**: Maps HTTP `DELETE` requests to handler methods.
- **`@PathVariable`**: Binds URI template variables (such as `{id}`, `{name}`, `{keyword}`, `{email}`, `{studentId}`) to method parameters.
- **`@RequestBody`**: Deserializes incoming JSON payloads into `Student` or `Course` Java objects.

### 3. JPA Entities & Database Mapping (`Student`, `Course`)
- **`@Entity`**: Marks `Student` and `Course` as JPA entities mapped to relational database tables.
- **`@Table(name = "students")`** (`Student`): Maps the `Student` entity to the `students` table in MySQL.
- **`@Id`**: Specifies the primary key field (`id`) of each entity.
- **`@GeneratedValue(strategy = GenerationType.IDENTITY)`**: Delegates primary key generation to MySQL's `AUTO_INCREMENT`.
- **`@OneToMany(mappedBy = "student", cascade = CascadeType.ALL, orphanRemoval = true)`** (`Student.courses`): Defines the one-to-many relationship from `Student` to `Course`, owned by `Course.student`, cascading all persistence operations and deleting orphaned courses.
- **`@ManyToOne`** (`Course.student`): Defines the many-to-one side linking each `Course` back to its parent `Student`.

### 4. JSON Serialization (Jackson)
- **`@JsonManagedReference`** (`Student.courses`): Marks the forward side of the `Student` -> `Course` relationship so `courses` are serialized in JSON.
- **`@JsonBackReference`** (`Course.student`): Marks the inverse reference so `Course` does not serialize `student` back again, preventing infinite recursion.

### 5. Validation & Exception Handling
- **`@Valid`** (`HelloController`): Triggers Jakarta Bean Validation on the incoming `Student` request body in `POST /students` and `PUT /students/{id}`.
- **`@NotBlank`** (`Student.name`, `Student.email`): Ensures the field is not `null` and contains at least one non-whitespace character.
- **`@Email`** (`Student.email`): Validates that the `email` field matches a valid email format.
- **`@RestControllerAdvice`** (`GlobalExceptionHandler`): Enables centralized exception handling across all REST controllers.
- **`@ExceptionHandler(MethodArgumentNotValidException.class)`** (`GlobalExceptionHandler`): Catches validation errors and returns field-level error messages with HTTP `400 Bad Request`.

### 6. Testing (`StudentManagementApplicationTests`)
- **`@SpringBootTest`**: Boots the full Spring application context for integration tests.
- **`@AutoConfigureMockMvc`**: Configures `MockMvc` to test controller endpoints without running an external HTTP server.
- **`@Autowired`**: Injects the `MockMvc` instance into the test class.
- **`@Test`**: Marks JUnit 5 test methods (`contextLoads`, `helloEndpointReturnsMessage`, `invalidStudentDataReturnsBadRequest`).

---

## Running the Project Locally

### Prerequisites
- **Java 21**
- **MySQL** running on `localhost:3306` with a database named `studentdb`:
  ```sql
  CREATE DATABASE studentdb;
  ```

### Configuration
`src/main/resources/application.properties` uses placeholder credentials and imports `application-local.properties` automatically (`spring.config.import=optional:classpath:application-local.properties`).

Create `src/main/resources/application-local.properties` (already ignored by `.gitignore`) with your local passwords:

```properties
spring.datasource.password=your_mysql_password
spring.security.user.password=your_admin_password
```

### Start the Application

On **Windows (PowerShell / CMD)**:
```powershell
.\mvnw spring-boot:run
```

On **macOS / Linux**:
```bash
./mvnw spring-boot:run
```

The server starts on **`http://localhost:8081`**.

---

## Example Requests (`curl`)

```bash
# Public endpoint
curl http://localhost:8081/hello

# Create a student (authenticated)
curl -u admin:your_admin_password -X POST http://localhost:8081/students \
  -H "Content-Type: application/json" \
  -d '{"name":"Rahul","email":"rahul@example.com"}'

# Get all students
curl -u admin:your_admin_password http://localhost:8081/students

# Add a course to student ID 1
curl -u admin:your_admin_password -X POST http://localhost:8081/students/1/courses \
  -H "Content-Type: application/json" \
  -d '{"courseName":"Data Structures","credits":4}'
```








## Lab wise Work done in this project
  This project was done in Enterprise java lab. Following will be the lab by lab work done for the project:

# Lab 1 & Lab 2:
starting two labs comprosises the theory part of the Enterprise Java and downloading the resources like java 21 and intellij Idea for the project.

# HelloController.java
HelloController.java is the REST API controller (@RestController) of project. It receives incoming HTTP requests, calls your database repositories (StudentRepository and CourseRepository), and sends back JSON or text responses.
-- Connects to Databases via Repositories
-- Handles 4 Group of API Repositories
1. Test/Sample Endpoints
2. Handles Student CRUD
3. Search,Pagination and Sorting
4. Course Management




## Lab 3
# 1
First created a jave file named **Student.java** which contains variables *Id* , *Name*,  *email*  and default, parameterized constructor, *setter and Getter function* for variables.

# 2 
we stored student in a temporary ArrayList instead of a DB.
In *HelloController.java* 
-- also Created *GET API* getStudents() = to get all students
-- created a *POST API* for adding students = to add a student in DB or list(for now)
-- Created a *GET API* getStudentById(int id) = to get a specific student by Id
-- Created a *Put API* = to update an existing student details
-- Created a *DELETE API* = TO DELETE A EXISTING STUDENT

# What We Learned ?
1. List<Student> stores multiple student objects.
2. ArrayList allows adding, updating, and removing students.
3. This data will remain only while the application is running.
4. @PutMapping handles HTTP PUT requests.
5. @PathVariable identifies the student to update.
6. @RequestBody receives the updated information.
7. Setter methods modify the existing object





## LAB 4
WORK DONE IN THIS LAB
1. DATABASE CONNECTION
2. JPA ENTITY(*'@Entity'*)
3. Spring Data JPA Repository

STEP BY STEP WORK 
1. CONNECTED TO MYSQL USING CMD(mysql -u root -p)
2. Created a database(studentdb)
3. Added 2 required dependencies (Spring Data JPA and MYSQL Connector/J)
4. Configure **application.properties**
5. Convert Student.java into a JPA Entity 
  -- using @Entity  = for defining students as an entity
  -- using @Table(name="students") = for defining students as a table 
  -- @Id =  Marking Id as the Primary key

6. Created a *Student Repository* in a new Repository Package
  -- Convert it into a Spring Data JPA Repository
  --**public interface StudentRepository extends JpaRepository<Student, Integer> {}**


## What Changed ?
CRUD Operations are now completely connected to MYSQL through *Student Repository* instead on an *ArrayList*.

# What is Spring Data JPA used for ? 
Spring Data JPA acts as the bridge between our Java application and the relational database. We don't write SQL for these basic CRUD operations; JpaRepository provides methods such as findAll(), findById(), save(), and deleteById(). 





## LAB 5 
 1. Derived Query Methods
 -- Instead of writing SQL we use Spring Data JPA methods and take them to our use to make methods like:
   -- **List<Student> findByName(String name);** = finds students by name
   -- **findByNameContaining(String keyword)** = finds students whose name contains the keyword
   -- **findByEmail(String email)** = Find student(s) with a particular email

# Spring Data JPA understands keywords such as:
  findBy
  And
  Or
  Containing
  StartingWith
  EndingWith
  GreaterThan
  LessThan
  Between
  OrderBy

2. Connecting the repository method to out *REST API*
  -- create an endpoint in *HelloController.java*
                    Browser / Client
                          ↓
                    GET /students/search/name/Rahul
                          ↓
                    HelloController
                          ↓
                    studentRepository.findByName("Rahul")
                          ↓
                    Spring Data JPA
                          ↓
                    MySQL
                          ↓
                    Student records

3. Adding pagination and sorting to a listing endpoint.
  -- Faster Database & Network Responses
  -- Lower Memory Usage
  -- Better Frontend User Experience
  -- Built-in Sorting Support

 # How?
 1. add imports in *HelloController.java*
 2. Create a *GET API*  
 **public Page<Student> getStudentsPage(Pageable pageable) {return studentRepository.findAll(pageable);}**
 3. Adding Sort import in *HelloController.java*
 4. Sort Endpoint
     **@GetMapping("/students/sort/name")public List<Student>getStudentsSortedByName() {return studentRepository.findAll(Sort.by("name"));}**





## LAB 6
This experiment implements and verifies a One-to-Many relationship between Student and Course in a Spring Boot application using JPA.

The key steps and work completed include:
1. Created the Course Entity: Added *Course.java* with JPA annotations (*@Entity, @Id*) and fields for *id*, *courseName*, and *credits*.
2. Configured the Relationship:
  --Updated *Student.java* with a *@OneToMany* mapping containing a list of courses.
  --Added a *@ManyToOne* mapping in *Course.java* to establish the owning side of the relationship and maintain the foreign key *(student_id)*.
3. Created *CourseRepository*: Implemented a *JpaRepository* interface for the *Course* entity to handle database operations.

4. Added REST Endpoints:
  --Added a GET endpoint *(/courses)* to fetch all courses.
  --Added a POST endpoint *(/students/{studentId}/courses)* to create a course and associate it with a specific student.

5. Handled JSON Serialization: Used Jackson's *@JsonManagedReference* and *@JsonBackReference* annotations to prevent infinite recursion when converting the bidirectional relationship to JSON.

6. Tested Functionality: Verified database persistence by assigning multiple courses to an existing student and retrieving the student's records to confirm the One-to-Many mapping works correctly.





## LAB 7
This experiment implements input validation and centralized exception handling for your Spring Boot API:

1. Added Validation Dependency: Included *spring-boot-starter-validation* in *pom.xml*.
2. Updated Model Validation: Added validation annotations (*@Positive* for ID, *@NotBlank* for name and email, and *@Email* for email format) to *Student.java*.
3. Enabled Endpoint Validation: Applied *@Valid* to the POST and PUT student endpoints in *HelloController.java*.
4. Created Centralized Exception Handling: Implemented the *GlobalExceptionHandler* class annotated with *@RestControllerAdvice* to catch *MethodArgumentNotValidException* and return HTTP 400 Bad Request responses with readable error messages.
5. Tested and Verified: Used cURL commands to test valid and invalid data for POST and PUT requests, confirming proper error responses and data integrity.

**Conclusion**: You have successfully implemented and tested:
1. Field validation using *@NotBlank*, *@Email*, and *@Positive*.
2. Request validation using *@Valid* in POST and PUT endpoints.
3. Centralized exception handling using *@RestControllerAdvice* and *@ExceptionHandler*.
4. HTTP 400 responses with readable validation messages.
5. Data integrity — invalid updates do not change existing records





## LAB 8
In this experiment, we successfully built, tested, and verified a Spring Boot REST API for a Student Management system

1. Database and Model Refinements: We configured our MySQL database by ensuring the student ID column auto-incremented properly while maintaining foreign key relationships with courses. We also updated our *Student.java* entity model (adjusting fields, constructors, and getters/setters).
2. Automated Testing with JUnit & MockMvc: We wrote and verified unit tests in *StudentManagementApplicationTests.java.* This included testing our basic */hello* endpoint and validating that submitting incomplete or incorrect student data properly returns a *400 Bad Request* status along with appropriate error messages.
3. API Testing via cURL: We used the Windows Command Prompt to test the complete CRUD lifecycle of our application:
  --Read: Retrieved all student records and queried specific students by ID (verifying *HTTP 200 OK* responses).
  --Create: Added new student records via POST requests and confirmed that new IDs were correctly generated.
  --Update: Modified existing student details using PUT requests and verified the changes.
  --Delete: Removed student records and verified via GET requests that they were successfully deleted.

**Conclusion**: REST API endpoints were successfully tested using cURL, CRUD operations were verified with the MySQL database, and automated tests were executed using JUnit and MockMvc.






## LAB 9
Basic authentication to the API with Spring Security 
In this experiment, we implemented and configured security controls for a Spring Boot application using Spring Security. 

# Step by Step Working :
1. **Dependency Integration**: Added the Spring Security module into the project's *pom.xml* file and updated Maven dependencies.
2. **Default Security Observation**: Restarted the application and accessed the */students* endpoint to observe how Spring Security intercepts unauthenticated traffic and requires credentials.
3. **Default Authentication Testing**: Retrieved the auto-generated temporary development password from the console and successfully authenticated using the default *user* account.
4. **Custom Credential Setup**: Configured fixed, application-level login credentials *(admin / admin123)* inside *application.properties* for consistent local testing.
5. **Explicit Security Configuration**: Created a *SecurityConfig.java* class to explicitly manage HTTP Basic Authentication, disable CSRF for local REST API testing, and establish endpoint rules (such as keeping the */hello* route public while protecting data endpoints).
6. **Endpoint Validation**: Verified the security setup by testing public and protected routes using valid, invalid, and missing credentials to ensure proper access control.

**Conclusion**: Experiment was successfully implemented and 
Now application Beahaves as follows:
1. */hello* — Public endpoint; no authentication required.
2. */students* — Protected endpoint; requires a username and password.
3. */courses* — Protected endpoint; requires authentication.
4. Correct credentials *(admin / admin123)* — Access granted.
5. Incorrect credentials — Access denied.






