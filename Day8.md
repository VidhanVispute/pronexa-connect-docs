
# ✅ Day 8 – Designing Signup Form Using Tailwind CSS & Flowbite (register.html)

---

## ✅ 1. Goal of This Task

Create a responsive and visually appealing signup form using:

- **Tailwind CSS** – for utility-first styling  
- **Flowbite** – for prebuilt Tailwind UI components  

This form will be used in `register.html`.

**Form Elements Used:**

- `form` tag to wrap inputs  
- Input fields for name, email, password  
- Buttons: Signup & Reset  

---

## 📦 2. Setup Flowbite in Your HTML

Ensure that your `base.html` includes the Tailwind and Flowbite CDN links:

```html
<!-- Tailwind CSS -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- Flowbite -->
<script src="https://unpkg.com/flowbite@1.6.5/dist/flowbite.min.js"></script>
````

---

## 🧾 3. register.html (Signup Form Page Fragment)

```html
<!DOCTYPE html>
<html lang="en" th:replace="~{base :: parent(~{::#content}, ~{::title}, ~{})}">
<head>
    <title>Register page</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
    <!-- register.html -->
    <section id="content" class="bg-gray-50 dark:bg-gray-900 py-10">
        <div class="w-full max-w-md mx-auto bg-white p-8 rounded-lg shadow dark:bg-gray-800">
            <h2 class="text-xl font-bold text-center text-gray-900 dark:text-white mb-6">Create Account</h2>
            <form>
                <!-- Full Name -->
                <div>
                    <label for="fullName" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Full Name</label>
                    <input type="text" id="fullName" name="fullName" placeholder="John Doe"
                        class="w-full p-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required />
                </div>

                <!-- Email -->
                <div>
                    <label for="email" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Email Address</label>
                    <input type="email" id="email" name="email" placeholder="john@example.com"
                        class="w-full p-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required />
                </div>

                <!-- Password -->
                <div>
                    <label for="password" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Password</label>
                    <input type="password" id="password" name="password" placeholder="Enter a strong password"
                        class="w-full p-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                        required minlength="8" />
                    <p class="mt-1 text-xs text-gray-500 dark:text-gray-400">Minimum 8 characters required</p>
                </div>

                <!-- Contact -->
                <div>
                    <label for="contact" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Phone Number</label>
                    <input type="tel" id="contact" name="contact" placeholder="+1 (555) 123-4567"
                        class="w-full p-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required />
                </div>

                <!-- About -->
                <div>
                    <label for="about" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">About You</label>
                    <textarea id="about" name="about" rows="3" placeholder="Tell us a little about yourself..."
                        class="w-full p-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500 resize-none" required></textarea>
                </div>

                <!-- Buttons -->
                <div class="flex justify-between items-center pt-4">
                    <button type="submit"
                        class="bg-blue-600 text-white px-6 py-2.5 rounded-lg font-medium hover:bg-blue-700 focus:ring-4 focus:ring-blue-300 focus:outline-none transition-colors dark:focus:ring-blue-800">Create Account</button>

                    <button type="reset"
                        class="bg-gray-200 text-gray-900 px-6 py-2.5 rounded-lg font-medium hover:bg-gray-300 focus:ring-4 focus:ring-gray-300 focus:outline-none transition-colors dark:bg-gray-700 dark:text-white dark:hover:bg-gray-600 dark:focus:ring-gray-600">Reset</button>
                </div>
            </form>

            <!-- Login Link -->
            <div class="text-center mt-6">
                <p class="text-sm text-gray-600 dark:text-gray-400">
                    Already have an account?
                    <a href="/login"
                        class="text-blue-600 hover:text-blue-700 font-medium dark:text-blue-400 dark:hover:text-blue-300">Sign in here</a>
                </p>
            </div>
        </div>
    </section>

    <script>
        console.log("services script")
    </script>
</body>
</html>
```

---

## 🧠 4. Important Concepts

| Concept               | Explanation                                                   |
| --------------------- | ------------------------------------------------------------- |
| `form`                | Wraps all input fields; `method="post"` sends data to backend |
| `input`               | Used for data entry: `type="text"`, `email`, `password`       |
| `required`            | Ensures field is filled before submission                     |
| `button type="reset"` | Clears all fields instantly                                   |
