

# ✅ **Day 14 – Spring Security: Syncing Users with DB & SecurityFilterChain Setup**

---

## 🎯 **Objective**

* Integrate user authentication with database-backed users.
* Implement `UserDetails` in the custom `User` entity class.
* Configure user saving logic with password encoding and default roles.
* Create `AppConstant` for storing constants like roles.
* Implement a custom `UserDetailsService` to load users from the database.
* Configure `AuthenticationManager` for authentication requests.
* Setup `SecurityFilterChain` to control URL access, login, logout, and CSRF.
* Create `AuthFailureHandler` to handle authentication failures gracefully.

---

## 🟠 **1. Implementing `UserDetails` in `User` Entity**

✅ Code Snippet:

```java
public class User implements UserDetails {
    private String userId;
    private String name;
    private String email;
    private String password;
    private List<String> roleList;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roleList.stream()
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
    }

    @Override
    public String getPassword() { return password; }

    @Override
    public String getUsername() { return email; }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return true; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return true; }
}
```

### ✅ Key Concepts:

* Custom `User` entity implements `UserDetails`.
* Defines user information and authentication methods used by Spring Security.

---

## 🟠 **2. Saving Users (`UserServiceImpl`)**

✅ Code Snippet:

```java
@Override
public User saveUser(User user) {
    user.setUserId(UUID.randomUUID().toString());
    user.setPassword(passwordEncoder.encode(user.getPassword()));
    user.setRoleList(List.of(AppConstant.ROLE_USER));
    return userRepo.save(user);
}
```

---

## 🟠 **3. Defining Constants in `AppConstant`**

✅ Code Snippet:

```java
public class AppConstant {
    public static final String APP_NAME = "SCM";
    public static final String ROLE_USER = "ROLE_USER";
    public static final int CONTACT_IMAGE_WIDTH = 500;
    public static final int CONTACT_IMAGE_HEIGHT = 500;
    public static final String CONTACT_IMAGE_CROP = "fill";
    public static final int PAGE_SIZE = 10;
}
```

---

## 🟠 **4. Implementing `UserDetailsService`**

✅ Code Snippet:

```java
@Service
public class SecurityCustomUserDetailService implements UserDetailsService {

    @Autowired
    private UserRepo userRepo;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepo.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found with email : " + username));
    }
}
```

---

## 🟠 **5. Configuring `AuthenticationManager`**

✅ Code Snippet (`SecurityConfig.java`):

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

@Bean
public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
    return authenticationConfiguration.getAuthenticationManager();
}
```

---

## 📘 **6. Spring Security – SecurityFilterChain Configuration**

This defines how authentication and authorization are handled in the app.

### ✅ Code Snippet:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {

    httpSecurity.authorizeHttpRequests(authorize -> authorize
        .requestMatchers("/home", "/register", "/services").permitAll()
        .requestMatchers("/user/**").authenticated()
        .anyRequest().permitAll()
    );

    httpSecurity.formLogin(formLogin -> formLogin
        .loginPage("/login")
        .loginProcessingUrl("/authenticate")
        .successForwardUrl("/user/profile")
        .usernameParameter("email")
        .passwordParameter("password")
        .failureHandler(authFailtureHandler)
    );

    httpSecurity.csrf(AbstractHttpConfigurer::disable);

    httpSecurity.logout(logout -> logout
        .logoutUrl("/do-logout")
        .logoutSuccessUrl("/login?logout=true")
    );

    return httpSecurity.build();
}
```

### ✅ Key Concepts Explained:

| Feature                    | Explanation                                                       |
| -------------------------- | ----------------------------------------------------------------- |
| `.authorizeHttpRequests()` | Defines which URLs are public and which require authentication.   |
| `.formLogin()`             | Sets up custom login forms and behavior.                          |
| `.csrf().disable()`        | Disables CSRF protection for simplicity (only in specific cases). |
| `.logout()`                | Configures logout URLs and redirection.                           |
| `failureHandler`           | Custom handler to process login failures like disabled accounts.  |

---

## 📘 **7. Implementing `AuthFailureHandler`**

✅ Code Snippet:

```java
@Component
public class AuthFailtureHandler implements AuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException exception) throws IOException, ServletException {

        if (exception instanceof DisabledException) {
            HttpSession session = request.getSession();
            session.setAttribute("message",
                Message.builder()
                    .content("User is disabled, Email with verification link is sent on your email id !!")
                    .type(MessageType.red)
                    .build());
            response.sendRedirect("/login");
        } else {
            response.sendRedirect("/login?error=true");
        }
    }
}
```

### ✅ Key Concepts:

✔ Handles authentication failures such as disabled accounts.
✔ Provides user-friendly messages in case of failed login attempts.
✔ Redirects users based on error types.

---

## 📂 **8. Data Flow Overview**

1. User submits login form → request intercepted by `SecurityFilterChain`.
2. Authentication is processed using `AuthenticationManager`.
3. `SecurityCustomUserDetailService` loads user from database.
4. Password is checked with `BCryptPasswordEncoder`.
5. On success → user is redirected to `/user/profile`.
6. On failure → handled by `AuthFailureHandler`.
7. Logout functionality also configured.

---

## 📊 **9. Summary Table**

| Task                               | Status | Notes                                            |
| ---------------------------------- | ------ | ------------------------------------------------ |
| Implemented `UserDetails`          | ✅      | User entity customized for Spring Security       |
| Saved users with encoded passwords | ✅      | Used BCrypt and default roles                    |
| Created constants in `AppConstant` | ✅      | Centralized configuration                        |
| Implemented `UserDetailsService`   | ✅      | Loads users by email from DB                     |
| Configured `AuthenticationManager` | ✅      | Modern approach without deprecated classes       |
| Setup `SecurityFilterChain`        | ✅      | Defined routes, login, logout, and CSRF handling |
| Created `AuthFailureHandler`       | ✅      | Handles login failures and disabled accounts     |
| Explained data flow                | ✅      | Complete authentication lifecycle                |

---

## ✅ Final Outcome

* The application now supports **secure authentication backed by the database**.
* URLs are protected and users need to log in to access `/user/**`.
* Passwords are encrypted with **BCrypt**, ensuring security.
* Custom login forms and error handling are implemented through **SecurityFilterChain** and **AuthFailureHandler**.
* CSRF protection is handled appropriately depending on requirements.
* The solution aligns with **modern Spring Security practices**, avoiding deprecated methods.

---
✅ Purpose & Why We Need Them (Short & Simple)

| Component                             | Purpose                                                                | Why We Need It                                                                       |
| ------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **`User implements UserDetails`**     | Connects our user data to Spring Security.                             | So the framework knows how to check username, password, and roles from our database. |
| **`saveUser()` method**               | Saves users with encoded passwords and default roles.                  | To store passwords securely and control user permissions from the start.             |
| **`AppConstant` class**               | Stores constants like roles and settings.                              | To keep values in one place, avoid errors, and make updates easy.                    |
| **`SecurityCustomUserDetailService`** | Loads user details from the database during login.                     | So Spring can authenticate real users, not just hardcoded ones.                      |
| **`AuthenticationManager` bean**      | Manages how login requests are processed.                              | It links user data and password checking together safely.                            |
| **`SecurityFilterChain` bean**        | Controls which pages are public or secured and how login/logout works. | To protect the app and customize login behavior for users.                           |
| **`AuthFailureHandler`**              | Handles login errors and shows messages.                               | So users know why login failed and how to fix it.                                    |
| **Disabling CSRF (optional)**         | Turns off CSRF protection during development.                          | Makes testing easier, but should be enabled in real apps for safety.                 |
| **Role-based access control**         | Assigns roles like `ROLE_USER` to users.                               | To control who can access certain pages and features in the app.                     |

