

# **Day 23 – List Contacts with Pagination**

---

## **Objective**

* Fetch logged-in user’s contacts.
* Display them in pages (pagination).
* Sort contacts alphabetically by name.
* Use **Spring Data JPA pagination** (`Pageable`) and **Thymeleaf** for UI.

---

## **Step 1 – Update Contact Repository**

Add a method to fetch contacts by `userId` with pagination:

```java
package com.pronexa.connect.repositories;

import java.util.List;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import com.pronexa.connect.entities.Contact;
import com.pronexa.connect.entities.User;

@Repository
public interface ContactRepo extends JpaRepository<Contact, String> {

    // Find all contacts for a user (non-paginated)
    List<Contact> findByUser(User user);

    // Find contacts by userId with pagination
    @Query("SELECT c FROM Contact c WHERE c.user.id = :userId")
    Page<Contact> findByUserId(@Param("userId") String userId, Pageable pageable);
}
```

---

## **Step 2 – Update Contact Service Interface**

Add a **pagination-aware method**:

```java
package com.pronexa.connect.services;

import java.util.List;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import com.pronexa.connect.entities.Contact;

public interface ContactService {

    // Save a contact
    Contact save(Contact contact);

    // Update a contact
    Contact update(Contact contact);

    // Get all contacts (non-paginated)
    List<Contact> getAll();

    // Get contacts by userId with pagination
    Page<Contact> getByUserId(String userId, Pageable pageable);

    // Delete a contact
    void delete(String id);
}
```

---

## **Step 3 – Update Contact Service Implementation**

```java
package com.pronexa.connect.services.impl;

import java.util.List;
import java.util.UUID;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import com.pronexa.connect.entities.Contact;
import com.pronexa.connect.helpers.ResourceNotFoundException;
import com.pronexa.connect.repositories.ContactRepo;
import com.pronexa.connect.services.ContactService;

@Service
public class ContactServiceImpl implements ContactService {

    @Autowired
    private ContactRepo contactRepo;

    @Override
    public Contact save(Contact contact) {
        contact.setId(UUID.randomUUID().toString());
        return contactRepo.save(contact);
    }

    @Override
    public Contact update(Contact contact) {
        var contactOld = contactRepo.findById(contact.getId())
                .orElseThrow(() -> new ResourceNotFoundException("Contact not found"));

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
    public Page<Contact> getByUserId(String userId, Pageable pageable) {
        return contactRepo.findByUserId(userId, pageable);
    }

    @Override
    public void delete(String id) {
        var contact = contactRepo.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Contact not found with given id " + id));
        contactRepo.delete(contact);
    }
}
```

---

## **Step 4 – Update Controller**

Handle request parameters for page and size, fetch contacts for logged-in user:

```java
@GetMapping
public String listContacts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "5") int size,
        Model model,
        Authentication authentication
) {
    // 1️⃣ Get logged-in user
    String email = Helper.getLoggedInUserEmail(authentication);
    User user = userService.getUserByEmail(email).orElse(null);
    if (user == null) return "redirect:/login";

    // 2️⃣ Setup pageable object (sort by name ascending)
    Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());

    // 3️⃣ Fetch contacts page
    Page<Contact> contactsPage = contactService.getByUserId(user.getId(), pageable);

    // 4️⃣ Add data to model
    model.addAttribute("contacts", contactsPage.getContent());
    model.addAttribute("currentPage", page);
    model.addAttribute("totalPages", contactsPage.getTotalPages());

    return "user/contacts";
}
```

---

## **Step 5 – Thymeleaf Pagination Example**

```html
<!DOCTYPE html>
<html lang="en" th:replace="~{base :: parent(~{::#content}, ~{::title}, ~{})}">

<head>
    <title>Dashboard - Contacts</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.1.1/css/all.min.css">
</head>

<body>
    <div id="content" class="flex flex-col p-6 bg-gray-50 sm:ml-64">

        <!-- Sidebar -->
        <div th:replace="~{User/sidebar :: sidebar}"></div>

        <!-- Contacts Table -->
        <div class="overflow-x-auto bg-white rounded-lg shadow p-4 w-full">

            <table class="w-full text-sm text-left text-gray-500">
                <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                    <tr>
                        <th scope="col" class="px-6 py-3">Name</th>
                        <th scope="col" class="px-6 py-3">Phone</th>
                        <th scope="col" class="px-6 py-3">Links & Favorites</th>
                        <th scope="col" class="px-6 py-3">Action</th>
                    </tr>
                </thead>
                <tbody>
                    <tr th:each="contact : ${contacts}" class="bg-white border-b hover:bg-gray-50">

                        <!-- Name + Email + Image -->
                        <td class="px-6 py-4 flex items-center gap-3">
                            <!-- Contact Image -->
                            <img th:src="${contact.picture != null && contact.picture != '' ? contact.picture : '/css/default-user.jpeg'}"
                                alt="Contact Image" class="w-12 h-12 rounded-full object-cover" />

                            <!-- Name + Email -->
                            <div class="flex flex-col overflow-hidden">
                                <span th:text="${contact.name}" class="font-medium text-gray-800"></span>
                                <span th:text="${contact.email}" class="text-gray-500 text-sm"></span>
                            </div>
                        </td>

                        <!-- Phone -->
                        <td class="px-6 py-4">
                            <i class="fa-solid fa-phone w-4 h-4"></i>
                            <span data-th-text="${contact.phoneNumber}"></span>
                        </td>

                        <!-- Links (Favorite + Icons) -->
                        <td class="p-2 py-4">
                            <div class="flex items-center gap-3">
                                <!-- Favorite Star -->
                                <span th:if="${contact.favorite}" class="text-yellow-500 text-lg">★</span>
                                <span th:unless="${contact.favorite}" class="text-gray-400 text-lg">☆</span>

                                <!-- LinkedIn Icon -->
                                <i class="fa-brands fa-linkedin text-blue-600 text-lg"></i>

                                <!-- Personal Link Icon -->
                                <i class="fa-solid fa-link text-gray-700 text-lg"></i>

                            </div>

                        </td>

                        <!-- Actions (Edit/Delete/View) -->
                        <td class="p-2 py-4 ">

                            <div class="flex items-center gap-3">

                                <!-- Edit -->
                                <a th:href="@{'/user/contacts/edit/' + ${contact.id}}"
                                    class="flex items-center gap-1 text-blue-500 hover:underline whitespace-nowrap">
                                    <i class="fa-solid fa-file-pen"></i>
                                </a>

                                <!-- Delete -->
                                <a th:href="@{'/user/contacts/delete/' + ${contact.id}}"
                                    class="flex items-center gap-1 text-red-500 hover:underline whitespace-nowrap"
                                    onclick="return confirm('Are you sure?')">
                                    <i class="fa-solid fa-trash"></i>
                                </a>

                                <!-- View -->
                                <a th:href="@{'/user/contacts/view/' + ${contact.id}}"
                                    class="flex items-center gap-1 text-gray-700 hover:underline whitespace-nowrap">
                                    <i class="fa-solid fa-eye"></i>
                                </a>
                            </div>

                        </td>



                    </tr>

                </tbody>
            </table>

            <!-- Pagination -->
            <div class="mt-4 flex justify-center space-x-2">
                <button th:each="i : ${#numbers.sequence(0, totalPages-1)}" th:text="${i + 1}"
                    th:classappend="${i == currentPage} ? 'font-bold text-blue-600' : 'text-gray-600'"
                    th:onclick="'window.location.href=\'?page=' + i + '\''"
                    class="px-3 py-1 border rounded hover:bg-gray-100">
                </button>
            </div>
        </div>

    </div>
</body>

</html>
```

---

## ✅ **Step 7 – Summary**

1. Repository: Added `findByUserId(userId, Pageable)` for paginated query.
2. Service: Added `getByUserId(String userId, Pageable pageable)` in interface & impl.
3. Controller: Handles `page` and `size` parameters, fetches `Page<Contact>`, sends to model.
4. Thymeleaf: Added pagination UI (Previous, Next, page numbers).
5. Contacts are **sorted alphabetically** using `Sort.by("name")`.


