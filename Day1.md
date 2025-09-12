# ✅ PronexaConnect – Day 1 Documentation

Welcome to **PronexaConnect**! This document covers the **Day 1 setup**, providing detailed instructions on initializing the project, configuring dependencies, understanding the architecture, and editing notes for future reference.

---

## 🔧 Project Setup – Spring Boot with Java 21

For the initial setup, we used [Spring Initializr](https://start.spring.io) to bootstrap the project structure.

### 📂 Project Configuration

- **Project Type:** `Maven`
- **Language:** `Java`
- **Java Version:** `21`

> ✅ **Why Java 21?**  
> Java 21 is a **LTS (Long-Term Support)** version, offering improvements in performance, pattern matching, records, switch enhancements, and memory management. It’s ideal for enterprise-grade applications.

---

## 📦 Initial Dependencies

| **Dependency**                | **Purpose**                                                                 |
| ----------------------------- | --------------------------------------------------------------------------- |
| **Spring Web**                 | Build RESTful web services and web applications.                             |
| **Thymeleaf**                  | Server-side template engine for generating dynamic HTML pages.              |
| **Spring Data JPA**            | ORM tool to interact with databases using Java objects.                     |
| **Spring Validation**          | Validate user inputs at frontend and backend.                               |
| **Lombok**                     | Reduce boilerplate code with annotations like `@Getter`, `@Setter`, etc.    |
| **Spring Boot DevTools**      | Provides auto-restart, live reload, and enhanced development experience.   |
| **MySQL Driver**               | Connect the application to a MySQL database.                                |

---

## 🧩 Future Dependencies (To Be Added Later)

| **Dependency**            | **Purpose**                                                            |
| ------------------------- | ---------------------------------------------------------------------- |
| **Spring Security**       | Secure endpoints and manage authentication/authorization.               |
| **OAuth2 Client**         | Enable social logins like Google, GitHub, etc.                         |
| **Spring Boot Starter Mail** | Send emails such as OTP and notifications.                          |
| **ModelMapper**           | Map between DTOs and Entity classes easily.                             |

---

## 🌐 Basic Architecture Flow

The flow from client to server is as follows:

1. **Client (Browser):** Sends requests like accessing pages or submitting forms.
2. **Spring Boot (Controller):** Processes the request.
3. **Database (via JPA):** Stores or fetches data.
4. **Thymeleaf Template:** Renders dynamic HTML content.
5. **Client Response:** The processed page is sent back to the user.

```mermaid
sequenceDiagram
    Client ->> Spring Controller: HTTP Request (e.g., /register)
    Spring Controller ->> JPA Repo: Save or Fetch Data
    JPA Repo ->> Database: Executes SQL
    Database -->> JPA Repo: Returns Data
    JPA Repo -->> Controller: Returns Java Object
    Controller ->> Thymeleaf Template: Injects Data
    Thymeleaf Template -->> Client: Rendered HTML Response
