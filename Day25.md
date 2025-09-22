

# **Day 25 – Contacts CRUD: View, Edit, Delete**

## **1️⃣ Delete Contact**

### **Thymeleaf Card (Delete Form)**

```html
<div class="flex items-center gap-2">
    <a th:href="@{'/user/contacts/view/' + ${contact.id}}" class="text-blue-400 hover:text-blue-900">
        <i class="fa-solid fa-eye"></i>
    </a>
    <a th:href="@{'/user/contacts/edit/' + ${contact.id}}" class="text-green-400 hover:text-green-900">
        <i class="fa-solid fa-user-pen"></i>
    </a>

    <!-- DELETE FORM -->
    <form th:action="@{/user/contacts/delete/{id}(id=${contact.id})}" method="post"
          th:onsubmit="return confirm('Are you sure you want to delete this contact?');">
        <input type="hidden" name="_method" value="delete"/>
        <button type="submit" class="text-red-400 hover:text-red-900">
            <i class="fa-solid fa-trash"></i>
        </button>
    </form>
</div>
```

✅ Notes:

* Confirmation popup via `onsubmit`.
* Uses `HiddenHttpMethodFilter` (`_method=delete`) to map POST → DELETE.
* Flash messages display success/error feedback.

---

### **Controller: Delete Endpoint**

```java
@PostMapping("/delete/{id}")
public String deleteContact(@PathVariable String id,
                            Authentication authentication,
                            RedirectAttributes redirectAttributes) {

    String email = Helper.getLoggedInUserEmail(authentication);
    User user = userService.getUserByEmail(email)
            .orElseThrow(() -> new RuntimeException("User not found"));

    try {
        Contact contact = contactService.getById(id)
                .orElseThrow(() -> new RuntimeException("Contact not found"));

        // Ownership check
        if (!contact.getUser().getUserId().equals(user.getUserId())) {
            redirectAttributes.addFlashAttribute("error", "You are not authorized to delete this contact.");
            return "redirect:/user/contacts";
        }

        contactService.delete(id);
        redirectAttributes.addFlashAttribute("success", "Contact deleted successfully.");
    } catch (Exception e) {
        redirectAttributes.addFlashAttribute("error", "Error deleting contact: " + e.getMessage());
    }

    return "redirect:/user/contacts";
}
```

✅ Key Points:

* Ownership check prevents unauthorized deletion.
* Uses `RedirectAttributes` for flash messages.

---

### **Service Layer**

**Interface: ContactService**

```java
Optional<Contact> getById(String id);
void delete(String id);
```

**Implementation: ContactServiceImpl**

```java
@Override
public Optional<Contact> getById(String id) {
    return contactRepo.findById(id);
}

@Override
public void delete(String id) {
    Contact contact = contactRepo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Contact not found with id " + id));
    contactRepo.delete(contact);
}
```

✅ Notes:

* `getById` returns Optional for safer null handling.
* `delete` throws `ResourceNotFoundException` if contact not found.

---

### **Flash Messages in Thymeleaf**

```html
<div th:if="${success}" class="bg-green-100 text-green-800 p-2 rounded mb-2">
    <span th:text="${success}"></span>
</div>
<div th:if="${error}" class="bg-red-100 text-red-800 p-2 rounded mb-2">
    <span th:text="${error}"></span>
</div>
```

---

## **2️⃣ View Contact**

### **Controller**

```java
@GetMapping("/view/{id}")
public String viewContact(@PathVariable String id,
                          Authentication authentication,
                          Model model,
                          RedirectAttributes redirectAttributes) {

    String email = Helper.getLoggedInUserEmail(authentication);
    User user = userService.getUserByEmail(email)
            .orElseThrow(() -> new RuntimeException("User not found"));

    Contact contact = contactService.getById(id)
            .orElseThrow(() -> new RuntimeException("Contact not found"));

    // Ownership check
    if (!contact.getUser().getUserId().equals(user.getUserId())) {
        redirectAttributes.addFlashAttribute("error", "You are not authorized to view this contact.");
        return "redirect:/user/contacts";
    }

    model.addAttribute("contact", contact);
    return "user/contact-view"; // Thymeleaf template
}
```

### **Thymeleaf: contact-view\.html**

```html
<h1 th:text="${contact.name}">Name</h1>
<p>Email: <span th:text="${contact.email}"></span></p>
<p>Phone: +91 <span th:text="${contact.phoneNumber}"></span></p>
<p>Address: <span th:text="${contact.address}"></span></p>
<p>Description: <span th:text="${contact.description}"></span></p>
<p>Website: <a th:href="${contact.websiteLink}" target="_blank" th:text="${contact.websiteLink}"></a></p>
<p>LinkedIn: <a th:href="${contact.linkedInLink}" target="_blank" th:text="${contact.linkedInLink}"></a></p>
<p>Favorite: <span th:text="${contact.favorite ? 'Yes' : 'No'}"></span></p>
<a th:href="@{/user/contacts}">Back to Contacts</a>
```

✅ Notes:

* Shows all contact fields.
* Ownership enforced.
* Clickable links, default image if none.

---

## **3️⃣ Edit Contact**

### **Controller**

```java
// GET: Load form
@GetMapping("/edit/{id}")
public String editContactForm(@PathVariable String id,
                              Authentication authentication,
                              Model model,
                              RedirectAttributes redirectAttributes) {

    String email = Helper.getLoggedInUserEmail(authentication);
    User user = userService.getUserByEmail(email)
            .orElseThrow(() -> new RuntimeException("User not found"));

    Contact contact = contactService.getById(id)
            .orElseThrow(() -> new RuntimeException("Contact not found"));

    if (!contact.getUser().getUserId().equals(user.getUserId())) {
        redirectAttributes.addFlashAttribute("error", "You are not authorized to edit this contact.");
        return "redirect:/user/contacts";
    }

    model.addAttribute("contact", contact);
    return "user/contact-edit";
}

// POST: Update contact
@PostMapping("/edit/{id}")
public String updateContact(@PathVariable String id,
                            @Valid @ModelAttribute("contact") Contact contact,
                            BindingResult result,
                            Authentication authentication,
                            RedirectAttributes redirectAttributes) {

    String email = Helper.getLoggedInUserEmail(authentication);
    User user = userService.getUserByEmail(email)
            .orElseThrow(() -> new RuntimeException("User not found"));

    Contact existingContact = contactService.getById(id)
            .orElseThrow(() -> new RuntimeException("Contact not found"));

    if (!existingContact.getUser().getUserId().equals(user.getUserId())) {
        redirectAttributes.addFlashAttribute("error", "You are not authorized to edit this contact.");
        return "redirect:/user/contacts";
    }

    if (result.hasErrors()) return "user/contact-edit";

    // Preserve immutable fields
    contact.setId(existingContact.getId());
    contact.setUser(existingContact.getUser());

    contactService.update(contact);
    redirectAttributes.addFlashAttribute("success", "Contact updated successfully.");
    return "redirect:/user/contacts/view/" + id;
}
```

### **Thymeleaf Form: contact-edit.html**

```html
<form th:action="@{/user/contacts/edit/{id}(id=${contact.id})}" th:object="${contact}" method="post">
    <input type="text" th:field="*{name}" required/>
    <input type="email" th:field="*{email}" required/>
    <input type="text" th:field="*{phoneNumber}"/>
    <input type="text" th:field="*{address}"/>
    <input type="text" th:field="*{description}"/>
    <input type="url" th:field="*{websiteLink}"/>
    <input type="url" th:field="*{linkedInLink}"/>
    <input type="checkbox" th:field="*{favorite}"/> Favorite
    <button type="submit">Update Contact</button>
</form>
<a th:href="@{/user/contacts/view/{id}(id=${contact.id})}">Back</a>
```

✅ Notes:

* Pre-populated form using `th:field`.
* Ownership and validation enforced.
* Redirects to **view page** after update.
* Keeps consistent styling with dashboard.

---

### **Day 25 Summary**

1. **Delete Contact**

   * Card form + confirmation.
   * Controller checks ownership.
   * Flash messages for success/error.
2. **View Contact**

   * View page shows all fields.
   * Only owner can view.
3. **Edit Contact**

   * GET → pre-filled form.
   * POST → validation + ownership check + update.
4. **Thymeleaf**

   * Reuse dashboard style.
   * Flash messages for feedback.
5. **Service Layer**

   * `getById`, `delete`, `update`.

---

