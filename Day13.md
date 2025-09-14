

# ✅ Day 13 – Appling Spring Secruity and do Practice
---

## 🎯 **Objective**

* Secure all user-related pages using **Spring Security**.
* Implement login functionality with predefined users.
* Learn how to configure users both via code (`InMemoryUserDetailsManager`) and via `application.properties`.
* Understand **why BCrypt is important for password encryption** and how Spring handles password security.
* Set the foundation for role-based access control and safe password management.

---

## 🟠 **1. Creating User Routes (`UserController.java`)**

We define URLs related to users such as dashboard and profile, which will later be protected by Spring Security.

### ✅ Code snippet:

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping(value = "/dashboard")
    public String userDashboard() {
        System.out.println("User dashboard");
        return "user/dashboard";
    }

    @RequestMapping(value = "/profile")
    public String userProfile(Model model) {
        System.out.println("User profile");
        return "user/profile";
    }

    // Future endpoints:
    // - Add contact
    // - View contacts
    // - Edit contact
    // - Delete contact
}
```

### ✅ Key Concepts:

* `@Controller`: Marks the class as a web controller.
* `@RequestMapping("/user")`: All routes under `/user/`.
* Returning view names connects these methods to corresponding HTML templates.
* These pages will later be accessible only after login.

---

## 🟠 **2. Creating Basic Views**

### ✅ `dashboard.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Dashboard</title>
</head>
<body>
    <h1>User Dashboard Page</h1>
</body>
</html>
```

### ✅ `profile.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Profile</title>
</head>
<body>
    <h1>User Profile Page</h1>
</body>
</html>
```

### ✅ Notes:

* These files should be placed in `src/main/resources/templates/user/`.
* Later, forms and data can be added dynamically using Thymeleaf.
* The pages are simple but ready to be secured by authentication.

---

## 🟠 **3. Adding Spring Security Dependency**

In your `pom.xml`, add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### ✅ What happens after this:

| Scenario                   | Result                                                 |
| -------------------------- | ------------------------------------------------------ |
| Access any `/user/**` page | Redirects to login page                                |
| Default credentials        | Printed in console as `user` / auto-generated password |
| Without proper login       | User cannot access pages                               |

---

## 🟠 **4. Implementing In-Memory Authentication with Encoded Passwords**

You create your own users instead of relying on default credentials.

### ✅ Code snippet (`SecurityConfig.java`):

```java
@Configuration
public class SecurityConfig {

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user1 = User.builder()
            .username("admin123")
            .password(passwordEncoder().encode("admin123"))
            .roles("ADMIN", "USER")
            .build();

        UserDetails user2 = User.builder()
            .username("user123")
            .password(passwordEncoder().encode("password"))
            .roles("USER")
            .build();

        return new InMemoryUserDetailsManager(user1, user2);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### ✅ Key Points:

| Component                    | Explanation                                                    |
| ---------------------------- | -------------------------------------------------------------- |
| `User.builder()`             | Fluent API to create users easily.                             |
| `username()`                 | Defines the login name.                                        |
| `password()`                 | Password is hashed before storing.                             |
| `roles()`                    | Assigns roles to users (e.g. `USER`, `ADMIN`).                 |
| `InMemoryUserDetailsManager` | Stores users in memory — good for learning or prototypes.      |
| `BCryptPasswordEncoder`      | Provides secure encryption with salting and adaptive strength. |

---

## 🟠 **5. Why Use BCrypt?**

### ✅ Important reasons:

✔ **One-way encryption** → Password cannot be reversed.
✔ **Salting** → Each password is uniquely hashed, preventing rainbow table attacks.
✔ **Adaptive strength** → As technology evolves, the hashing complexity can be increased.
✔ **Industry best practice** → Avoid storing plain text or weak hashes.

### ✅ Code explanation:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

This bean ensures that whenever a password is stored or checked, it is encoded properly.

---

## 🟠 **6. Alternative: Using `application.properties`**

For quick demos or testing, you can configure user credentials in your `application.properties` like this:

```properties
spring.security.user.name=user123
spring.security.user.password=password
spring.security.user.roles=USER
```

### ✅ Notes:

* Passwords are stored in plain text by default.
* Recent Spring Boot versions may require prefixing the password with `{noop}` to bypass encoding:

```properties
spring.security.user.password={noop}password
```

* **Not recommended for production** because passwords are easily accessible.

---

### ✅ Comparison Table

| Feature           | InMemoryUserDetailsManager (Java)           | `application.properties` Approach |
| ----------------- | ------------------------------------------- | --------------------------------- |
| Security          | Strong, uses BCrypt                         | Weak, may store plain text        |
| Control           | Define multiple users & roles               | Limited to basic setup            |
| Flexibility       | High                                        | Low                               |
| Recommended for   | Development/testing with security awareness | Quick demos only                  |
| Password handling | Encoded                                     | Plain text (unless specified)     |

---

## 📂 **7. Data Flow Overview**

1. User requests `/user/dashboard` → Spring Security intercepts.
2. Spring checks if user is authenticated.
3. If not, user is redirected to the login page.
4. Upon submitting credentials, Spring matches using `UserDetailsService`.
5. Password comparison is done with `BCryptPasswordEncoder`.
6. Authenticated users access the requested page.
7. Roles (e.g. `ADMIN`, `USER`) can be checked for advanced access control.

---

## 📊 **8. Summary Table**

| Task                                           | Status | Notes                              |
| ---------------------------------------------- | ------ | ---------------------------------- |
| Created user-related routes                    | ✅      | `/dashboard` and `/profile` set up |
| Created view templates                         | ✅      | Simple HTML files created          |
| Added Spring Security dependency               | ✅      | All routes now require login       |
| Implemented in-memory authentication           | ✅      | Defined users with roles           |
| Explained and applied BCrypt password encoding | ✅      | Ensures security against attacks   |
| Explained `application.properties` setup       | ✅      | Useful but less secure             |
| Provided data flow overview                    | ✅      | Step-by-step request handling      |

---

## ✅ **Final Outcome**

* All `/user/**` routes are **secured** with Spring Security.
* Users must **login** to access pages like dashboard and profile.
* Passwords are **hashed securely with BCrypt**, making them resistant to brute-force attacks.
* Roles like `USER` and `ADMIN` are set up for future enhancements.
* Credentials can be configured via code or `application.properties`, with security implications explained.
* Your project now has a solid **authentication layer**, ready for further features like authorization and persistent storage.

