

# ✅ **Day 20 – Save Form Data into Database**

### 🎯 **Objective**

Today’s goal is to **store contact form data into the database**.
We created a proper **Repository → Service → Controller flow**, ensuring clean code, reusability, and separation of concerns.

---

## 🛠️ **Steps Completed**

### **1️⃣ Create ContactRepo**

```java
@Repository
public interface ContactRepo extends JpaRepository<Contact, String> {

    // Custom finder method (Spring Data JPA derives query automatically)
    List<Contact> findByUser(User user);

    // Custom JPQL query using @Query annotation
    @Query("SELECT c FROM Contact c WHERE c.user.id = :userId")
    List<Contact> findByUserId(@Param("userId") String userId);
}
```

**Explanation**

* `JpaRepository<Contact, String>` → gives CRUD methods out of the box.
* `findByUser(User user)` → auto-generated query (method naming convention).
* `@Query` → allows custom JPQL queries when method names are not enough.

👉 **Why we need this?**
This repo layer directly interacts with DB. Without it, we would have to write manual SQL every time.

---

### **2️⃣ Create ContactService (Interface)**

```java
public interface ContactService {
    Contact save(Contact contact);
    Contact update(Contact contact);
    List<Contact> getAll();
    Contact getById(String id);
    void delete(String id);
    List<Contact> getByUserId(String userId);
}
```

**Explanation**

* Defines **contract** for service methods.
* Ensures **loose coupling** (controller depends on interface, not implementation).
* Useful if later we add caching, validation, or external API calls.

👉 **Why we need this?**
Helps to separate **business logic from DB operations**.

---

### **3️⃣ Implement Service (ContactServiceImpl)**

```java
@Service
public class ContactServiceImpl implements ContactService {

    @Autowired
    private ContactRepo contactRepo;

    @Override
    public Contact save(Contact contact) {
        String contactId = UUID.randomUUID().toString();  // unique ID
        contact.setId(contactId);
        return contactRepo.save(contact);
    }

    @Override
    public Contact update(Contact contact) {
        var contactOld = contactRepo.findById(contact.getId())
                .orElseThrow(() -> new ResourceNotFoundException("Contact not found"));

        // Update fields
        contactOld.setName(contact.getName());
        contactOld.setEmail(contact.getEmail());
        contactOld.setPhoneNumber(contact.getPhoneNumber());
        contactOld.setAddress(contact.getAddress());
        contactOld.setDescription(contact.getDescription());
        contactOld.setPicture(contact.getPicture());
        contactOld.setFavorite(contact.isFavorite());
        contactOld.setWebsiteLink(contact.getWebsiteLink());
        contactOld.setLinkedInLink(contact.getLinkedInLink());
        contactOld.setCloudinaryImagePublicId(contact.getCloudinaryImagePublicId());

        return contactRepo.save(contactOld);
    }

    @Override
    public List<Contact> getAll() {
        return contactRepo.findAll();
    }

    @Override
    public Contact getById(String id) {
        return contactRepo.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contact not found with id " + id));
    }

    @Override
    public void delete(String id) {
        var contact = contactRepo.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contact not found with id " + id));
        contactRepo.delete(contact);
    }

    @Override
    public List<Contact> getByUserId(String userId) {
        return contactRepo.findByUserId(userId);
    }
}
```

**Explanation**

* `UUID.randomUUID().toString()` → generates unique ID for each contact.
* `ResourceNotFoundException` → ensures proper error handling.
* `@Service` → marks class as service bean, making it injectable.

👉 **Why we need this?**
Keeps business logic safe from DB complexity, reusable in multiple controllers.

---

### **4️⃣ Save Contact from Controller**

```java
@PostMapping("/add")
    public String saveContact(
            @Valid
            @ModelAttribute ContactForm contactForm,
            BindingResult result,
            Authentication authentication,
            RedirectAttributes redirectAttributes //  We useRedirectAttributes insted of Session to show message as flash
    ) {

        // 1️⃣ Validate form input
        if (result.hasErrors()) {
            // Log all validation errors
            result.getAllErrors().forEach(error -> logger.warn("Validation error: {}", error));

            // Add flash attribute
            redirectAttributes.addFlashAttribute("message",
                    Message.builder()
                            .content("Please correct the highlighted errors.")
                            .type(MessageType.red)
                            .build());

            // Redirect back to form
            return "user/add_contact";
        }

        // 2️⃣ Get the logged-in user
        String email = Helper.getLoggedInUserEmail(authentication);
        User user = userService.getUserByEmail(email).orElse(null);

        if (user == null) {
            redirectAttributes.addFlashAttribute("message",
                    Message.builder()
                            .content("User not found. Please login again.")
                            .type(MessageType.red)
                            .build());
            return "redirect:/login";
        }

        // 3️⃣ Convert ContactForm → Contact
        Contact contact = new Contact();
        contact.setName(contactForm.getName());
        contact.setFavorite(contactForm.isFavorite());
        contact.setEmail(contactForm.getEmail());
        contact.setPhoneNumber(contactForm.getPhoneNumber());
        contact.setAddress(contactForm.getAddress());
        contact.setDescription(contactForm.getDescription());
        contact.setLinkedInLink(contactForm.getLinkedInLink());
        contact.setWebsiteLink(contactForm.getWebsiteLink());
        contact.setUser(user);
        }

        //5️⃣ Save contact in DB
        contactService.save(contact);

        // 6️⃣ Set success message
        redirectAttributes.addFlashAttribute("message",
        Message.builder()
               .content("You have successfully added a new contact!")
               .type(MessageType.green)
               .build());


        logger.info("New contact added: {}", contact.getName());

        return "redirect:/user/contacts/add";
    }

```

**Explanation**

* `@ModelAttribute Contact contact` → automatically maps form fields to object.
* `contactService.save(contact)` → saves into DB via service layer.
* Redirection avoids form resubmission issues (Post/Redirect/Get pattern).

👉 **Why we need this?**
Ensures **form data flows cleanly** from UI → Controller → Service → Repository → DB.

---

## 🔄 **Data Flow Overview**

```
Form (Thymeleaf/HTML)
   ⬇
Controller (Receives data using @ModelAttribute)
   ⬇
Service (Business logic: set ID, validate, handle exceptions)
   ⬇
Repository (JPA saves into database)
   ⬇
Database (Contact stored successfully)
```

---

## 📋 **Summary Table**

| Step | Task                        | Status |
| ---- | --------------------------- | ------ |
| 1    | Create ContactRepo          | ✅ Done |
| 2    | Define ContactService       | ✅ Done |
| 3    | Implement ServiceImpl       | ✅ Done |
| 4    | Save contact via Controller | ✅ Done |

---

## 🏆 **Outcome**

* Successfully created **end-to-end flow** for saving contact data into DB.
* Implemented **clean architecture (Repo → Service → Controller)**.
* Learned **best practices: UUID for IDs, error handling, Post/Redirect/Get pattern**.

---

