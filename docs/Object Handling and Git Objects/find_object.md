# find_object.md

# Locating Git Objects by SHA-1 Prefix

---

## Overview

This document explains the functionality provided by the `find_object` module, which is responsible for locating Git objects stored in the `.git/objects` directory by their SHA-1 prefix. In Git, every object (blob, tree, commit, tag) is identified by a unique SHA-1 hash. Often, tools and commands accept abbreviated SHA-1 prefixes for convenience, and the system must resolve these prefixes to the full object hash.

`find_object` plays a crucial role in the object handling subsystem of this Git implementation, enabling commands and utilities to locate Git objects efficiently and accurately even when only partial SHA-1 hashes are provided. This functionality supports other modules such as `read_object`, `cat_file`, and diff-related commands that rely on resolving SHA-1 prefixes to actual objects.

---

## Function Documentation

### find_object(sha1_prefix)

**Purpose:**

Locate a Git object in the `.git/objects` directory based on a given SHA-1 hash prefix and return the full path to the object file.

**Parameters:**

- `sha1_prefix` (str): A string representing the initial characters of a SHA-1 hash. Must be at least 2 characters to avoid ambiguity.

**Returns:**

- `str`: The full file system path to the object file matching the prefix.

**Raises:**

- `ValueError`: If the prefix is less than 2 characters.
- `ValueError`: If no object with the given prefix is found.
- `ValueError`: If multiple objects match the prefix (ambiguous prefix).

---

**Detailed Explanation:**

1. **Prefix Length Check:**  
   The function requires at least the first two characters of the SHA-1 prefix because Git objects are stored under directories named by the first two characters of their SHA-1.

2. **Locate Object Directory:**  
   The first two characters form the directory name under `.git/objects/`. For example, a prefix `ab12` points to `.git/objects/ab/`.

3. **Match Objects in Directory:**  
   Inside this directory, the function lists all files and filters those starting with the remaining characters of the prefix (characters after the first two).

4. **Validation of Matches:**  
   - If no files match, the function raises an error indicating the object was not found.
   - If multiple files match, the function raises an error indicating multiple objects match the prefix.
   - If exactly one file matches, it returns the full path to that object.

---

**Example Usage:**

```python
try:
    object_path = find_object('f1a3c4')
    print(f"Object found at: {object_path}")
except ValueError as e:
    print(f"Error: {e}")
```

If `.git/objects/f1/a3c4...` exists and is unique, `object_path` will be the full path to that file.

---

**ASCII Diagram:**

```
.git/
 └── objects/
      ├── f1/
      │    ├── a3c4...  <-- matched file (full SHA-1: f1a3c4...)
      │    └── b5d6...
      ├── ab/
      │    ├── 12ef...
      │    └── 34cd...
      └── ...
```

Given `sha1_prefix='f1a3'`, the function looks inside `.git/objects/f1/` and finds files starting with `a3`.

---

## Related Functions in Object Handling

- **read_object(sha1_prefix)**: Uses `find_object` to locate and then read and decompress the Git object data.
- **cat_file(mode, sha1_prefix)**: Displays contents or metadata of the object found by `find_object`.
- **hash_object(data, obj_type, write=True)**: Generates the SHA-1 for new object data and stores it under `.git/objects`.
  
All these functions depend on accurate and efficient resolution of SHA-1 prefixes, which is the responsibility of `find_object`.

---

## Summary

The `find_object` function is a fundamental utility in accessing Git object data by partially specified SHA-1 hashes. It enforces strict prefix length requirements and ensures unambiguous resolution to a single object, supporting robust command and internal Git operations.

---

# Code Snippet Reference

```python
def find_object(sha1_prefix):
    """Find object with given SHA-1 prefix and return path to object in object
    store, or raise ValueError if there are no objects or multiple objects
    with this prefix.
    """
    if len(sha1_prefix) < 2:
        raise ValueError('hash prefix must be 2 or more characters')
    obj_dir = os.path.join('.git', 'objects', sha1_prefix[:2])
    rest = sha1_prefix[2:]
    objects = [name for name in os.listdir(obj_dir) if name.startswith(rest)]
    if not objects:
        raise ValueError('object {!r} not found'.format(sha1_prefix))
    if len(objects) >= 2:
        raise ValueError('multiple objects ({}) with prefix {!r}'.format(
                len(objects), sha1_prefix))
    return os.path.join(obj_dir, objects[0])
```

---

# End of find_object.md documentation.