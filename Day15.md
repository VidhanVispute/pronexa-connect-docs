

# ✅ **Day 15 – OAuth2 Login Integration with Spring Security**

---

## 🎯 **Objective**

* Enable users to log in using OAuth2 providers like Google and GitHub.
* Save new OAuth users in the database after successful login.
* Redirect users to their profile page.
* Integrate OAuth2 with the existing Spring Security setup.

---

## 🟠 **1. Add Maven Dependency**

Add the Spring Boot OAuth2 client starter in your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

**Purpose:**

* Provides support for OAuth2 login and client registration.
* Manages OAuth2 authentication flow automatically.

---

## 🟠 **2. Add Login Buttons in `login.html`**

```html
<div class="mt-2 flex justify-center space-x-4">
    <a data-th-href="@{'/oauth2/authorization/github'}"
       class="px-4 py-2 bg-gray-200 rounded-md hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600">
       GitHub
    </a>
    <a data-th-href="@{'/oauth2/authorization/google'}"
       class="px-4 py-2 bg-gray-200 rounded-md hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600">
       Google
    </a>
</div>
```

**Purpose:**

* Clicking these buttons starts the OAuth2 login flow.
* `data-th-href="@{'/oauth2/authorization/github'}"` is important because it triggers Spring Security’s OAuth2 authentication for the provider.

---

## 🟠 **3. Configure SecurityFilterChain for OAuth2**

Update `SecurityConfig.java` to enable OAuth2 login:

```java
httpSecurity.oauth2Login(oauth -> {
    oauth.loginPage("/login");        // Custom login page
    oauth.successHandler(oAuthSuccessHandler); // Custom handler after login
});
```

**Purpose:**

* Tells Spring Security to use OAuth2 login.
* Redirects to `/login` for authentication.
* Uses a custom success handler to save or update users in the database.

**Injection Example:**

```java
@Autowired
private OAuthAuthenticationSuccessHandler oAuthSuccessHandler;
```

---

## 🟠 **4. Implement `OAuthAuthenticationSuccessHandler`**

```java
@Component
public class OAuthAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Autowired
    private UserRepo userRepo;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
                                        Authentication authentication) throws IOException {
        OAuth2AuthenticationToken oauthToken = (OAuth2AuthenticationToken) authentication;
        DefaultOAuth2User oauthUser = (DefaultOAuth2User) oauthToken.getPrincipal();
        String provider = oauthToken.getAuthorizedClientRegistrationId().toLowerCase();

        User user = mapOAuthUserToUser(oauthUser, provider);

        // Save user if not exists
        User existingUser = userRepo.findByEmail(user.getEmail()).orElse(null);
        if (existingUser == null) {
            userRepo.save(user);
        }

        // Redirect to user profile
        new DefaultRedirectStrategy().sendRedirect(request, response, "/user/profile");
    }

    private User mapOAuthUserToUser(DefaultOAuth2User oauthUser, String provider) {
        User user = new User();
        user.setUserId(UUID.randomUUID().toString());
        user.setRoleList(List.of(AppConstant.ROLE_USER));
        user.setEnabled(true);
        user.setEmailVerified(true);
        user.setPassword("dummy"); // OAuth users usually don’t need a password

        switch (provider) {
            case "google" -> {
                user.setEmail(oauthUser.getAttribute("email"));
                user.setName(oauthUser.getAttribute("name"));
                user.setProfilePic(oauthUser.getAttribute("picture"));
                user.setProviderUserId(oauthUser.getName());
                user.setProvider(Providers.GOOGLE);
                user.setAbout("This account is created using Google.");
            }
            case "github" -> {
                user.setEmail(oauthUser.getAttribute("email"));
                user.setName(oauthUser.getAttribute("name"));
                user.setProfilePic(oauthUser.getAttribute("avatar_url"));
                user.setProviderUserId(oauthUser.getName());
                user.setProvider(Providers.GITHUB);
                user.setAbout("This account is created using GitHub.");
            }
            default -> {}
        }

        return user;
    }
}
```

**Purpose:**

* Maps OAuth2 attributes to your `User` entity.
* Saves new users in the database automatically.
* Handles redirection after successful login.

---

## 🟠 **5. Configure `application.properties` for OAuth2 Providers**

```properties
# Google OAuth
spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET
spring.security.oauth2.client.registration.google.scope=email,profile

# GitHub OAuth
spring.security.oauth2.client.registration.github.client-id=YOUR_GITHUB_CLIENT_ID
spring.security.oauth2.client.registration.github.client-secret=YOUR_GITHUB_CLIENT_SECRET
spring.security.oauth2.client.registration.github.scope=user:email

# Optional: custom login page
spring.security.oauth2.client.login-page=/login
```

**Purpose:**

* Registers OAuth2 providers with client ID/secret.
* Defines the scopes (data requested from the provider).
* Connects Spring Security to external OAuth services.

---

## 🟠 **6. Steps Flow**

1. User clicks Google/GitHub login button.
2. Redirected to OAuth2 provider for authentication.
3. After successful login, `OAuthAuthenticationSuccessHandler` is called.
4. User data is saved in the database if new.
5. User is redirected to `/user/profile`.

---

## 📊 **Summary Table**

| Task                                        | Status | Notes                                          |
| ------------------------------------------- | ------ | ---------------------------------------------- |
| Add OAuth2 dependency                       | ✅      | `spring-boot-starter-oauth2-client`            |
| Add login buttons                           | ✅      | Buttons for GitHub and Google                  |
| Configure SecurityFilterChain               | ✅      | Enable OAuth2 login and inject success handler |
| Implement OAuthAuthenticationSuccessHandler | ✅      | Maps attributes, saves new users, redirects    |
| Configure application.properties            | ✅      | Client IDs, secrets, scopes for providers      |
| Data flow verified                          | ✅      | OAuth login → DB save → redirect               |

---

## ✅ **Outcome**

* Users can log in using Google or GitHub.
* New users are automatically saved in the database.
* OAuth login is fully integrated with existing Spring Security setup.
* Redirection after login and role assignment are handled automatically.

