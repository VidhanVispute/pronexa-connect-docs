
## ✅ **Day 12 – User Form Validation**

### 🎯 **Objective**

* Implement **server-side validation** for user registration form.
* Ensure **user-friendly error messages** for invalid inputs.
* Integrate **Spring Boot + Thymeleaf** validation seamlessly.
* Skip client-side validation to handle all checks on the server.

---

### 📝 **1. User Form Setup with Annotations**

**File:** `UserForm.java`

We use **Jakarta Bean Validation annotations** to enforce rules directly in the form class.

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@ToString
public class UserForm {

    @NotBlank(message = "Username is required")
    @Size(min = 3, message = "Min 3 Characters is required")
    private String name;

    @Email(message = "Invalid Email Address")
    @NotBlank(message = "Email is required")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 6, message = "Min 6 Characters is required")
    private String password;

    @NotBlank(message = "About is required")
    private String about;

    @Size(min = 8, max = 12, message = "Invalid Phone Number")
    private String phoneNumber;
}
```

**✅ Key Points:**

| Annotation        | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| `@NotBlank`       | Ensures field is not empty (ignores whitespace). |
| `@Size(min, max)` | Sets min and max character limits.               |
| `@Email`          | Validates proper email format.                   |
| `@Getter/@Setter` | Lombok simplifies getter/setter boilerplate.     |
| `@Builder`        | Allows building objects fluently for testing.    |

---

### 📝 **2. Controller Configuration**

**File:** `UserController.java`

* Use `@Valid` to trigger validation.
* Use `BindingResult` to capture errors.
* Store feedback in `HttpSession` if needed.

```java
@RequestMapping(value = "/do-register", method = RequestMethod.POST)
public String processRegister(
        @Valid @ModelAttribute UserForm userForm, 
        BindingResult bindingResult, 
        HttpSession session
) {
    // Check for validation errors
    if (bindingResult.hasErrors()) {
        return "signup"; // Return to form if errors exist
    }

    // Proceed to save userForm data
    // e.g., userService.save(userForm);
    return "success"; // Redirect to success page
}
```

**✅ Key Points:**

| Component         | Explanation                                                      |
| ----------------- | ---------------------------------------------------------------- |
| `@Valid`          | Triggers server-side validation based on annotations.            |
| `BindingResult`   | Holds validation errors; must come **immediately after @Valid**. |
| `@ModelAttribute` | Binds form data to the UserForm object automatically.            |
| `HttpSession`     | Optional: can be used to store messages for later display.       |

---

### 📝 **3. Thymeleaf Integration for Error Display**

**File:** `signup.html`

* Display field-specific validation messages.
* Disable client-side validation with `novalidate`.

```html
<form th:action="@{/do-register}" th:object="${userForm}" method="post" novalidate>
    <input type="text" th:field="*{name}" placeholder="Username"/>
    <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}" class="text-red-600 px-1 py-2"></p>

    <input type="email" th:field="*{email}" placeholder="Email"/>
    <p th:if="${#fields.hasErrors('email')}" th:errors="*{email}" class="text-red-600 px-1 py-2"></p>

    <input type="password" th:field="*{password}" placeholder="Password"/>
    <p th:if="${#fields.hasErrors('password')}" th:errors="*{password}" class="text-red-600 px-1 py-2"></p>

    <textarea th:field="*{about}" placeholder="About"></textarea>
    <p th:if="${#fields.hasErrors('about')}" th:errors="*{about}" class="text-red-600 px-1 py-2"></p>

    <input type="text" th:field="*{phoneNumber}" placeholder="Phone Number"/>
    <p th:if="${#fields.hasErrors('phoneNumber')}" th:errors="*{phoneNumber}" class="text-red-600 px-1 py-2"></p>

    <button type="submit">Register</button>
</form>
```

**✅ Key Points:**

| Feature      | Explanation                                                            |
| ------------ | ---------------------------------------------------------------------- |
| `th:field`   | Binds input to form object property.                                   |
| `th:errors`  | Displays validation error for that field.                              |
| `novalidate` | Disables browser default validation to rely on server-side validation. |

---

### 📝 **4. Data Flow Overview**

1. User fills registration form and submits → `signup.html`.
2. Spring binds form data to `UserForm` via `@ModelAttribute`.
3. `@Valid` triggers annotation-based validation.
4. `BindingResult` checks for errors:

   * **If errors exist:** Return to signup form with messages.
   * **If no errors:** Save user data, redirect to success page.
5. Thymeleaf renders error messages next to respective fields.

---

### 📌 **5. Best Practices**

* Keep validation annotations **close to data** (DTO/Form classes).
* Always **check `BindingResult.hasErrors()`** before saving data.
* Use **custom messages** for better UX.
* Disable client-side validation only if you want **full control** on server-side checks.
* Consider **custom validators** for complex rules (e.g., password strength).

---

### 📊 **Task Summary**

| Task                                          | Status      | Notes                                                 |
| --------------------------------------------- | ----------- | ----------------------------------------------------- |
| Create `UserForm` with validation annotations | ✅ Completed | Fields validated using `@NotBlank`, `@Size`, `@Email` |
| Configure controller with `@Valid`            | ✅ Completed | Used `BindingResult` to capture errors                |
| Setup Thymeleaf error messages                | ✅ Completed | `th:errors` used next to input fields                 |
| Disable client-side validation                | ✅ Completed | Used `novalidate` in form                             |

---

### 🎯 **Outcome**

* User registration form is **fully validated on the server**.
* Users receive **field-specific, friendly error messages**.
* Data flow from frontend → backend → DB is **validated and safe**.
* Ready for further enhancements like **custom validators or password encryption**.


