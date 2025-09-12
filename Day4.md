

# ✅ Day 4 – Thymeleaf Fragments (Header, Footer, Sidebar, etc.)

---

## 🎯 Objective

Understand and implement **Thymeleaf fragments** to **reuse common HTML components** like headers, footers, sidebars, and navigation bars across multiple pages, improving **modularity and maintainability**.

---

## 📘 What is a Fragment in Thymeleaf?

> A **fragment** in Thymeleaf is a **reusable piece of HTML** (e.g., header, navbar, footer) that can be defined once and included in multiple templates to avoid repetition and make code cleaner.

---

## 🛠️ 1. Created a Common Layout Directory

All fragment files were placed in:

```

src/main/resources/templates/fragments/

````

---

## 🧩 2. Created Fragment Files

### `header.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title th:text="${title}">NexaConnect</title>
    <link rel="stylesheet" th:href="@{/css/output.css}">
</head>
<body>
````

### `navbar.html`

```html
<div class="bg-blue-600 p-4 text-white">
    <h1 class="text-xl font-bold">NexaConnect</h1>
</div>
```

### `footer.html`

```html
<footer class="bg-gray-800 text-white text-center p-4">
    <p>© 2025 NexaConnect. All rights reserved.</p>
</footer>
</body>
</html>
```

---

## 🔗 3. How to Use Fragments in Templates

### `home.html` Example

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" th:replace="fragments/header :: head">

<body>
    <!-- Navbar -->
    <div th:replace="fragments/navbar :: navbar"></div>

    <!-- Main Content -->
    <div class="container mx-auto mt-4">
        <h2 class="text-2xl font-semibold text-gray-800">Welcome to NexaConnect</h2>
        <p>Hello, <span th:text="${name}"></span></p>
    </div>

    <!-- Footer -->
    <div th:replace="fragments/footer :: footer"></div>
</body>
</html>
```

---

## 🧪 4. Fragment Syntax (Reference)

| Syntax                          | Purpose                                                |
| ------------------------------- | ------------------------------------------------------ |
| `th:replace="file :: fragment"` | Replaces the current tag with the fragment             |
| `th:include="file :: fragment"` | Includes the fragment content inside the current tag   |
| `th:insert="file :: fragment"`  | Inserts the fragment as a child inside the current tag |

---

## 🎯 5. Pass Variables to Fragments (Optional Advanced Use)

You can pass local variables to fragments using:

```html
<div th:replace="fragments/navbar :: navbar (~{::name='Nexa'})"></div>
```

In the fragment file:

```html
<h1 th:text="${name}">Default Name</h1>
```

---

## 🧠 Why Use Thymeleaf Fragments?

| Benefit           | Explanation                           |
| ----------------- | ------------------------------------- |
| ✅ Reusability     | Define once, use everywhere           |
| ✅ Maintainability | Update once, reflects everywhere      |
| ✅ Cleaner Code    | Separate layout from content          |
| ✅ MVC Friendly    | Integrates naturally with Spring Boot |

---

## 📝 Summary of Day 4

| Task                                     | Status |
| ---------------------------------------- | ------ |
| Understood Fragment Concepts             | ✅ Done |
| Created Header, Navbar, Footer Fragments | ✅ Done |
| Used `th:replace` to include fragments   | ✅ Done |
| Practiced dynamic values in fragments    | ✅ Done |
| Applied Tailwind styles inside fragments | ✅ Done |
