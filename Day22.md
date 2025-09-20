

# ✅ **Day 22 – Upload Contact Image to Cloud (Cloudinary)**

### 🎯 **Objective**

* Upload contact images to **Cloudinary** instead of storing locally.
* Keep images accessible via URL and optionally preview them before submitting.

---

## 🛠️ **1️⃣ Add Cloudinary Dependency**

```xml
<dependency>
    <groupId>com.cloudinary</groupId>
    <artifactId>cloudinary-http5</artifactId>
    <version>2.0.0</version>
</dependency>
```

---

## 🛠️ **2️⃣ Configure Cloudinary Properties**

```properties
# Cloudinary credentials
cloudinary.cloud.name=${CLOUDINARY_CLOUD_NAME:duwvmlbbxsj}
cloudinary.api.key=${CLOUDINARY_API_KEY:7738613779427699342}
cloudinary.api.secret=${CLOUDINARY_API_SECRET:jMlOHbGLUwA0rCDXPkC8Q21mFEakeW}
```

> Use environment variables in production for security.

---

## 🛠️ **3️⃣ Create ImageService Interface**

```java
package com.pronexa.connect.services;

import org.springframework.web.multipart.MultipartFile;

public interface ImageService {
    String uploadImage(MultipartFile contactImage, String filename);
    String getUrlFromPublicId(String publicId);
}
```

---

## 🛠️ **4️⃣ Implement ImageService**

```java
package com.pronexa.connect.services.impl;

import java.io.IOException;
import java.util.Map;

import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import com.cloudinary.Cloudinary;
import com.cloudinary.Transformation;
import com.cloudinary.utils.ObjectUtils;
import com.pronexa.connect.services.ImageService;

@Service
public class ImageServiceImpl implements ImageService {

    private final Cloudinary cloudinary;

    public ImageServiceImpl(Cloudinary cloudinary) {
        this.cloudinary = cloudinary;
    }

    @Override
    public String uploadImage(MultipartFile contactImage, String filename) {
        try {
            Map<?, ?> uploadResult = cloudinary.uploader().upload(
                    contactImage.getBytes(),
                    ObjectUtils.asMap(
                            "public_id", filename,
                            "overwrite", true,
                            "resource_type", "image"
                    )
            );
            return (String) uploadResult.get("secure_url");
        } catch (IOException e) {
            throw new RuntimeException("Image upload failed: " + e.getMessage(), e);
        }
    }

    @Override
    public String getUrlFromPublicId(String publicId) {
        return cloudinary.url()
                .transformation(new Transformation<>().width(300).height(300).crop("fill"))
                .generate(publicId);
    }
}
```

---

## 🛠️ **5️⃣ AppConfig for Cloudinary**

```java
package com.pronexa.connect.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.cloudinary.Cloudinary;
import com.cloudinary.utils.ObjectUtils;

@Configuration
public class AppConfig {

    @Value("${cloudinary.cloud.name}")
    private String cloudName;

    @Value("${cloudinary.api.key}")
    private String apiKey;

    @Value("${cloudinary.api.secret}")
    private String apiSecret;

    @Bean
    public Cloudinary cloudinary() {
        return new Cloudinary(ObjectUtils.asMap(
                "cloud_name", cloudName,
                "api_key", apiKey,
                "api_secret", apiSecret
        ));
    }
}
```

---

## 🛠️ **6️⃣ Controller – Upload Image & Save Contact**

```java
@PostMapping("/add")
public String saveContact(
        @Valid @ModelAttribute ContactForm contactForm,
        BindingResult result,
        Authentication authentication,
        RedirectAttributes redirectAttributes
) {

    // 1️⃣ Validate form
    if (result.hasErrors()) {
        redirectAttributes.addFlashAttribute("message",
                Message.builder()
                       .content("Please correct the highlighted errors.")
                       .type(MessageType.red)
                       .build());
        return "user/add_contact";
    }

    // 2️⃣ Get logged-in user
    User user = userService.getUserByEmail(Helper.getLoggedInUserEmail(authentication)).orElse(null);
    if (user == null) return "redirect:/login";

    // 3️⃣ Map form → entity
    Contact contact = new Contact();
    contact.setName(contactForm.getName());
    contact.setEmail(contactForm.getEmail());
    contact.setPhoneNumber(contactForm.getPhoneNumber());
    contact.setAddress(contactForm.getAddress());
    contact.setDescription(contactForm.getDescription());
    contact.setFavorite(contactForm.isFavorite());
    contact.setLinkedInLink(contactForm.getLinkedInLink());
    contact.setWebsiteLink(contactForm.getWebsiteLink());
    contact.setUser(user);

    // 4️⃣ Upload image if provided
    if (contactForm.getContactImage() != null && !contactForm.getContactImage().isEmpty()) {
        String uniqueFileName = UUID.randomUUID().toString();
        String imageUrl = imageService.uploadImage(contactForm.getContactImage(), uniqueFileName);
        contact.setPicture(imageUrl);
        contact.setCloudinaryImagePublicId(uniqueFileName);
    }

    // 5️⃣ Save contact
    contactService.save(contact);

    // 6️⃣ Success message
    redirectAttributes.addFlashAttribute("message",
            Message.builder().content("You have successfully added a new contact!")
                   .type(MessageType.green).build());

    return "redirect:/user/contacts/add";
}
```

---

## 🛠️ **7️⃣ Preview Image in Form**

**HTML Preview:**

```html
<img id="contactImagePreview"
     class="w-32 h-32 mx-auto rounded-lg shadow-lg object-cover transition-transform duration-300 hover:scale-105 hover:border-4 hover:border-blue-400" />
```

**JS for Preview:**

```javascript
document.addEventListener("DOMContentLoaded", function () {
    const input = document.getElementById("contactImage");
    const preview = document.getElementById("contactImagePreview");

    input.addEventListener("change", function () {
        const file = this.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = e => preview.src = e.target.result;
            reader.readAsDataURL(file);
        } else {
            preview.src = "";
        }
    });
});
```

> This ensures **users see the image before submission**.

---

### ✅ **Outcome**

* Contact images are uploaded to **Cloudinary** with a unique filename.
* URL is stored in the DB (`contact.picture`).
* Form previews the image **client-side**.
* Works seamlessly with **Spring Boot + Thymeleaf + MultipartFile**.

---

