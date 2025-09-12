
# ✅ Day 7 – Part 1: JPA Entities & ORM in PronexaConnect

---

## 🧩 1. What are Entities & ORM in JPA?

**Entity:**  
A Java class mapped to a database table.

**ORM (Object Relational Mapping):**  
Technique to map Java classes with DB tables (Hibernate does this internally via JPA).

**Example:**  
`User.java` ↔ `users` table

---

## 🗃️ 2. Entity Class Structure

**Package:** `com.nexaconnect.entities`

### ✅ Essential Annotations

| Annotation                 | Purpose |
| -------------------------- | ------ |
| `@Entity`                  | Marks this class as a JPA entity. |
| `@Table(name="users")`     | Optional, gives the DB table a custom name. |
| `@Id`                      | Defines the primary key. |
| `@Column`                  | Customize columns like `unique`, `length`, etc. |
| `@Enumerated(EnumType.STRING)` | Saves enum values as a string instead of ordinal. |

### ➤ Example: `User.java`

```java
package com.pronexa.connect.entities;

import jakarta.persistence.*;
import java.util.*;
import lombok.*;

@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder

@Entity
@Table(name = "users")
public class User {

    @Id
    private String userId;

    @Column(name = "user_name", nullable = false)
    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    private String password;

    @Column(length = 1000)
    private String about;

    @Column(length = 1000)
    private String profilePic;

    private String phoneNumber;

    private boolean enabled = false;

    private boolean emailVerified = false;

    private boolean phoneNumberVerified = false;

    @Enumerated(EnumType.STRING)
    private Providers provider = Providers.SELF;

    private String providerUserId;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY, orphanRemoval = true)
    private List<Contact> contacts = new ArrayList<>();
}
````

---

## 🧾 3. Enum for Social Login Providers

```java
package com.pronexa.connect.entities;

public enum Providers {
    SELF, GOOGLE, GITHUB
}
```

**Explanation:**
Helps track how the user registered (own, Google, etc.).
Stored in DB as a readable string, not an integer (`EnumType.STRING`).

---

## 🛠️ 4. Contact Entity – OneToMany Side

Each user can have multiple contacts. This is the `@ManyToOne` side in the `Contact` entity.

```java
package com.pronexa.connect.entities;

import java.util.*;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Contact {

    @Id
    private String id;
    private String name;
    private String email;
    private String phoneNumber;
    private String address;
    private String picture;

    @Column(length = 1000)
    private String description;
    private boolean favorite = false;
    private String websiteLink;
    private String linkedInLink;
    private String cloudinaryImagePublicId;

    @ManyToOne // Many contacts can belong to one user
    private User user;

    @OneToMany(mappedBy = "contact", cascade = CascadeType.ALL, fetch = FetchType.EAGER, orphanRemoval = true)
    private List<SocialLink> links = new ArrayList<>();
}
```

**Explanation:**

* Defines a one-to-many relationship where one contact can have multiple social links.
* `mappedBy = "contact"` specifies that the `contact` field in `SocialLink` owns the relationship.
* `cascade = CascadeType.ALL` propagates all operations from Contact to SocialLink.
* `fetch = FetchType.EAGER` loads the social links immediately when the contact is loaded.
* `orphanRemoval = true` automatically deletes social links if they are removed from the list.

---

## 🔗 5. SocialLink Entity – Mapping with Contact

Each contact can have multiple social links like LinkedIn, Twitter, etc.

```java
package com.pronexa.connect.entities;

import jakarta.persistence.*;
import lombok.*;

@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder

@Entity
public class SocialLink {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String link;
    private String title;

    @ManyToOne
    private Contact contact;
}
```

**Explanation:**
Many social links can belong to one contact.

---

## ⚙️ 6. Lombok Annotations

Used to reduce boilerplate code.

| Annotation            | Purpose                                       |
| --------------------- | --------------------------------------------- |
| `@Getter`/`@Setter`   | Automatically generates getters/setters       |
| `@NoArgsConstructor`  | Default constructor                           |
| `@AllArgsConstructor` | Constructor with all fields                   |
| `@Builder`            | Helps to create objects using builder pattern |

**Example Usage:**

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
public class SocialLink {
    ...
}
```

---

## 🔄 7. Cascading & Orphan Removal

| Feature                     | Explanation                                                                    |
| --------------------------- | ------------------------------------------------------------------------------ |
| `cascade = CascadeType.ALL` | All operations (save, delete, update) on parent will reflect on children.      |
| `orphanRemoval = true`      | If a child is removed from the parent list, it gets deleted from the database. |

---

## 📌 8. Summary of Relationships

| Entity     | Relation  | With       | Type         |
| ---------- | --------- | ---------- | ------------ |
| User       | 1 -> Many | Contact    | `@OneToMany` |
| Contact    | Many -> 1 | User       | `@ManyToOne` |
| Contact    | 1 -> Many | SocialLink | `@OneToMany` |
| SocialLink | Many -> 1 | Contact    | `@ManyToOne` |

---

✅ Final Notes

* Run your Spring Boot app → Hibernate auto-generates tables based on your entities (`spring.jpa.hibernate.ddl-auto=update`).
* Make sure to annotate the main class with `@EntityScan("com.nexaconnect.entities")` if necessary.
