
# PronexaConnect – Day 10 Notes: Save Signup Form Data to Database

---

## ✅ Day 10 – Save Signup Form Data to Database

### 🎯 Objective
- Extract data from the signup form (`UserForm`)
- Convert it to the **actual `User` entity**
- Save it into the **database** using Spring Data JPA and service layer
- Handle validations and exceptions

---

## 📝 1. Create Service Interface – `UserService`

```java
@Service
public interface UserService {

    User saveUser(User user);

    Optional<User> getUserById(String id);

    Optional<User> updateUser(User user);

    void deleteUser(String id);

    boolean isUserExists(String userId);

    boolean isUserExistsByEmail(String email);

    List<User> getAllUser();

}
````

> **Key Points:**
>
> * Declares all methods related to User operations
> * Keeps business logic separate from controllers
> * Interface allows flexibility and testability

---

## 🛠️ 2. Create Repository – `UserRepo`

```java
@Repository
public interface UserRepo extends JpaRepository<User, String> {
    Optional<User> findByEmail(String email);
}
```

> **How `findByEmail` works:**
>
> * Spring Data JPA parses method name at runtime
> * Executes query like:
>
> ```sql
> SELECT u FROM User u WHERE u.email = :email
> ```
>
> * No explicit SQL or JPQL needed

**Benefits:**

* Type-safe, declarative, no boilerplate
* Easy to define new queries: `findByNameAndStatus`, `countByRole`, etc.

---

## ⚙️ 3. Service Implementation – `UserServiceImpl`

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepo userRepo;

    @Override
    public User saveUser(User user) {
        user.setUserId(UUID.randomUUID().toString());
        return userRepo.save(user);
    }

    @Override
    public Optional<User> getUserById(String id) {
        return userRepo.findById(id);
    }

    @Override
    public Optional<User> updateUser(User user) {
        User existingUser = userRepo.findById(user.getUserId())
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));

        existingUser.setName(user.getName());
        existingUser.setEmail(user.getEmail());
        existingUser.setPassword(user.getPassword());
        existingUser.setPhoneNumber(user.getPhoneNumber());
        existingUser.setAbout(user.getAbout());

        User updatedUser = userRepo.save(existingUser);
        return Optional.of(updatedUser);
    }

    @Override
    public void deleteUser(String id) {
        User existingUser = userRepo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        userRepo.delete(existingUser);
    }

    @Override
    public boolean isUserExists(String userId) {
        return userRepo.findById(userId).isPresent();
    }

    @Override
    public boolean isUserExistsByEmail(String email) {
        return userRepo.findByEmail(email).isPresent();
    }

    @Override
    public List<User> getAllUser() {
        return userRepo.findAll();
    }
}
```

> **Notes:**
>
> * `ResourceNotFoundException` handles missing entities
> * Business logic resides here; controller stays clean
> * Uses `UUID` for user IDs

---

## 📝 4. Controller – Handling Form Submission

```java
@RequestMapping(value = "/do-register", method = RequestMethod.POST)
public String processRegister(@ModelAttribute UserForm userForm) {

    // Convert UserForm to User entity
    User user = User.builder()
        .name(userForm.getName())
        .email(userForm.getEmail())
        .password(userForm.getPassword()) // should encode password
        .phoneNumber(userForm.getPhoneNumber())
        .about(userForm.getAbout())
        .profilePic("default-profile-pic-url")
        .build();

    // Save user in DB through service layer
    User savedUser = userService.saveUser(user);

    System.out.println(savedUser + " user is saved");
    System.out.println(userForm);

    return "redirect:/login";
}
```

> **Key Points:**
>
> * `@ModelAttribute UserForm userForm` automatically binds form data
> * Maps `UserForm` → `User` entity
> * Calls service layer to save to DB
> * Redirects to login page after successful registration

---

## 🔑 5. Data Flow Recap

1. **GET /signup** → Controller returns empty `UserForm` → Thymeleaf binds form
2. User fills form → **POST /do-register**
3. `UserForm` object receives data automatically via `@ModelAttribute`
4. Controller converts to `User` entity
5. Service layer (`UserServiceImpl`) saves `User` in DB
6. User redirected to login page

---

## ✅ Summary Table – Day 10

| Task                                                          | Status |
| ------------------------------------------------------------- | ------ |
| Created `UserService` interface                               | ✅ Done |
| Created `UserRepo` extending `JpaRepository`                  | ✅ Done |
| Implemented `UserServiceImpl` with full CRUD logic            | ✅ Done |
| Added custom exception handling (`ResourceNotFoundException`) | ✅ Done |
| Updated controller to save `UserForm` to DB                   | ✅ Done |
| Verified data flow from form → User entity → DB               | ✅ Done |


```
