# PortSwigger Web Security Academy - File Path Traversal Lab 1

**Platform:** PortSwigger Web Security Academy | **Category:** Web Security / Path Traversal | **Difficulty:** Apprentice | **Date:** 10/10/2026

## Objective

Retrieve the contents of `/etc/passwd` by exploiting a file path traversal vulnerability in the application's image-loading functionality.

## Approach

The lab contained product images loaded through a URL with a filename parameter. I inspected the image request and identified the filename parameter as a possible entry point for path traversal.

Instead of loading only the intended image, I tested whether modifying the filename could make the server access files outside the image directory.

My troubleshooting process was:

1. **Tried two `../` sequences:** The request failed with a "No such file" error. This suggested that the resulting path did not point to an existing file, but it did not prove that path traversal was blocked.
2. **Tried the `....//` pattern:** I suspected the application might use a naive filter that removes `../` sequences. This pattern can bypass certain filters if they remove a matching substring only once without rechecking the result. However, it did not work in this lab.
3. **Tried URL-encoding the slash as `%2f`:** This also failed. Encoding a character does not guarantee a bypass; the result depends on how the server and any filters decode and process the input.
4. **Tested different traversal depths:** I methodically tried one, three, four, and five `../` sequences rather than repeatedly using the same path. The three-level traversal worked.

The successful filename was:

```text
../../../etc/passwd
```

The server returned the contents of `/etc/passwd`, confirming that I could access a file outside the intended image directory.

## What Was Actually Vulnerable

The application concatenated the user-controlled filename directly onto a fixed directory path, conceptually similar to:

```text
/var/www/images/ + user_input
```

It did not properly validate that the resulting file path remained inside the intended image directory.

The `../` sequence instructs the filesystem to move up one directory level. Using three such sequences allowed the path to move out of the image directory hierarchy and reach the filesystem root before accessing `etc/passwd`.

Conceptually:

```text
Image directory
      |
      | ../
      v
Parent directory
      |
      | ../
      v
Higher-level directory
      |
      | ../
      v
Filesystem root /
      |
      v
/etc/passwd
```

The underlying issue was **insufficient validation of user-controlled file paths**, allowing a request intended for an image to read a different file from the server's filesystem.

## What We Retrieved

The successful request returned `/etc/passwd`, a standard Linux system file containing account information.

A typical entry looks like this:

```text
root:x:0:0:root:/root:/bin/bash
```

| Field | Meaning |
|---|---|
| `root` | Account username |
| `x` | Password information is stored separately, typically in `/etc/shadow` |
| `0` | User ID (UID); UID 0 identifies the root superuser |
| `0` | Primary group ID (GID) |
| `root` | Comment or description field |
| `/root` | Home directory |
| `/bin/bash` | Login shell |

The file also contains entries for service accounts such as `daemon`, `www-data`, and `backup`. These accounts are commonly used to run system services and applications with appropriate privileges.

### Why `/etc/passwd`?

`/etc/passwd` is a standard proof-of-concept target for file path traversal on Linux systems for two main reasons:

- **It is commonly readable:** On typical Linux installations, ordinary users can read this file, making it suitable for demonstrating arbitrary file access without needing root privileges.
- **It provides clear evidence:** Receiving its contents proves that the application accessed a file outside the directory intended for product images.

The `x` in the password field is not the user's actual password. Password hashes are generally stored in the more restricted `/etc/shadow` file.

In a real-world attack, arbitrary file read could expose more sensitive information, such as application configuration files, database credentials, source code, or private keys, depending on file permissions and the server's configuration. `/etc/passwd` was the lab's proof-of-concept target, not the ultimate objective of a real attack.

## What I Actually Learned

- **Traversal depth matters:** Using too few `../` sequences may leave the path inside the wrong directory. Using too many may produce a different invalid path. Both can return the same "No such file" message, so the error alone does not explain what went wrong.
- **A bypass pattern is not universal:** The `....//` technique can work against certain naive filters that remove `../` once, but it is not a universal path traversal bypass. My attempt failed because the application's actual processing behavior did not make that pattern effective.
- **URL encoding is not automatically a bypass:** Encoding `/` as `%2f` only helps under particular decoding and filtering conditions. The order in which a server decodes and validates input matters.
- **Error messages can be ambiguous:** A failed request might mean the file does not exist at the constructed path, the traversal depth is incorrect, or a filter or other control has interfered with the request. I needed to vary one part of the input at a time.
- **A proof of concept should demonstrate impact:** Retrieving `/etc/passwd` showed that the application could read outside its intended directory. Understanding why that file is useful evidence is more important than simply recording the successful payload.

## Key Takeaway

This lab taught me to investigate file path traversal methodically instead of relying on a single payload. By inspecting the image request, testing different traversal patterns, and narrowing down the directory depth, I confirmed that the application trusted user-controlled path input without ensuring that the final file remained inside the intended directory.

The main lesson is that file access should be restricted to an approved directory, preferably using canonicalized paths and proper containment checks or safer file-handling designs.
