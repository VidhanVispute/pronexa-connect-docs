
# PronexaConnect – Day 11 Notes: Default Values & Message Handling

---

## ✅ Day 11 – Default Values & Message Handling

### 🎯 Objectives
1. Set default values in entities when saving to DB (e.g., `provider`)
2. Implement user feedback messages after actions like registration
3. Handle session-based message display and automatic removal

---

## 📝 1. Setting Default Values in Entities

**Example – `User` entity**

```java
@Enumerated(EnumType.STRING)
private Providers provider = Providers.SELF;
````

**Explanation:**

* `@Enumerated(EnumType.STRING)` → Stores enum as readable string, not ordinal
* `provider` defaults to `SELF` → user registered directly through app
* Ensures DB clarity and avoids issues if enum order changes

**Important:**

* Using `User.builder()` may **not set default values**
* Instead, create object and set fields explicitly:

```java
User user = new User();
user.setName(userForm.getName());
user.setAbout(userForm.getAbout());
user.setEmail(userForm.getEmail());
user.setPassword(userForm.getPassword());
user.setPhoneNumber(userForm.getPhoneNumber());
user.setProfilePic("default-profile-pic-url");
```

---

## 🛠️ 2. Message Handling

### a) `Message` class

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Message {
    private String content;      // Text to display
    private MessageType type;    // Color type: blue, red, green, yellow
}
```

### b) `MessageType` enum

```java
public enum MessageType {
    blue, red, green, yellow
}
```

### c) SessionHelper – Remove message after display

```java
@Component
public class SessionHelper {

    public static void removeMessage() {
        try {
            HttpSession session = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
                .getRequest().getSession();
            session.removeAttribute("message");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### d) Thymeleaf fragment – `message.html`

```html
<!-- This container shows a session message if available -->
<div
  th:if="${session.message}"                 
  th:object="${session.message}"           
  th:fragment="messagebox"                 
  th:classappend="*{'text-'+type+'-800 border border-'+type+'-300 rounded-lg bg-'+type+'-50 dark:bg-gray-800 dark:text-'+type+'-400 dark:border-'+type+'-800'}"

  class="flex items-center p-4 mb-4 text-sm"
  role="alert"
><!-- Dynamically appends CSS classes based on the message 'type' (like red, blue, green), allowing styling variations -->
  <!-- Icon shown alongside the message -->
  <svg
    class="flex-shrink-0 inline w-4 h-4 me-3"
    aria-hidden="true"
    xmlns="http://www.w3.org/2000/svg"
    fill="currentColor"
    viewBox="0 0 20 20"
  >
    <!-- SVG path data draws an info or warning icon -->
    <path
      d="M10 .5a9.5 9.5 0 1 0 9.5 9.5A9.51 9.51 0 0 0 10 .5ZM9.5 4a1.5 1.5 0 1 1 0 3 1.5 1.5 0 0 1 0-3ZM12 15H8a1 1 0 0 1 0-2h1v-3H8a1 1 0 0 1 0-2h2a1 1 0 0 1 1 1v4h1a1 1 0 0 1 0 2Z"
    />
  </svg>

  <!-- Screen-reader-only label for accessibility -->
  <span class="sr-only">Info</span>

  <!-- The message content -->
  <div>
    <span data-th-text="*{content}"></span> <!-- Binds and displays the 'content' property of the message -->
  </div>

  <!-- This block executes the sessionHelper to remove the message after displaying -->
  <th:block th:text="${@sessionHelper.removeMessage()}"></th:block>
  <!-- Ensures the message is removed after being shown once, preventing it from appearing again -->
</div>

```

**Explanation:**

* Dynamically displays a message from session
* CSS classes depend on message type (`green` for success, `red` for error)
* Message is removed after first display

---

## ⚙️ 3. Controller Handling – Add Message After Registration

```java
@RequestMapping(value = "/do-register", method = RequestMethod.POST)
public String processRegister(@ModelAttribute UserForm userForm, HttpSession session) {

    // Create User object and set fields manually (for defaults)
    User user = new User();
    user.setName(userForm.getName());
    user.setAbout(userForm.getAbout());
    user.setEmail(userForm.getEmail());
    user.setPassword(userForm.getPassword());
    user.setPhoneNumber(userForm.getPhoneNumber());
    user.setProfilePic("default-profile-pic-url");

    // Save user to DB
    User savedUser = userService.saveUser(user);

    // Create success message
    Message message = new Message();
    message.setContent("Registration successful!");
    message.setType(MessageType.green);

    // Store message in session
    session.setAttribute("message", message);

    return "redirect:/signup"; // Redirect to show message
}
```

**Key Points:**

* `HttpSession` stores message temporarily
* Thymeleaf fragment reads `session.message`
* `SessionHelper` ensures message disappears after page refresh
* Provides clear feedback to the user

---

### 🔑 4. Summary – Day 11

| Task                                                   | Status |
| ------------------------------------------------------ | ------ |
| Set default entity values (`provider`, etc.)           | ✅ Done |
| Manual object creation to respect defaults             | ✅ Done |
| Implement `Message` & `MessageType`                    | ✅ Done |
| Create Thymeleaf fragment for messages                 | ✅ Done |
| Implement session-based message display & auto-removal | ✅ Done |
| Update controller to save user & show message          | ✅ Done |

---
