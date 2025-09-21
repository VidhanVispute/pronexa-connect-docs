

# **Day 24 – Contacts Search & Favorites with Pagination**

## **Objectives**

* Search contacts by name/email (case-insensitive)
* Filter only favorites
* Paginate results
* Keep filters persistent across pages
* Responsive UI with cards (Thymeleaf + Tailwind)

---

## **Backend**

### **1️⃣ ContactRepo**

```java
package com.pronexa.connect.repositories;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import com.pronexa.connect.entities.Contact;

@Repository
public interface ContactRepo extends JpaRepository<Contact, String> {

    @Query("SELECT c FROM Contact c WHERE c.user.id = :userId")
    Page<Contact> findByUserId(@Param("userId") String userId, Pageable pageable);

    @Query("SELECT c FROM Contact c WHERE c.user.id = :userId " +
           "AND (LOWER(c.name) LIKE LOWER(CONCAT('%', :keyword, '%')) " +
           "OR LOWER(c.email) LIKE LOWER(CONCAT('%', :keyword, '%'))) ")
    Page<Contact> searchByUserIdAndKeyword(@Param("userId") String userId,
                                           @Param("keyword") String keyword,
                                           Pageable pageable);

    @Query("SELECT c FROM Contact c WHERE c.user.id = :userId AND c.favorite = true")
    Page<Contact> findFavoritesByUserId(@Param("userId") String userId, Pageable pageable);
}
```

---

### **2️⃣ ContactService**

```java
package com.pronexa.connect.services;

import java.util.List;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import com.pronexa.connect.entities.Contact;

public interface ContactService {
    Contact save(Contact contact);
    Contact update(Contact contact);
    List<Contact> getAll();
    Page<Contact> getByUserId(String userId, Pageable pageable);
    void delete(String id);

    // Day 24 additions
    Page<Contact> search(String userId, String keyword, Pageable pageable);
    Page<Contact> getFavorites(String userId, Pageable pageable);
}
```

---

### **3️⃣ ContactServiceImpl**

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
    public Page<Contact> search(String userId, String keyword, Pageable pageable) {
        if (keyword == null || keyword.trim().isEmpty()) {
            return contactRepo.findByUserId(userId, pageable);
        }
        return contactRepo.searchByUserIdAndKeyword(userId, keyword.trim(), pageable);
    }

    @Override
    public Page<Contact> getFavorites(String userId, Pageable pageable) {
        return contactRepo.findFavoritesByUserId(userId, pageable);
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

### **4️⃣ Controller**

```java
package com.pronexa.connect.controllers;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

import com.pronexa.connect.entities.Contact;
import com.pronexa.connect.entities.User;
import com.pronexa.connect.services.ContactService;
import com.pronexa.connect.services.UserService;
import com.pronexa.connect.utils.Helper;

@Controller
public class ContactController {

    @Autowired
    private ContactService contactService;

    @Autowired
    private UserService userService;

    @GetMapping
    public String listContacts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "6") int size,
            @RequestParam(required = false) String keyword,
            @RequestParam(required = false) Boolean favorite,
            Model model,
            Authentication authentication
    ) {
        String email = Helper.getLoggedInUserEmail(authentication);
        User user = userService.getUserByEmail(email).orElse(null);
        if (user == null) return "redirect:/login";

        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        Page<Contact> contactsPage;

        if (keyword != null && !keyword.isBlank()) {
            contactsPage = contactService.search(user.getId(), keyword, pageable);
            model.addAttribute("keyword", keyword);
        } else if (favorite != null && favorite) {
            contactsPage = contactService.getFavorites(user.getId(), pageable);
            model.addAttribute("favorite", true);
        } else {
            contactsPage = contactService.getByUserId(user.getId(), pageable);
        }

        model.addAttribute("contacts", contactsPage.getContent());
        model.addAttribute("currentPage", page);
        model.addAttribute("totalPages", contactsPage.getTotalPages());
        model.addAttribute("pageSize", size);

        return "user/contacts";
    }
}
```

---

## **Frontend – `contacts_list.html` (Thymeleaf + Tailwind)**

* Supports **search**, **favorite filter**, **pagination**
* Responsive grid of cards with **image, links, actions, favorite star**

> Full HTML as provided in your Day 24 notes.

```html
<!DOCTYPE html>
<html lang="en" th:replace="~{base :: parent(~{::#content}, ~{::title}, ~{})}">

<head>
    <title>Dashboard - Contacts</title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.1.1/css/all.min.css" />
</head>

<body>
    <div id="content" class="flex flex-col p-6 bg-gray-50 sm:ml-64">

        <!-- Sidebar -->
        <div th:replace="~{User/sidebar :: sidebar}"></div>

        <!-- Controls: Search + Favorites + Add -->
        <div class="mb-4 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3">
            <form th:action="@{/user/contacts}" method="get" class="flex items-center gap-2">
                <input type="hidden" name="page" value="0" />
                <input type="text" name="keyword" th:value="${keyword}" placeholder="Search by name or email..."
                    class="px-3 py-2 border rounded-full w-64" />
                <button type="submit" class="px-4 py-2 bg-blue-600 text-white rounded-full">Search</button>
                <a th:href="@{/user/contacts}" class="px-3 py-2 bg-gray-200 rounded-full ml-4">Clear</a>
            </form>

            <div class="flex items-center gap-2">
                <a th:if="${favorite == null || !favorite}" th:href="@{/user/contacts(favorite=true)}"
                    class="px-4 py-2 bg-yellow-500 text-white rounded-full">Show Favorites</a>
                <a th:if="${favorite != null && favorite}" th:href="@{/user/contacts}"
                    class="px-4 py-2 bg-gray-400 text-white rounded-full">Show All</a>

                <a th:href="@{/user/contacts/add}" class="px-4 py-2 bg-green-600 text-white rounded-full">+ Add
                    Contact</a>
            </div>
        </div>

        <!-- Contacts Grid -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
            <div th:each="contact : ${contacts}" class="bg-white rounded-lg shadow p-4 flex flex-col">
                <div class="flex items-center gap-3">
                    <img th:src="${contact.picture != null && contact.picture != '' ? contact.picture : '/css/default-user.jpeg'}"
                        alt="Contact Image"
                        class="w-16 h-16 rounded-full object-cover border-2 border-gray-300 dark:border-gray-300" />


                    <div class="flex-1">
                        <div class="flex items-center justify-between gap-2">
                            <div>
                                <h3 class="text-lg font-semibold uppercase text-blue-700 dark:text-blue-400"
                                    th:text="${contact.name}">Name</h3>

                                <p class="text-sm text-gray-500" th:text="${contact.email}">email@example.com</p>
                            </div>

                            <div class="text-xl">
                                <span th:if="${contact.favorite}" class="text-yellow-500">★</span>
                                <span th:unless="${contact.favorite}" class="text-gray-300">☆</span>
                            </div>
                        </div>

                        <p class="mt-2 text-sm text-gray-600">
                            <i class="fa-solid fa-phone mr-2"></i>
                            <span th:text="'+91 ' + ${contact.phoneNumber}">+91 9xxxxxxxxx</span>
                        </p>

                    </div>
                </div>

                <!-- Links & Actions -->
                <div class="mt-4 flex items-center justify-between">
                    <div class="flex items-center gap-3 text-gray-600">
                        <a th:if="${contact.linkedInLink != null && contact.linkedInLink != ''}"
                            th:href="${contact.linkedInLink}" target="_blank" class="hover:text-blue-600">
                            <i class="fa-brands fa-linkedin"></i>
                        </a>
                        <a th:if="${contact.websiteLink != null && contact.websiteLink != ''}"
                            th:href="${contact.websiteLink}" target="_blank" class="hover:text-gray-800">
                            <i class="fa-solid fa-link"></i>
                        </a>
                    </div>

                    <div class="flex items-center gap-2">
                        <a th:href="@{'/user/contacts/view/' + ${contact.id}}"
                            class="text-blue-400 hover:text-blue-900"><i class="fa-solid fa-eye"></i></a>
                        <a th:href="@{'/user/contacts/edit/' + ${contact.id}}"
                            class="text-green-400 hover:text-green-900"><i class="fa-solid fa-user-pen"></i></a>
                        <a th:href="@{'/user/contacts/delete/' + ${contact.id}}" class="text-red-400 hover:text-red-900"
                            onclick="return confirm('Are you sure?')"><i class="fa-solid fa-trash"></i></a>
                    </div>
                </div>
            </div>
        </div>

        <!-- Pagination -->
        <div class="mt-6 flex justify-center items-center gap-2">
            <a th:if="${currentPage > 0}"
                th:href="@{/user/contacts(page=${currentPage - 1}, keyword=${keyword}, favorite=${favorite})}"
                class="px-3 py-1 border rounded">Previous</a>

            <span th:each="i : ${#numbers.sequence(0, totalPages - 1)}">
                <a th:href="@{/user/contacts(page=${i}, keyword=${keyword}, favorite=${favorite})}" th:text="${i + 1}"
                    th:classappend="${i == currentPage} ? 'font-bold text-blue-600' : 'text-gray-600'"
                    class="px-3 py-1 border rounded hover:bg-gray-100"></a>
            </span>

            <a th:if="${currentPage < totalPages - 1}"
                th:href="@{/user/contacts(page=${currentPage + 1}, keyword=${keyword}, favorite=${favorite})}"
                class="px-3 py-1 border rounded">Next</a>
        </div>

    </div>
</body>

</html>
```

---

## **✅ Notes**

1. **Priority logic**: `search → favorites → all contacts`.
2. `page` defaults to `0` and `size` defaults to `6`.
3. Filters (`keyword`, `favorite`) are persisted across pagination links.
4. Case-insensitive search via JPQL `LOWER() LIKE`.
5. Customize grid, cards, or page size in frontend as needed.
6. Supports **LinkedIn** and **Website** links.
7. Favorite contacts shown with **★** or **☆**.


