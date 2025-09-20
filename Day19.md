
## ✅ Day 19 – Add Contact Form (Validation Only)

### 🎯 Objective

* Build a form for adding a new contact.
* Apply **server-side validation** with `@Valid`.
* Capture data and return messages **without saving to DB**.

---

### 📂 1. Create `ContactForm` DTO

* Holds form input fields.
* Uses **Jakarta Validation** annotations (`@NotBlank`, `@Email`, `@Pattern`, etc.).
* Future support for file validation with custom annotations.

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class ContactForm {

    @NotBlank(message = "Name is required")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid Email Address [ example@gmail.com ]")
    private String email;

    @NotBlank(message = "Phone Number is required")
    @Pattern(regexp = "^[0-9]{10}$", message = "Invalid Phone Number")
    private String phoneNumber;

    @NotBlank(message = "Address is required")
    private String address;

    private String description;
    private boolean favorite;
    private String websiteLink;
    private String linkedInLink;
    private String picture;
}
```

---

### 🖼️ 2. Create Thymeleaf Form (`add_contact.html`)

* Uses `data-th-field` to bind fields.
* Shows error messages with `#fields.hasErrors`.
* Includes reset & submit buttons.
```java
* <!DOCTYPE html>
<html lang="en" th:replace="~{base :: parent(~{::#content},~{::title},~{::script})}">

<head>
  <title>Add Contact</title>
</head>

<body>
  <div id="content">
    <!-- sidebar -->

    <!-- user is logged in : sidebar -->

    <div th:if="${loggedInUser}">
      <div data-th-replace="~{user/sidebar :: sidebar}"></div>
    </div>
    <div class="flex justify-center flex-col items-center">
      <div class="sm:pl-70">

        <div class="grid grid-cols-12">
          <div class="col-span-3"></div>
          <div class="col-span-12 md:col-span-6">
            <div
              class="card block p-6 bg-white border border-gray-200 rounded-lg shadow hover:bg-gray-100 dark:bg-gray-800 dark:border-gray-700 dark:hover:bg-gray-700">


              <h1 class="text-2xl font-semibold">Add New Contact</h1>
              <p class="text-gray-500">
                This contact will be stored on cloud, you can direct email this
                client from scm...
              </p>

              <form action="" class="mt-8" data-th-action="@{'/user/contacts/add'}" data-th-object="${contactForm}"
                method="post"
 enctype="multipart/form-data">
                <!-- name form -->
                <div class="mb-3">
                  <label for="input-group-1"
                    class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Contact Name</label>
                  <div class="relative mb-1">
                    <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
                      <i class="fa-regular w-4 h-4 fa-user"></i>
                    </div>
                    <input type="text"
                     data-th-field="*{name}"
                      class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full ps-10 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                      placeholder="name" />
                      <p
                    class="text-red-500"
                    data-th-if="${#fields.hasErrors('name')}"
                    data-th-errors="*{name}"
                  >
                    Invalid Name
                  </p>
                  </div>
                </div>

                <!-- email form -->
                <div class="mb-3">
                  <label for="input-group-1"
                    class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Contact Email</label>
                  <div class="relative mb-1">
                    <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
                      <i class="fa-regular w-4 h-4 fa-envelope"></i>
                    </div>
                    <input type="text"
                     data-th-field="*{email}"
                      class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full ps-10 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                      placeholder="example@gmail.com" />
                  </div>
                  <p
                    class="text-red-500"
                    data-th-if="${#fields.hasErrors('email')}"
                    data-th-errors="*{email}"
                  >
                    Invalid Name
                  </p>
                </div>

                <!-- phone number  -->
                <div class="mb-3">
                  <label for="input-group-1"
                    class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Contact Phone</label>
                  <div class="relative mb-1">
                    <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
                      <i class="fa-solid w-4 h-4 fa-phone"></i>
                    </div>
                    <input type="text"
                     data-th-field="*{phoneNumber}"
                      class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full ps-10 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                      placeholder="9823525525" />
                  </div>
                  <p
                    class="text-red-500"
                    data-th-if="${#fields.hasErrors('phoneNumber')}"
                    data-th-errors="*{phoneNumber}"
                  >
                    Invalid phoneNumber
                  </p>
                </div>

                <!-- address -->

                <div class="mb-3">
                  <label for="message" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Contact
                    Address</label>
                  <textarea rows="4"  data-th-field="*{address}"
                    class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                    placeholder="Address of the contact"></textarea>
                    <p
                    class="text-red-500"
                    data-th-if="${#fields.hasErrors('address')}"
                    data-th-errors="*{address}"
                  >
                    Invalid address
                  </p>
                </div>

                <!-- description -->

                <div class="mb-3">
                  <label for="message" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Contact
                    Description</label>
                  <textarea rows="4" data-th-field="*{description}"
                    class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                    placeholder="Write about your contact"></textarea>
                </div>

                <!-- social links -->
                <div class="flex space-x-3 mb-3">
                  <div class="w-full">
                    <!-- website  link  -->
                    <div class="mb-3">
                      <div class="relative mb-6">
                        <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
                          <i class="fa-solid w-4 h-4 fa-earth-americas"></i>
                        </div>
                        <input
                         data-th-field="*{websiteLink}"
                          class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full ps-10 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                          placeholder="htt://learncodewithdurgesh.com/" />
                      </div>
                    </div>
                  </div>
                  <div class="w-full">
                    <!-- linkedin  link  -->
                    <div class="mb-3">
                      <div class="relative mb-6">
                        <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
                          <i class="fa-brands w-4 h-4 fa-linkedin"></i>
                        </div>
                        <input type="text"
                        data-th-field="*{linkedInLink}"
                          class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full ps-10 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                          placeholder="htt://learncodewithdurgesh.com/" />
                      </div>
                    </div>
                  </div>
                </div>

                <!-- contact image field -->
                <!-- <div class="mb-3">
                  <label class="block mb-2 text-sm font-medium text-gray-900 dark:text-white" for="large_size">Contact
                    Image</label>
                  <input id="image_file_input"
                   data-th-field="*{contactImage}"
                    class="block w-full mb-2 text-xs text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 dark:text-gray-400 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400"
                    type="file" />


                  <img class="max-h-52 mx-auto rounded-lg shadow m-3" src="" id="upload_image_preview" alt="" />
                </div> -->

                <!-- favorite checkbox -->

                <div class="flex items-center mb-3">
                  <input type="checkbox" data-th-field="*{favorite}"
                    class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
                  <label for="default-checkbox" class="ms-2 text-sm font-medium text-gray-900 dark:text-gray-300">Is
                    this contact is your favorite one ?</label>
                </div>

                <div class="button-container text-center">
                  <button type="submit"
                    class="px-3 py-2 dark:bg-blue-600 hover:dark:bg-blue-700 rounded bg-black text-white">
                    Add Contact
                  </button>
                  <button type="reset"
                    class="px-3 py-2 dark:bg-blue-600 hover:dark:bg-blue-700 rounded bg-black text-white">
                    Reset
                  </button>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>

    <script data-th-src="@{'/javascript/admin.js'}"></script>
    <script>
      console.log("this is profile page");
    </script>
</body>

</html>
```
---

### 🎛️ 3. Controller Method (Validation Only)

✅ Industry Recommendation

 - Use RedirectAttributes.addFlashAttribute for temporary messages after form submission + redirect.
 - Use HttpSession only for real session data, like logged-in user info, shopping cart, etc.
 - Most modern Spring Boot applications never use session manually for flash messages; RedirectAttributes is the standard.

```java
@PostMapping("/add")
public String saveContact(
        @Valid @ModelAttribute ContactForm contactForm,
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

    // 3️⃣ Convert form → entity (but don’t save yet)
    Contact contact = new Contact();
    contact.setName(contactForm.getName());
    contact.setEmail(contactForm.getEmail());
    contact.setPhoneNumber(contactForm.getPhoneNumber());
    contact.setAddress(contactForm.getAddress());
    contact.setDescription(contactForm.getDescription());
    contact.setWebsiteLink(contactForm.getWebsiteLink());
    contact.setLinkedInLink(contactForm.getLinkedInLink());
    contact.setFavorite(contactForm.isFavorite());
    contact.setUser(user);

    // 4️⃣ Simulate success message
    redirectAttributes.addFlashAttribute("message",
        Message.builder()
               .content("You have successfully added a new contact!")
               .type(MessageType.green)
               .build());

    logger.info("New contact (not saved): {}", contact);

    return "redirect:/user/contacts/add";
}
```
---
### 🖼️ 2.Adding Flash Session to add_contact.html

```
<!-- ✅ Flash/Session Message goes here -->
              <div id="flashMessage" th:if="${message}"
                th:class="'w-full p-3 mb-4 rounded border ' + 
               (${message.type.name()} eq 'green' ? 'border-green-400 bg-green-100 text-green-800' : 'border-red-400 bg-red-100 text-red-800')">
                <span th:text="${message.content}"></span>
              </div>
```
---

### 🔑 Key Purpose of Each Part

* **DTO (ContactForm)** → ensures input data is structured & validated.
* **Thymeleaf form** → binds HTML inputs with server-side validation messages.
* **Controller with @Valid** → checks input correctness before processing.
* **Session messages** → provide feedback to user.
* **No DB save yet** → ensures validation flow works before persistence logic.

---
