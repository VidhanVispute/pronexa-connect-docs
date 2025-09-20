

# ✅ **Day 21 – Custom File Validation**

### 🎯 **Objective**

* Validate uploaded contact images for **size** and **file type** using a **custom annotation**.
* Keep validation **reusable** for any file field in the project.

---

## 🛠️ **1️⃣ Create Custom Annotation**

```java
package com.pronexa.connect.validation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

@Documented
@Constraint(validatedBy = FileValidator.class)
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidFile {

    String message() default "Invalid file";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    // Custom params
    long maxSize() default 5 * 1024 * 1024; // 5MB default

    String[] allowedTypes() default { "image/jpeg", "image/png" }; // default allowed types
}
```

**Explanation:**

* `@Constraint(validatedBy = FileValidator.class)` → links annotation to validator logic.
* `maxSize` and `allowedTypes` → configurable parameters for flexibility.

---

## 🛠️ **2️⃣ Create FileValidator**

```java
package com.pronexa.connect.validation;

import org.springframework.web.multipart.MultipartFile;
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class FileValidator implements ConstraintValidator<ValidFile, MultipartFile> {

    private long maxSize;
    private String[] allowedTypes;

    @Override
    public void initialize(ValidFile constraintAnnotation) {
        this.maxSize = constraintAnnotation.maxSize();
        this.allowedTypes = constraintAnnotation.allowedTypes();
    }

    @Override
    public boolean isValid(MultipartFile file, ConstraintValidatorContext context) {

        // allow empty file (optional upload)
        if (file == null || file.isEmpty()) {
            return true;
        }

        // 1️⃣ Check file size
        if (file.getSize() > maxSize) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(
                    "File size exceeds limit of " + (maxSize / 1024 / 1024) + "MB")
                   .addConstraintViolation();
            return false;
        }

        // 2️⃣ Check file type
        String contentType = file.getContentType();
        for (String type : allowedTypes) {
            if (type.equalsIgnoreCase(contentType)) {
                return true;
            }
        }

        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate("Invalid file type. Allowed: jpg, png")
               .addConstraintViolation();

        return false;
    }
}
```

**Explanation:**

* Validates **size** and **type**.
* Returns `true` for **optional empty files**.
* Uses `ConstraintValidatorContext` to provide **custom error messages**.

---

## 🛠️ **3️⃣ Use in ContactForm**

```java
@ValidFile(
    message = "Please upload a valid image (JPG/PNG, max 5MB)",
    maxSize = 5 * 1024 * 1024, // 5MB
    allowedTypes = {"image/jpeg", "image/png"}
)
private MultipartFile contactImage;
```

**Explanation:**

* Annotation is now reusable for **any file upload field**.
* Custom message is displayed if validation fails.

---

## 🛠️ **4️⃣ Application Properties**

```properties
# Increase file upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=15MB
```

**Why this is needed:**

* Ensures **Spring Boot** can handle files larger than the default 1MB limit.

---

## 🔄 **Data Flow**

```
Form (file input)
   ⬇
ContactForm (MultipartFile)
   ⬇
@ValidFile → FileValidator
   ⬇
ValidationResult
   ⬇
Controller handles errors via BindingResult
```

---

## ✅ **Outcome**

* Custom annotation **ValidFile** works for **JPG/PNG up to 5MB**.
* Validation integrated with **Spring Boot / Thymeleaf BindingResult**.
* Error messages appear in form automatically.

---
