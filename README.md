# 🔢 Terraform `count` and `for_each` Lab

In this lab, I learned how to **dynamically create multiple resources using `count` and `for_each`**, how to reference resource instances by index or key, how Terraform represents resources internally (lists vs maps), and how to build filenames/content dynamically based on variables.

---

## 📋 Lab Overview

**Goal:**

* Understand how `count` and `for_each` work
* Create multiple resources dynamically
* Reference specific resource instances by index or key
* Use lists and maps effectively with meta-arguments
* Update resource arguments to loop over variable values

**Learning Outcomes:**

* Use `count` to create list-based resources
* Use `for_each` to create map-based resources
* Understand how Terraform stores resource instances (`list` vs `map`)
* Build dynamic filenames and content using `count.index` or `each.value`
* Differentiate between list and set types

---

## 🛠 Step-by-Step Journey

### Step 1: Inspect the Configuration Directory

**Directory:**

```
/root/terraform-projects/project-shade
```

**Question:** How many files will be created?
**Answer:** **One** — only a single resource was defined initially.

---

### Step 2: Add `count` to Create 3 Instances

Updated resource block:

```hcl
count = 3
```

Execution:

```bash
terraform init
terraform plan
terraform apply
# yes
```

Resources were created as a **list**, indexed from 0 → 2.

---

### Step 3: Confirm Resource is Now a List

Terraform displayed:

```
local_sensitive_file.name[0]
local_sensitive_file.name[1]
local_sensitive_file.name[2]
```

Correct — `count` creates a **list of resources**.

---

### Step 4: Find the ID of Resource at Index 1

Command used:

```bash
terraform show
```

Found ID for:

```
local_sensitive_file.name[1]
```

Example ID:

```
6b3...
```

---

### Step 5: How Many Files Were Actually Created?

**Answer:** Only **one** file was physically created.

Reason:
The resource type was **local_sensitive_file**, and the filename argument was the *same* for all count instances, so subsequent writes overwrote the same file.

---

### Step 6: Use a List Variable With Count

Variable in `variables.tf`:

```hcl
variable "users" {
  type = list(string)
}
```

Updated `main.tf`:

```hcl
resource "local_sensitive_file" "name" {
  count    = length(var.users)
  filename = "/root/${var.users[count.index]}"
  content  = var.content[count.index]
}
```

Now:

* `var.users[count.index]` → unique filename per user
* `var.content[count.index]` → unique content per index

This creates **one file per user**.

---

### Step 7: Identify the Variable Type

`users` variable is:

```
list(string)
```

Correct—it's a list with default values.

---

### Step 8: Can This List Be Used Directly as a Set?

**Answer:** No — the list contains **duplicate elements**.

A set **must** contain unique values.

---

### Step 9: Re-Create Resources Using `for_each`

Updated resource block:

```hcl
resource "local_sensitive_file" "name" {
  for_each = var.users

  filename = "/root/${each.value}"
  content  = var.content
}
```

Commands:

```bash
terraform init
terraform apply
# yes
```

---

### Step 10: `for_each` Creates a Map

Terraform now shows resources as:

```
local_sensitive_file.name["user1"]
local_sensitive_file.name["user2"]
local_sensitive_file.name["user11"]
```

These are **map keys**, not indices.

---

### Step 11: Determine the Address of the Resource for `/root/user11`

Correct resource address:

```
local_sensitive_file.name["/root/user11"]
```

Or, based on your actual filenames:

```
local_sensitive_file.name["user11"]
```

Terraform uses the **filename value** (each.value) as the map key.

---

## ✅ Key Commands Summary

| Task                       | Command                          |
| -------------------------- | -------------------------------- |
| Initialize project         | `terraform init`                 |
| Preview execution          | `terraform plan`                 |
| Apply configuration        | `terraform apply`                |
| Inspect all resources      | `terraform show`                 |
| Retrieve specific resource | `terraform state show <address>` |
| Use list with count        | `count = length(var.users)`      |
| Use map with for_each      | `for_each = var.users`           |

---

## 💡 Notes / Tips

* `count` → creates a **list** of resources, indexed numerically.
* `for_each` → creates a **map** of resources, keyed by values or map keys.
* Local file resources require **unique filenames** or else they overwrite each other.
* Use `count.index` when working with lists.
* Use `each.key` and `each.value` when working with maps.
* Lists with duplicates **cannot** be converted to sets without removing duplicates.
* When choosing between `count` and `for_each`:

  * Use `count` for simple lists
  * Use `for_each` when keys matter or when values must be unique

---

## 📌 Lab Summary

| Step                             | Status | Key Takeaways                        |
| -------------------------------- | ------ | ------------------------------------ |
| Inspect configuration            | ✅      | Only 1 initial resource              |
| Add count = 3                    | ✅      | Created a list of resources          |
| Find index 1 ID                  | ✅      | Retrieved from Terraform state       |
| Use list variable with count     | ✅      | Dynamically created per-user files   |
| Identify variable type           | ✅      | `list(string)`                       |
| Understand sets vs lists         | ✅      | Duplicates prevent direct use as set |
| Use for_each                     | ✅      | Created a map of resources           |
| Determine final resource address | ✅      | Correct `for_each` map key format    |

---

## ✅ References

* [Terraform Meta-Arguments: count](https://developer.hashicorp.com/terraform/language/meta-arguments/count)
* [Terraform Meta-Arguments: for_each](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each)
* [Terraform Resource Addressing](https://developer.hashicorp.com/terraform/language/state/resources)
* [Local Sensitive File Resource](https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/sensitive_file)
