# find_object.md

---

## Overview

This document describes the `find_object` function used within the `pygit` project to locate Git object files stored in the `.git/objects` directory by their SHA-1 prefix. This utility is critical for accessing Git objects (commits, trees, blobs) efficiently and reliably by verifying object existence and uniqueness based on partial SHA-1 hashes.

Within the broader `pygit` documentation tree, this function resides under the **Object Handling / Object Utilities** section, supporting core operations like reading objects (`read_object`), displaying objects (`cat_file`), and managing object storage. Its accurate functionality is essential for higher-level Git operations such as commits, diffs, and pushes.

---

## Function: `find_object`

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

### Purpose

The `find_object` function locates the full path to a Git object file within the `.git/objects` directory given a SHA-1 prefix string. Git object files are stored in a two-level directory structure based on the SHA-1 hash:

- The first two characters of the SHA-1 form the directory name inside `.git/objects`.
- The remaining 38 characters form the file name inside that directory.

This function allows users to specify only a prefix of the SHA-1, as long as it is unambiguous and at least two characters long.

### Parameters

- `sha1_prefix` (str): A hexadecimal string representing the prefix of a SHA-1 hash. Must be at least 2 characters long.

### Returns

- `str`: The full filesystem path to the object file corresponding to the SHA-1 prefix.

### Raises

- `ValueError`: If the prefix is less than 2 characters.
- `ValueError`: If no object matches the prefix.
- `ValueError`: If multiple objects match the prefix (ambiguous prefix).

### How It Works

1. **Validate prefix length**: Ensures the prefix is at least 2 characters, because the first two characters are used as a directory name.

2. **Determine object directory**: Uses the first two characters of the prefix to identify the subdirectory inside `.git/objects`.

3. **List candidate files**: Lists all files in the object directory that start with the remaining characters of the prefix.

4. **Check matches**:
   - If no files match, raises an error that the object was not found.
   - If multiple files match, raises an error that the prefix is ambiguous.

5. **Return unique object path**: Returns the full path to the uniquely identified object file.

### Example Usage

```python
try:
    # Locate object file with prefix 'a1b2c3'
    object_path = find_object('a1b2c3')
    print('Object found at:', object_path)
except ValueError as e:
    print('Error locating object:', e)
```

If the `.git/objects/a1` directory contains a file named `b2c3d4e5f6...`, this function will return the path `.git/objects/a1/b2c3d4e5f6...`. If no such file exists or multiple files match the prefix, an error is raised.

---

## ASCII Diagram: Git Object Storage Structure

```
.git/
└── objects/
    ├── aa/
    │   ├── b12345...  # Object file with SHA-1 hash starting 'aab12345...'
    │   └── c67890...
    ├── b1/
    │   └── 234567...
    └── ...
```

When searching for prefix `aab1`, the function:

- Looks in `.git/objects/aa/`
- Finds files starting with `b1`
- If exactly one file matches (e.g., `b12345...`), returns the full path `.git/objects/aa/b12345...`

---

## Related Functions

- `read_object(sha1_prefix)`: Uses `find_object` to locate and read Git objects by SHA-1 prefix.
- `cat_file(mode, sha1_prefix)`: Displays object contents after locating it with `find_object`.
- `hash_object(data, obj_type)`: Creates and stores objects that `find_object` can later retrieve.

---

## Summary

`find_object` is a fundamental utility for locating Git objects by partial SHA-1 hashes within the Git object store. It ensures prefixes are unambiguous and maps them to actual object file paths, enabling efficient access to Git's internal data structures.