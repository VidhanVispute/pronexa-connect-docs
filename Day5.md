
# ✅ Day 5 – Manage Layout Using Thymeleaf Fragments (Master Page Setup)

---

## 🎯 Objective

Build a reusable master layout (`base.html`) using Thymeleaf fragments, and inject page-specific content from other views like `home.html`, `about.html`, `contact.html` using `th:replace` and dynamic placeholders.

---

## 🧱 1. Created Master Layout: `base.html`

This acts as a parent fragment for common layout elements like `<head>`, CSS, JS, Navbar, Footer, etc.

```html
<!DOCTYPE html>
<html
    lang="en"
    th:fragment="parent(content,title,script)"
    xmlns:th="http://www.thymeleaf.org"
>
<!-- Defines a Thymeleaf fragment named 'parent' with parameters content, title, and script -->
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title th:replace="${title}">NexaConnect</title>
    <!-- th:replace will show proper page title -->

    <link rel="stylesheet" th:href="@{/css/output.css}" />
    <link rel="stylesheet" th:href="@{/css/style.css}" />
</head>

<body>
    <!-- Child page content dynamically inserted here -->
    <section data-th-replace="${content}"></section>

    <footer class="bg-gray-800 text-white text-center p-4">
        <p>© 2025 NexaConnect. All rights reserved.</p>
    </footer>

    <th:block data-th-replace="${script}"></th:block>
</body>
</html>
````

---

## 📄 2. Create a Child Page Example: `home.html`

Inject content dynamically into the base layout:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="en">

<!-- Inject into base layout -->
<div th:replace="fragments/base :: parent(
    ~{::#content},
    ~{::title},
    ~{::#script}
)">

    <!-- Page Title -->
    <th:block id="title">Home | NexaConnect</th:block>

    <!-- Page Specific Content -->
    <div id="content">
        <h1 class="text-2xl font-bold text-blue-600">
            Welcome, <span th:text="${name}"></span>!
        </h1>
        <p>This is the Home Page of NexaConnect.</p>
    </div>

    <!-- Page Specific JS -->
    <th:block id="script">
        <script>
            console.log("Home Page Script");
        </script>
    </th:block>

</div>
</html>
```

---

## 🧠 Why Use This Pattern?

| Feature                     | Benefit                                                     |
| --------------------------- | ----------------------------------------------------------- |
| ✅ Centralized Layout        | Define header, navbar, footer only once                     |
| ✅ Dynamic Content Injection | Every page injects its specific content using `th:replace`  |
| ✅ Cleaner Code              | Separation of concerns: structure (base) vs content (child) |
| ✅ Maintainability           | Change layout in one place → reflects everywhere            |

---

## ✅ Day 5 Summary Table

| Task                                                    | Status |
| ------------------------------------------------------- | ------ |
| Created `base.html` with dynamic fragment="parent(...)" | ✅ Done |
| Injected title, content, script dynamically             | ✅ Done |
| Applied layout to home, about, contact pages            | ✅ Done |
| Practiced fragment injection with Tailwind and scripts  | ✅ Done |
| Cleanly separated common UI from page-specific logic    | ✅ Done |
