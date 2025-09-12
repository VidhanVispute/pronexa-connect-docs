
# ✅ Day 6 – Part 1: Responsive Navbar with TailwindCSS + Flowbite with Dark Theme Mode

---

## 🎯 Objective

Create a reusable and responsive navbar fragment using **TailwindCSS** and **Flowbite**, integrated with Thymeleaf using `th:replace`. Add routing links to different pages like Home, Services, About, Contact, and buttons for Login & Signup. Also include a Dark Mode toggle.

---

## 📄 1. `navbar.html` (Fragment)

**Location:** `src/main/resources/templates/navbar.html`

```html
<nav
    data-th-fragment="navbar"
    class="bg-white dark:bg-gray-900 fixed w-full z-20 top-0 start-0 border-b border-gray-200 dark:border-gray-600"
>
    <div class="max-w-screen-xl flex flex-wrap items-center justify-between mx-auto p-4">
        <!-- logo code start -->
        <a class="flex items-center space-x-3 rtl:space-x-reverse">
            <!-- Logo with Adjusted Size -->
            <img th:src="@{/css/logo.png}" class="h-11 w-11" alt="NexaConnect Logo" />

            <!-- Text with Emblema One Font -->
            <span class="self-center text-3xl font-bold text-gray-900 dark:text-white" style="font-family: 'Emblema One', cursive;">
                Nexa<span class="text-blue-500">Connect</span>
            </span>
        </a>
        <!-- logo code end -->

        <div class="flex gap-2 md:order-2 space-x-3 md:space-x-0 rtl:space-x-reverse">
            <!-- Login Button -->
            <a data-th-href="@{/login}" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-4 py-2 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">Login</a>

            <!-- Signup Button -->
            <a data-th-href="@{/signup}" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-4 py-2 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">Signup</a>

            <!-- Dark Mode Toggle Button -->
            <button id="darkModeToggle" class="relative flex items-center justify-between w-14 h-8 bg-gray-300 rounded-full p-1 transition-all duration-300 focus:outline-none shadow-md">
                <div class="w-6 h-6 bg-white rounded-full shadow-md transform transition-transform duration-300"></div>
                <span class="absolute left-2 text-yellow-500 transition-opacity duration-300 sun-icon">☀️</span>
                <span class="absolute right-2 text-gray-800 transition-opacity duration-300 moon-icon opacity-0">🌙</span>
            </button>

            <!-- Collapse Toggle for Mobile -->
            <button data-collapse-toggle="navbar-sticky" type="button" class="inline-flex items-center p-2 w-10 h-10 justify-center text-sm text-gray-500 rounded-lg md:hidden hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-gray-200 dark:text-gray-400 dark:hover:bg-gray-700 dark:focus:ring-gray-600" aria-controls="navbar-sticky" aria-expanded="false">
                <span class="sr-only">Open main menu</span>
                <svg class="w-5 h-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 17 14">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M1 1h15M1 7h15M1 13h15"/>
                </svg>
            </button>
        </div>

        <div class="items-center justify-between hidden w-full md:flex md:w-auto md:order-1" id="navbar-sticky">
            <ul class="flex flex-col p-4 md:p-0 mt-4 font-medium border border-gray-100 rounded-lg bg-gray-50 md:space-x-8 rtl:space-x-reverse md:flex-row md:mt-0 md:border-0 md:bg-white dark:bg-gray-800 md:dark:bg-gray-900 dark:border-gray-700">
                <li>
                    <a data-th-href="@{/home}" class="block py-2 px-3 text-white bg-blue-700 rounded-sm md:bg-transparent md:text-blue-700 md:p-0 md:dark:text-blue-500" aria-current="page">Home</a>
                </li>
                <li>
                    <a data-th-href="@{/about}" class="block py-2 px-3 text-gray-900 rounded-sm hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 md:dark:hover:text-blue-500 dark:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">About</a>
                </li>
                <li>
                    <a data-th-href="@{/service}" class="block py-2 px-3 text-gray-900 rounded-sm hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 md:dark:hover:text-blue-500 dark:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">Services</a>
                </li>
                <li>
                    <a data-th-href="@{/contact}" class="block py-2 px-3 text-gray-900 rounded-sm hover:bg-gray-100 md:hover:bg-transparent md:hover:text-blue-700 md:p-0 md:dark:hover:text-blue-500 dark:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">Contact</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
````

✅ You include this in your `base.html` like:

```html
<div th:replace="navbar :: navbar"></div>
```

---

## 🔁 2. Controller Mappings (Sample)

**In `PageController.java`:**

```java
@GetMapping("/home")
public String home() {
    return "home";
}

@GetMapping("/about")
public String about() {
    return "about";
}

@GetMapping("/services")
public String services() {
    return "services";
}

@GetMapping("/contact")
public String contact() {
    return "contact";
}

@GetMapping("/login")
public String login() {
    return "login";
}

@GetMapping("/signup")
public String signup() {
    return "signup";
}
```

---

# ✅ Day 6 – Part 2: Dark / Light Mode Toggle Functionality

---

## 🎯 Objective

Implement JavaScript to toggle between dark and light themes.

---

## 📄 1. Include JS Code for Theme Toggle

```javascript
document.addEventListener('DOMContentLoaded', () => {
    const darkModeToggle = document.getElementById('darkModeToggle');
    if (!darkModeToggle) return;

    const toggleCircle = darkModeToggle.querySelector('div');
    const sunIcon = darkModeToggle.querySelector('.sun-icon');
    const moonIcon = darkModeToggle.querySelector('.moon-icon');

    function setTheme(theme) {
        if (theme === 'dark') {
            document.documentElement.classList.add('dark');
            toggleCircle.style.transform = 'translateX(100%)';
            sunIcon.style.opacity = '0';
            moonIcon.style.opacity = '1';
        } else {
            document.documentElement.classList.remove('dark');
            toggleCircle.style.transform = 'translateX(0)';
            sunIcon.style.opacity = '1';
            moonIcon.style.opacity = '0';
        }
        localStorage.setItem('theme', theme);
    }

    const savedTheme = localStorage.getItem('theme') || 'light';
    setTheme(savedTheme);

    document.documentElement.setAttribute('data-loaded', 'true');

    darkModeToggle.addEventListener('click', () => {
        const newTheme = document.documentElement.classList.contains('dark') ? 'light' : 'dark';
        setTheme(newTheme);
    });
});
```

---

## 📄 2. Add Implementation of Script in `base.html`

```html
<!DOCTYPE html>
<html
    lang="en"
    th:fragment="parent(content,title,script)"
    xmlns:th="http://www.thymeleaf.org"
>
<head>
    <script>
        (function () {
            if (localStorage.getItem("theme") === "dark") {
                document.documentElement.classList.add("dark");
            }
        })();
    </script>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title th:replace="${title}">NexaConnect</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/flowbite/2.3.0/flowbite.min.css" rel="stylesheet"/>
    <link rel="stylesheet" th:href="@{/css/output.css}" />
    <link rel="stylesheet" th:href="@{/css/style.css}" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css" integrity="sha512-SnH5WK+bZxgPHs44uWIX+LLJAJ9/2PkPKZ5QiAj6Ta86w+fsb2TkcmfRyVX3pBnMFcV7oQPJkl9QevSCWr3W6A==" crossorigin="anonymous" referrerpolicy="no-referrer"/>
</head>

<body>
    <div th:if="${loggedInUser}">
        <div th:replace="~{user/user_navbar :: user-navbar}"></div>
    </div>

    <div th:if="${loggedInUser == null}">
        <div th:replace="~{navbar :: navbar}"></div>
    </div>

    <div class="p-4 pt-20">
        <section data-th-replace="${content}"></section>
    </div>

    <footer class="bg-gray-800 text-white text-center p-4">
        <p>© 2025 NexaConnect. All rights reserved.</p>
    </footer>

    <script th:src="@{/js/flowbite.min.js}"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script th:src="@{/javascript/script.js}"></script>
    <th:block data-th-replace="${script}"></th:block>
</body>
</html>
```

---

## ✅ Day 6 Summary Table

| Task                                         | Status |
| -------------------------------------------- | ------ |
| Created `navbar.html` fragment               | ✅ Done |
| Used Flowbite responsive navbar              | ✅ Done |
| Linked pages: Home, About, Services, Contact | ✅ Done |
| Buttons for Login & Signup                   | ✅ Done |
| Connected to `PageController`                | ✅ Done |
| Implement                                    |        |
