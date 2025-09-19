

## ✅ **Day 17 – Adding User Data to Response**

### 🎯 **Objective**

* Fetch the logged-in user’s details from the database.
* Make user data available globally (to all controllers + views) after login.
* Display user profile information dynamically using Thymeleaf.

---

### 🛠 **Steps Implemented**

#### 1. **Service Layer – Get User by Email**

In `UserService.java`:

```java
Optional<User> getUserByEmail(String email);
```

In `UserServiceImpl.java`:

```java
@Override
public Optional<User> getUserByEmail(String email) {
    return userRepo.findByEmail(email);
}
```

👉 **Purpose**: Encapsulates DB logic to fetch a user by their email (primary identifier after login).

---

#### 2. **Root Controller with @ControllerAdvice**

In `RootController.java`:

```java
@ControllerAdvice
public class RootController {

    @Autowired
    private UserService userService;

    private static final Logger logger = LoggerFactory.getLogger(RootController.class);

    @ModelAttribute
    public void addLoggedInUserInformation(Model model, Authentication authentication) {
        if (authentication == null) {
            return; // not logged in
        }

        String email = Helper.getLoggedInUserEmail(authentication);
        logger.info("Logged in user: {}", email);

        User user = userService.getUserByEmail(email).orElse(null);

        model.addAttribute("loggedInUser", user);
    }
}
```

👉 **Purpose**:

* Runs **before every controller method**.
* Automatically adds `loggedInUser` to the model if authenticated.
* Avoids repeating the same code in multiple controllers.

---

#### 3. **User Controller**

In `UserController.java`:

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @GetMapping("/dashboard")
    public String userDashboard() {
        return "user/dashboard"; 
    }

    @GetMapping("/profile")
    public String profile() {
        return "user/profile"; 
    }
}
```

👉 **Purpose**:

* Provides endpoints for user dashboard & profile.
* Doesn’t need to fetch user manually—`loggedInUser` is already injected.

---

#### 4. **Profile Page (Thymeleaf)**

In `profile.html`:

```html
<h1>User Profile</h1>

<p>Name: <span th:text="${loggedInUser != null ? loggedInUser.name : 'Guest'}"></span></p>
<p>Email: <span th:text="${loggedInUser != null ? loggedInUser.email : 'Not logged in'}"></span></p>
<p>Phone: <span th:text="${loggedInUser != null ? loggedInUser.phoneNumber : 'N/A'}"></span></p>
<p>About: <span th:text="${loggedInUser != null ? loggedInUser.about : 'N/A'}"></span></p>
<p>Provider: <span th:text="${loggedInUser != null ? loggedInUser.provider : 'SELF'}"></span></p>

<a th:href="@{/user/dashboard}">Back to Dashboard</a>
```

👉 **Purpose**: Displays user info dynamically (fetched from DB).

---

### 📊 **Summary Table**

| Component                          | Purpose                                     |
| ---------------------------------- | ------------------------------------------- |
| `UserService.getUserByEmail`       | Fetch user by email from DB                 |
| `RootController @ControllerAdvice` | Add logged-in user globally to all views    |
| `UserController`                   | Exposes `/dashboard` & `/profile` endpoints |
| `profile.html`                     | Displays user data with Thymeleaf           |

---

### ✅ **Outcome**

* After login, user details are automatically fetched and injected into the model.
* All views (dashboard, profile, etc.) can access `loggedInUser` without extra code.
* User profile page displays personalized information.

