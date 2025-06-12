# read_object.md

# Reading Git objects by SHA-1 (`read_object` and `find_object`)

---

## Overview

The `read_object.md` document focuses on the mechanisms to read Git objects from the Git object store by their SHA-1 hashes. It primarily covers the functions `read_object` and `find_object`, which are fundamental for retrieving raw Git objects (blobs, trees, commits, etc.) based on their SHA-1 identifiers. This file fits within the broader **Git Object Handling** section of the documentation, which addresses all aspects of Git object management including hashing, reading, encoding, and interpreting objects.

Understanding how to find and read objects by SHA-1 is crucial for operations such as diffing, committing, and inspecting repository contents. These functions underpin higher-level commands like `git diff` and `git cat-file`, enabling the retrieval and decompression of Git objects stored in the `.git/objects` directory.

---

## Function Documentation

### `find_object(sha1_prefix)`

**Purpose:**  
Locate the path to a Git object file in the `.git/objects` directory given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): The beginning portion of the SHA-1 hash of the object. Must be at least 2 characters.

**Returns:**  
- (str) The filesystem path to the Git object file corresponding to the full SHA-1 hash.

**Raises:**  
- `ValueError` if:
  - The prefix is shorter than 2 characters.
  - No object matches the prefix.
  - Multiple objects share the same prefix (ambiguous).

**Operation:**  
1. Validate that the prefix length is at least 2 characters.
2. Split the prefix into the first two characters (representing the directory) and the remainder (representing the file prefix).
3. Construct the path to the directory `.git/objects/<first_two_chars>`.
4. List all files in this directory starting with the given remainder.
5. Check if exactly one object matches:
   - If none, raise an error indicating no object found.
   - If multiple, raise an error indicating ambiguity.
6. Return the full path to the matched object file.

**Example Usage:**

```python
try:
    obj_path = find_object('a1b2c3')
    print("Object path:", obj_path)
except ValueError as e:
    print("Error:", e)
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Read and decompress a Git object by its SHA-1 prefix and return its type and data.

**Parameters:**  
- `sha1_prefix` (str): SHA-1 hash prefix of the object to read.

**Returns:**  
- Tuple `(object_type, data_bytes)`:
  - `object_type` (str): Type of the Git object (e.g., 'blob', 'tree', 'commit').
  - `data_bytes` (bytes): Raw decompressed data content of the object.

**Raises:**  
- `ValueError` if the object cannot be found or if data integrity checks fail.

**Operation:**  
1. Use `find_object(sha1_prefix)` to get the path to the compressed object file.
2. Read the compressed data from the file.
3. Decompress the data using zlib.
4. Parse the decompressed data into:
   - Header: `<type> <size>\0`
   - Content: raw data bytes of the object.
5. Verify that the size in the header matches the length of the decompressed content.
6. Return the object type and its data.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(f"Data (first 100 bytes): {data[:100]}")
```

---

## ASCII Diagram: Git Object Storage and Retrieval

```
.git/
 └── objects/
     ├── a1/
     │    └── b2c3d4e5f6g7h8i9j0k...  # compressed object file
     ├── 3f/
     │    └── 4e5d6c7b8a9f0e1d2c3...
     └── ...

+-----------------------------+
| find_object('a1b2c3')       |
|                             |
|   -> locate directory 'a1'  |
|   -> find file starting with|
|      'b2c3'                 |
|                             |
|   returns:                  |
|   '.git/objects/a1/b2c3d4...'|
+-------------|---------------+
              |
              v
+-----------------------------+
| read_object('a1b2c3d4')     |
|                             |
|   - read compressed file    |
|   - decompress with zlib    |
|   - parse header and data   |
|   - verify size             |
|   - return (type, data)     |
+-----------------------------+
```

---

## Summary

- `find_object` resolves a SHA-1 prefix to the exact object file path in the Git object store.
- `read_object` reads and decompresses the object file, returning its type and content.
- Together, these functions form the core of Git's ability to access objects by their SHA-1 hashes, enabling inspection and manipulation of repository data.

---

## Related Functions and Context

- `read_tree` (for reading tree objects from data)
- `hash_object` (for generating and writing objects)
- `diff` (uses `read_object` to compare blobs)
- `cat_file` (displays object content or info)
- `write_object` (for writing objects to the store)

These functions are part of the **Git Object Handling** section and support commands that require access to Git's internal object database.

---

# End of `read_object.md` documentation file content.