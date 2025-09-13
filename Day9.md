
# PronexaConnect – Day 9 Notes: Form Data Handling & Registration

---

## ✅ Day 9 – Handling Form Data in Signup using Thymeleaf & Spring Boot

### 🎯 Objective
- Bind form inputs from `signup.html` to a Java object (`UserForm`)
- Access user-submitted data in the backend controller
- Prepare for registration flow (storing user data later)

---

### 📝 1. Signup Form in Thymeleaf

Key points:
- `th:action` – specifies the URL where form data will be submitted
- `th:object` – binds the form to a Java object (`UserForm`)
- `th:field` – binds each input field to a property of the form object

**Example form field for Full Name:**

```html
<form th:action="@{/do-register}" method="post" th:object="${userForm}">
  <div>
    <label for="fullName" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">
      Full Name
    </label>
    <input type="text" id="fullName"
           th:field="*{name}"   <!-- Binds to userForm.name -->
           name="fullName"
           placeholder="John Doe"
           class="w-full p-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
           required />
  </div>

  <!-- Repeat similar fields for email, password, phoneNumber, about -->

  <button type="submit" class="btn btn-primary mt-4">Signup</button>
</form>
````

> ✅ **Why use `th:field`?**
> Automatically maps the input value to the corresponding property in `UserForm`. Avoids manual request parameter parsing.

---

### 🧩 2. Form Object Class – `UserForm`

```java
package com.pronexa.connect.forms;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ToString
public class UserForm {
    private String name;
    private String email;
    private String password;
    private String phoneNumber;
    private String about;
}
```

> **Lombok Annotations**
> \| Annotation        | Purpose |
> \|------------------|---------|
> \| `@Getter` / `@Setter` | Auto-generates getters & setters |
> \| `@NoArgsConstructor`   | Empty constructor needed by Spring |
> \| `@AllArgsConstructor`  | Constructor with all fields |
> \| `@Builder`             | Builder pattern to create objects easily |
> \| `@ToString`            | For debugging/logging purposes |

---

### 🛠️ 3. Controller Handlers

#### a) Show empty signup form

```java
@GetMapping("/signup")
public String signup(Model model) {
    UserForm userForm = new UserForm();      // Empty object for form binding
    model.addAttribute("userForm", userForm);
    return "signup";                          // Thymeleaf template
}
```

#### b) Process form submission

```java
@RequestMapping(value = "/do-register", method = RequestMethod.POST)
public String processRegister(@ModelAttribute UserForm userForm) {
    // userForm now holds all form data from signup.html
    System.out.println(userForm); // For testing

    // TODO: Save userForm data to User entity and database

    return "redirect:/login"; // After registration, redirect to login page
}
```

> ✅ **Key Concept – `@ModelAttribute`**
> Binds the form data automatically to the `UserForm` object. No need to manually extract request parameters.

---

### 💡 4. Data Flow Overview

1. User opens `/signup` → controller provides **empty `UserForm` object** → Thymeleaf binds form to it.
2. User fills form → submits → POST `/do-register`
3. `@ModelAttribute UserForm userForm` receives data automatically.
4. Backend can now use `userForm` to create a `User` entity and save it to DB.

---

### ✅ Summary Table – Day 9

| Task                                                      | Status |
| --------------------------------------------------------- | ------ |
| Created `UserForm` object for form data                   | ✅ Done |
| Signup page (`signup.html`) with `th:object` & `th:field` | ✅ Done |
| Controller GET mapping to load form                       | ✅ Done |
| Controller POST mapping to process form (`do-register`)   | ✅ Done |
| Verified that form data binds automatically to `UserForm` | ✅ Done |

---

Perfect! This is a very clear and thorough **Day 9 summary** for your PronexaConnect project. A few additional **tips and clarifications** to make it even more practical for your registration flow:

---

### 🔹 Extra Notes / Best Practices

1. **Validation**

   * You can add Spring’s validation annotations in `UserForm`:

     ```java
     import jakarta.validation.constraints.*;

     public class UserForm {
         @NotBlank(message="Name is required")
         private String name;

         @Email(message="Enter a valid email")
         @NotBlank(message="Email is required")
         private String email;

         @Size(min=6, message="Password must be at least 6 characters")
         private String password;

         private String phoneNumber;
         private String about;
     }
     ```
   * Then in the controller, use:

     ```java
     @PostMapping("/do-register")
     public String processRegister(@Valid @ModelAttribute UserForm userForm,
                                   BindingResult result) {
         if (result.hasErrors()) {
             return "signup"; // Re-render form with errors
         }
         // Save userForm to DB
         return "redirect:/login";
     }
     ```

2. **Error Display in Thymeleaf**

   ```html
   <div th:if="${#fields.hasErrors('name')}" th:errors="*{name}" class="text-red-500"></div>
   ```

