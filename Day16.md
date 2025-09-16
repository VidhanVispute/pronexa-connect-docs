
# ✅ **Day 16 – Manage Logged-in User & Display Profile Data**

---

## 🎯 **Objective**

* Fetch logged-in user’s email or username from Spring Security.
* Show user-specific data in dashboard and profile pages.
* Handle both **local login** and **OAuth2 login** (Google, GitHub).
* Provide a reusable helper method to fetch logged-in user details.

---

## 🟠 **1. UserController – Fetching Logged-in User Data**

```java
@Controller
@RequestMapping("/user")
public class UserController {

    // Dashboard page
    @GetMapping("/dashboard")
    public String userDashboard(Authentication authentication, Model model) {
        String email = Helper.getLoggedInUserEmail(authentication);
        model.addAttribute("email", email);
        return "user/dashboard";
    }

    // Profile page
    @GetMapping("/profile")
    public String profile(Authentication authentication, Model model) {
        String email = Helper.getLoggedInUserEmail(authentication);
        model.addAttribute("email", email);
        return "user/profile";
    }
}
```

**Purpose:**

* `Authentication` object provides logged-in user details.
* Adds email/username to Thymeleaf model for display.

---

## 🟠 **2. Display Email in Thymeleaf Templates**

**profile.html**

```html
<body>
    <h1>User Profile</h1>
    <p th:text="'Logged in user email: ' + ${email}"></p>
    <a th:href="@{/user/dashboard}">Back to Dashboard</a>
</body>
```

**dashboard.html (example)**

```html
<body>
    <h1>User Dashboard</h1>
    <p th:text="'Logged in user email: ' + ${email}"></p>
</body>
```

**Purpose:**

* Show user-specific information dynamically.
* Provide navigation between dashboard and profile.

---

## 🟠 **3. Helper Class – Fetch Logged-in User Email**

```java
public class Helper {

    private static final Logger logger = LoggerFactory.getLogger(Helper.class);

    public static String getLoggedInUserEmail(Authentication authentication) {

        if (authentication instanceof OAuth2AuthenticationToken) {
            OAuth2AuthenticationToken oauthToken = (OAuth2AuthenticationToken) authentication;
            OAuth2User oauthUser = (OAuth2User) authentication.getPrincipal();
            String clientId = oauthToken.getAuthorizedClientRegistrationId();

            logger.info("OAuth2 login detected. Provider: {}", clientId);

            String email = null;

            if (clientId.equalsIgnoreCase("google")) {
                email = oauthUser.getAttribute("email");
                if (email == null) {
                    logger.error("Email not provided by Google OAuth2 provider.");
                } else {
                    logger.info("Fetched email from Google: {}", email);
                }
            } else if (clientId.equalsIgnoreCase("github")) {
                email = oauthUser.getAttribute("email");
                if (email == null) {
                    String login = oauthUser.getAttribute("login");
                    if (login != null) {
                        email = login + "@github.com"; // fallback
                        logger.warn("Email not provided by GitHub; using fallback: {}", email);
                    } else {
                        logger.error("Neither email nor login provided by GitHub OAuth2 provider.");
                    }
                } else {
                    logger.info("Fetched email from GitHub: {}", email);
                }
            } else {
                logger.warn("Unknown OAuth2 provider: {}", clientId);
            }

            return email;

        } else {
            String username = authentication.getName();
            logger.info("Local login detected. Username: {}", username);
            return username;
        }
    }
}
```

### ✅ **Purpose & Key Points**

| Feature                                        | Purpose                                                                       |
| ---------------------------------------------- | ----------------------------------------------------------------------------- |
| `Authentication` object                        | Provides the current logged-in user.                                          |
| OAuth2 detection (`OAuth2AuthenticationToken`) | Checks if login is via Google/GitHub.                                         |
| Fetch email / fallback                         | Ensures email is always available, even if OAuth provider does not supply it. |
| Logging                                        | Helps debug which provider and email are used.                                |
| Local login fallback                           | Supports users who log in with username/password.                             |

---

## 🟠 **4. Data Flow Overview**

1. User logs in (local or OAuth2).
2. Visits `/user/profile` or `/user/dashboard`.
3. Controller calls `Helper.getLoggedInUserEmail()`.
4. Helper determines login type:

   * Local: returns username.
   * Google: returns email.
   * GitHub: returns email or fallback.
5. Email is added to Thymeleaf model.
6. Template displays logged-in user information.

---

## 📊 **5. Summary Table**

| Task                               | Status | Notes                              |
| ---------------------------------- | ------ | ---------------------------------- |
| Fetch logged-in user email         | ✅      | Supports local + OAuth2 logins     |
| Display email in dashboard/profile | ✅      | Thymeleaf `th:text` binding        |
| Implement `Helper` class           | ✅      | Reusable method for any controller |
| Handle missing email in GitHub     | ✅      | Fallback using login + domain      |
| Add navigation                     | ✅      | Link from profile to dashboard     |

---

## ✅ **Outcome**

* Logged-in users (local or OAuth2) can see their email on profile and dashboard.
* Helper class abstracts provider-specific logic.
* Dashboard and profile pages are dynamically personalized.
* System ready for further user-specific data display (contacts, settings, etc.).

