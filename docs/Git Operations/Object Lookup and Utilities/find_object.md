# find_object.md

# Locating Object Files by SHA-1 Prefix in the Object Database

---

## Overview

The `find_object.md` document details the `find_object` function, a crucial utility within the `pygit` project for locating Git object files stored in the `.git/objects` directory by their SHA-1 hash prefixes. This function enables other core operations—such as reading objects, displaying object contents, and pushing changes—to efficiently find object data based on partial SHA-1 strings, supporting Git's abbreviated SHA-1 referencing.

Within the broader `pygit` documentation tree, this file is categorized under **Object Handling → Object Utilities**, reflecting its role as an internal helper for object lookup that supports main operations like `cat_file`, `read_object`, and remote push functionality.

---

## Function Documentation

### `find_object(sha1_prefix)`

#### Purpose

Locate the path to a Git object file in the local `.git/objects` storage by a given SHA-1 prefix.

Git objects are stored in directories named after the first two characters of their SHA-1 hash, with filenames matching the remaining 38 characters. This function accepts a SHA-1 prefix string (at least two characters long) and searches the corresponding subdirectory to find a unique object whose filename starts with the remaining characters of the prefix.

If no matching object is found, or if multiple objects match the prefix (making it ambiguous), the function raises a `ValueError`.

#### Parameters

- `sha1_prefix` (`str`): A string representing the beginning of a SHA-1 hash. Must be at least two characters long.

#### Returns

- `str`: The filesystem path to the object file within `.git/objects` corresponding to the prefix.

#### Raises

- `ValueError`: If the prefix is shorter than 2 characters.
- `ValueError`: If no objects match the prefix.
- `ValueError`: If multiple objects match the prefix (ambiguous prefix).

#### Operation Steps

1. **Validate Prefix Length**: Ensure the `sha1_prefix` is at least 2 characters long, since Git object directories are named by the first two characters.

2. **Determine Object Directory**:  
   Extract the first two characters of `sha1_prefix` to identify the subdirectory under `.git/objects/`.  
   Example: For prefix `ab12cd...`, the directory is `.git/objects/ab`.

3. **List Candidate Objects**:  
   List all files in the object directory whose names start with the remaining characters of the prefix (i.e., `sha1_prefix[2:]`).

4. **Handle Results**:  
   - If no matching files found, raise `ValueError` indicating object not found.  
   - If multiple files match, raise `ValueError` indicating ambiguity.  
   - If exactly one file matches, return the full path to that object file.

#### Example Usage

```python
try:
    # Find the object file path for a given SHA-1 prefix
    object_path = find_object('e68a1b2c')
    print(f'Object file located at: {object_path}')
except ValueError as e:
    print(f'Error locating object: {e}')
```

This would look inside `.git/objects/e6/` for a file starting with `8a1b2c` and return the path if exactly one match is found.

---

## Additional Context and Integration

The `find_object` function is fundamental to several higher-level functions within pygit:

- **`read_object(sha1_prefix)`**: Uses `find_object` to locate and read the compressed object file, then decompress and parse its contents.  
- **`cat_file(mode, sha1_prefix)`**: Relies on `read_object` which in turn depends on `find_object` to retrieve object data by prefix.  
- **`push(git_url, username, password)`**: Uses object reading functions that utilize `find_object` to package objects for transfer.

---

## ASCII Diagram: Git Object Storage Structure and Lookup

This diagram illustrates the typical layout of the `.git/objects` directory and how `find_object` locates an object file by SHA-1 prefix:

```
.git/
 └── objects/
      ├── e6/
      │    ├── 8a1b2c3d4e5f6g7h8i9j0k...   <-- Object files named by SHA-1 suffix
      │    ├── f123456789abcdef012345...
      │    └── ...
      ├── ab/
      │    ├── 12cd34ef56ab78cd90ef12...
      │    └── ...
      └── ...
```

- `sha1_prefix`: "e68a1b2c"  
- Object directory: `.git/objects/e6/` (first two chars)  
- Search filenames starting with `"8a1b2c"` (remaining chars)  

If exactly one match is found, its full path is returned.

---

## Source Code Reference

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

## Summary

The `find_object` function is a simple yet critical utility in the pygit implementation that enables efficient and unambiguous retrieval of object files by their SHA-1 prefixes. It enforces prefix length constraints and handles error cases gracefully, ensuring reliable object lookup supporting all Git object operations in the system.