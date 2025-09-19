

## ✅ **Day 18 – Creating Admin Dashboard Panel**

### 🎯 **Objective**

* Separate **navigation** and **sidebar** into reusable Thymeleaf fragments.
* Show sidebar only inside **user pages** (dashboard/profile), not on public pages like home, about, etc.
* Apply conditional rendering of fragments depending on login state.

---

### 🛠 **Steps Implemented**

#### 1. **Create Fragments**

We created two fragment files inside `templates/User/`:

* `user_navbar.html` → navigation bar for logged-in users.
* `sidebar.html` → sidebar for dashboard/admin panel.

✅ Example – `user_navbar.html` (fragment):

```html
<nav data-th-fragment="user-navbar" class="bg-white dark:bg-gray-900 ...">
  <!-- Logo -->
  <a class="flex items-center">
    <img th:src="@{/css/logo.png}" class="h-11 w-11"/>
    <span class="text-3xl font-bold">Nexa<span class="text-blue-500">Connect</span></span>
  </a>

  <!-- Profile + Logout -->
  <div class="flex gap-2">
    <a data-th-href="@{/user/profile}" class="btn-green">
      <span th:text="${loggedInUser.name}"></span>
    </a>
    <a data-th-href="@{/logout}" class="btn-red">Logout</a>
  </div>
</nav>
```

✅ Example – `sidebar.html` (fragment):

```html
<aside data-th-fragment="sidebar" id="logo-sidebar" class="fixed top-[72px] left-0 ...">
   <ul class="space-y-2 font-medium">
      <li><a data-th-href="@{/user/dashboard}" class="flex items-center">Dashboard</a></li>
      <li><a data-th-href="@{/user/profile}" class="flex items-center">Profile</a></li>
      <li><a href="#" class="flex items-center">Users</a></li>
      <li><a href="#" class="flex items-center">Products</a></li>
   </ul>
</aside>
```

---

#### 2. **Conditional Import of Navbar**

In `base.html` (layout parent):

```html
<!-- If user is logged in -->
<div th:if="${loggedInUser}">
    <div th:replace="~{User/user_navbar :: user-navbar}"></div>
</div>

<!-- If user is NOT logged in -->
<div th:unless="${loggedInUser}">
    <div th:replace="~{navbar :: navbar}"></div>
</div>
```

👉 **Result**:

* Visitors (not logged in) → see public navbar (`navbar.html`).
* Logged-in users → see custom dashboard navbar (`user_navbar.html`).

---

#### 3. **Sidebar Only in User Pages**

We directly included sidebar only in `profile.html` (and can do the same for `dashboard.html`):

```html
<div id="content" class="ml-64 flex items-center justify-center">
    <!-- Sidebar -->
    <div th:replace="~{User/sidebar :: sidebar}"></div>

    <!-- Page Content -->
    <div>
        <h1>Dashboard</h1>
        <p>Name: <span th:text="${loggedInUser?.name ?: 'Guest'}"></span></p>
        <p>Email: <span th:text="${loggedInUser?.email ?: 'Not logged in'}"></span></p>
        <p>Phone: <span th:text="${loggedInUser?.phoneNumber ?: 'No number found'}"></span></p>
        <p>About: <span th:text="${loggedInUser?.about ?: 'N/A'}"></span></p>
        <p>Provider: <span th:text="${loggedInUser?.provider ?: 'SELF'}"></span></p>
    </div>
</div>
```

👉 **Reason**:

* Sidebar should not appear on public pages (`home`, `about`, `services`, `contact`).
* It only appears on logged-in **user/admin panel pages**.

---

### 📊 **Summary Table**

| File               | Purpose                                           |
| ------------------ | ------------------------------------------------- |
| `user_navbar.html` | Logged-in user navbar fragment                    |
| `sidebar.html`     | Dashboard sidebar fragment                        |
| `base.html`        | Conditional navbar rendering (guest vs logged-in) |
| `profile.html`     | Imports sidebar for logged-in profile view        |

---

### ✅ **Outcome**

* Public pages show **normal navbar** only.
* Logged-in pages show **user navbar + sidebar**.
* Dashboard layout looks like an **admin panel** (navbar top + sidebar left + content right).

---


