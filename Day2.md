🗃️ 1. Database Configuration
---

````markdown
# ✅ PronexaConnect – Day 2 Documentation

Welcome to **PronexaConnect**! This document covers **Day 2**, including database configuration, basic controller setup, Thymeleaf integration, and running the application.

### ✅ Changes in `pom.xml`

* Ensure that the **MySQL Connector/J** dependency is added for database connectivity.

### ✅ Added `application.properties` with Environment Variable Support

```properties
# ==============================
# APPLICATION CONFIGURATION
# ==============================
spring.application.name=${SPRING_APP_NAME:nexaconnect}
server.port=${SERVER_PORT:8081}
server.baseUrl=${SERVER_BASE_URL:http://localhost:8081}

# ==============================
# DATABASE CONFIGURATION
# ==============================
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:${MYSQL_PORT:3306}/${MYSQL_DB:nexaconnect}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:root1234}

# ==============================
# JPA/Hibernate settings
# ==============================
spring.jpa.hibernate.ddl-auto=${SPRING_JPA_HIBERNATE_DDL_AUTO:update}
spring.jpa.show-sql=${SPRING_JPA_SHOW_SQL:true}
````

> ✅ **Why use environment variables?**
> This allows for easy configuration across environments (development, test, production) without changing the code.

---

## 📁 2. Created Package: `pagecontroller`

* Defined a new controller class, e.g., `HomeController.java`
* Annotated with `@Controller` to indicate it's a Spring MVC controller

```java
@Controller
public class HomeController {

    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("name", "Vidhan");
        return "home"; // Renders home.html from templates
    }
}
```

> 💡 **Tip:** `Model` is used to pass data from the controller to the view (HTML page).

---

## 🌐 3. Thymeleaf Template – Sample Home Page

Created a sample page under `src/main/resources/templates/home.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Home - NEXAConnect</title>
</head>
<body>
    <h1>Welcome to NexaConnect</h1>
    <p th:text="${name}">Default Name</p>
</body>
</html>
```

> 🧠 **Note:** `th:text="${name}"` dynamically displays the value passed from the controller.

---

## 🚀 4. Running the Application

* Ran the application on `localhost:8081` (custom port as per configuration)
* Accessed via: [http://localhost:8081/home](http://localhost:8081/home)
* Verified that the controller rendered the Thymeleaf page with the dynamic name from the model.

---

## 📌 Summary of Day 2

| **Task**                                | **Status**   |
| --------------------------------------- | ------------ |
| Database Config with Env Variables      | ✅ Completed  |
| Created Page Controller (`@Controller`) | ✅ Completed  |
| Request Mapping for `/home`             | ✅ Completed  |
| Thymeleaf Sample Page (`home.html`)     | ✅ Working    |
| Passed Model Attribute (`name`)         | ✅ Working    |
| Application Running on Port 8081        | ✅ Successful |

---

