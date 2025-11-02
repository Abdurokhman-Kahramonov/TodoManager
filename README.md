# ✅ Spring Boot To‑Do App — Project Documentation

A secure, database‑backed **Spring Boot** application that lets authenticated users **create, list, update, and delete** personal to‑dos.  
The stack includes **Spring MVC**, **Spring Security**, **Spring Data JPA**, **Hibernate**, **PostgreSQL**, **JSP views**, and **Bean Validation**. fileciteturn3file0

---

## ✨ Features
- User login with **in‑memory users** (BCrypt‑hashed passwords). fileciteturn3file0  
- To‑Do CRUD (create, read, update, delete) with **JPA/Hibernate** and **PostgreSQL**. fileciteturn3file0  
- **Server‑side validation** on the `Todo` entity (description length). fileciteturn3file0  
- **JSP** views with prefix/suffix config and a simple welcome page that shows the logged‑in username. fileciteturn3file0  
- Two controller implementations provided: **in‑memory service** and a production **JPA** variant. fileciteturn3file0

---

## 🧱 Architecture & Modules

### 1) Domain Model — `Todo` Entity
```java
@Entity @Table(name = "todoT")
public class Todo {
  @Id @GeneratedValue private int id;
  private String username;
  @Size(min=5, message="enter at least 10 characters")
  private String description;
  private LocalDate targetDate;
  private boolean done;
  // getters/setters/ctors/toString...
}
```
- Mapped to table **`todoT`** with `@Entity/@Table`.  
- Validation: `@Size(min=5, message="enter at least 10 characters")`. (Note the **message says 10** while min=5—see _Known quirks_.) fileciteturn3file0

---

### 2) Persistence — Spring Data JPA
```java
public interface TodoRepository extends JpaRepository<Todo,Integer> {
  List<Todo> findByUsername(String username);
}
```
- Standard CRUD via `JpaRepository`, plus a derived query to get todos for the logged‑in user. fileciteturn3file0

---

### 3) Web Layer — Controllers

#### a) JPA Controller (active)
```java
@Controller @SessionAttributes("name")
public class TodoControllerJPA {
  @RequestMapping("list-todos")
  String listAllTodos(ModelMap model) { ... }

  @RequestMapping(value="add-todo", method=GET)
  String showNewTodoPage(ModelMap model) { ... }

  @RequestMapping(value="add-todo", method=POST)
  String addNewTodoPage(ModelMap model, @Valid Todo todo, BindingResult result) { ... }

  @RequestMapping("delete-todo")
  String deleteTodo(@RequestParam int id) { ... }

  @RequestMapping(value="update-todo", method=GET)
  String showUpdatePage(@RequestParam int id, ModelMap model) { ... }

  @RequestMapping(value="update-todo", method=POST)
  String updateTodo(@Valid Todo todo, BindingResult result, ModelMap model) { ... }

  private String getLoggedInUsername(ModelMap model) { ... }
}
```
- Uses the authenticated principal from `SecurityContextHolder` to scope data by username.  
- Persists changes via `TodoRepository.save(...)`. fileciteturn3file0

#### b) In‑Memory Controller (example/legacy)
- Same endpoints as above, but backed by `TodoService` (static list). Currently commented out with `//@Controller`. Useful for demos or tests without DB. fileciteturn3file0

#### Welcome Controller
- Maps **`/`** → `welcomePage.jsp`, adds the logged‑in username to the model. fileciteturn3file0

---

### 4) Service Layer (demo) — `TodoService`
- Stores todos in a static list, provides `findByUserName`, `addTodo`, `deleteTodo`, `findById`, `updateTodo`.  
- Intended for **non‑JPA** mode; the JPA controller supersedes it in production. fileciteturn3file0

> **Note:** `findByUserName` uses `==` for string comparison and `addTodo` ignores the incoming date; both are noted under _Known quirks_. fileciteturn3file0

---

### 5) Security — `SpringSecurityConfiguration`
- **Users**: `Jack/password`, `Ferfero/ferfer` (BCrypt‑encoded at startup).  
- **AuthN/AuthZ**: all routes require authentication; default **form login** is enabled.  
- **CSRF** disabled and frame options disabled (useful during H2/dev, though here DB is Postgres). fileciteturn3file0

```java
@Bean InMemoryUserDetailsManager createUserDetailsManager() { ... }
@Bean SecurityFilterChain filterChain(HttpSecurity http) { ... }
@Bean PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
```
fileciteturn3file0

---

### 6) Configuration — `application.properties`
```properties
spring.mvc.view.prefix=WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
spring.mvc.format.date=yyyy-MM-dd

spring.jpa.defer-datasource-initialization=true
spring.jpa.show-sql=true

spring.datasource.url=jdbc:postgresql://localhost:5432/todo
spring.datasource.username=postgres
spring.datasource.password=Abdurahmon2004
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```
- JSP view resolver is configured; dates formatted as `yyyy-MM-dd`.  
- PostgreSQL connection + Hibernate dialect; `ddl-auto=update`.  
- SQL logged to console. fileciteturn3file0

> ⚠️ **Security Reminder:** The file contains a **plain‑text DB password**. Move secrets to environment variables or a secrets manager before production use. fileciteturn3file0

#### Sample Data (schema.sql / data.sql semantics)
```sql
insert into todo(id, username, description, target_date, done) values
(10003, 'Jack', 'Prepare presentation slides', CURRENT_DATE, false);
...
```
- Inserts several starter rows for user `Jack`. fileciteturn3file0

---

## 🧩 Endpoints (HTTP + Views)

| Method | Path          | Purpose                      | View         |
|-------:|---------------|------------------------------|--------------|
| GET    | `/`           | Welcome page                 | `welcomePage` |
| GET    | `/list-todos` | List current user’s todos    | `listTodos`  |
| GET    | `/add-todo`   | Show new todo form           | `todo`       |
| POST   | `/add-todo`   | Create todo (validate)       | redirect→list |
| GET    | `/update-todo?id={id}` | Show edit form      | `todo`       |
| POST   | `/update-todo`| Update todo (validate)       | redirect→list |
| GET    | `/delete-todo?id={id}` | Delete by id        | redirect→list |

All endpoints require authentication; unauthenticated users are redirected to the login form. fileciteturn3file0

---

## 🚀 Getting Started

### Prerequisites
- JDK 17+  
- Maven 3.9+  
- PostgreSQL running locally (`todo` database created; user/password match application.properties). fileciteturn3file0

### Run
```bash
mvn spring-boot:run
```
Then open: `http://localhost:8080/` → login with `Jack / password` (or `Ferfero / ferfer`). fileciteturn3file0

---

## 🧪 Validation & Forms
- The `Todo` form binds to the entity with `@Valid`; if `BindingResult` has errors, the **`todo.jsp`** form is re‑shown.  
- Description must satisfy `@Size(min=5, message="enter at least 10 characters")`. Adjust message/min to be consistent. fileciteturn3file0

---

## 🛠 Known Quirks / Fix‑Ups

1) **Validation message mismatch**  
   - `@Size(min=5, message="enter at least 10 characters")` → choose either `min=10` or fix message to 5. fileciteturn3file0

2) **String comparison in `TodoService.findByUserName`**  
   - Uses `==` instead of `.equals(...)`. Replace with:
   ```java
   todo -> username.equals(todo.getUsername())
   ```
   fileciteturn3file0

3) **`TodoService.addTodo` ignores provided date**  
   - Currently uses `LocalDate.now()` regardless of input. Use the passed `localDate`. fileciteturn3file0

4) **CSRF disabled**  
   - Re‑enable CSRF for production or protect state‑changing endpoints via tokens. fileciteturn3file0

5) **Plain‑text DB password**  
   - Externalize to env vars or Spring Cloud Config / Vault. fileciteturn3file0

---

## 📦 Packaging & Deployment

Build an executable jar:
```bash
mvn clean package
java -jar target/myFirstApp-*.jar
```
- Configure DB credentials via environment variables or `--spring.datasource.*` properties at runtime. fileciteturn3file0

---

## 🔐 Users & Roles
Users are declared in‐memory with roles `USER` and `ADMIN`. Passwords are **BCrypt** encoded on startup:  
```java
User.builder().passwordEncoder(enc).username("Jack").password("password").roles("USER","ADMIN").build();
```
Update or replace with JDBC/LDAP/Keycloak for real deployments. fileciteturn3file0

---

## 🗺 Data Model Notes
- `id` is generated; ensure JSP forms include hidden `id` field during updates.  
- `done` is a boolean; render as checkbox in JSP.  
- `targetDate` format is configured to `yyyy-MM-dd` to match HTML date inputs. fileciteturn3file0

---

## 📚 Useful Extensions
- Replace JSP with **Thymeleaf**.  
- Add pagination/sorting with `Pageable`.  
- Add REST API (JSON) alongside MVC.  
- Add integration tests with **Testcontainers** for Postgres.  
- Connect Spring Security to a database or OAuth 2.0 provider. fileciteturn3file0

---

