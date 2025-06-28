# read_object.md

# Reading and Parsing Git Objects by SHA-1 Prefix

## Overview

This document describes the `read_object` function, a core utility in the pygit project for reading and parsing Git objects stored in the local Git object database. Git objects (commits, trees, blobs, and tags) are identified by their SHA-1 hash. This function supports reading objects given a SHA-1 prefix, resolving it to the full object, decompressing the stored data, and returning the parsed object type and raw data bytes.

The `read_object` function is a fundamental building block used by many higher-level commands and functions, including `cat_file` (for displaying objects), `status` (to compare working directory files with stored blobs), and the push operation (to collect objects for transfer). It fits within the "Object Handling" section of the pygit documentation, which details operations related to Git objects, their hashing, storage, and retrieval.

---

## Function: `read_object`

```python
def read_object(sha1_prefix):
    """Read object with given SHA-1 prefix and return tuple of
    (object_type, data_bytes), or raise ValueError if not found.
    """
    path = find_object(sha1_prefix)
    full_data = zlib.decompress(read_file(path))
    nul_index = full_data.index(b'\x00')
    header = full_data[:nul_index]
    obj_type, size_str = header.decode().split()
    size = int(size_str)
    data = full_data[nul_index + 1:]
    assert size == len(data), 'expected size {}, got {} bytes'.format(
            size, len(data))
    return (obj_type, data)
```

### Purpose

`read_object` retrieves a Git object stored in the `.git/objects` directory, identified by a SHA-1 prefix. It decompresses the object data (which is stored in zlib compressed format), parses the object header to identify the object type (`commit`, `tree`, `blob`, or `tag`) and the size of the object content, and then returns both the object type and the raw data bytes.

If the SHA-1 prefix is ambiguous or the object cannot be found, it raises a `ValueError`.

### Parameters

- `sha1_prefix` (str): The hexadecimal SHA-1 prefix string identifying the object to read. This prefix must be at least 2 characters long and uniquely identify a single object within the object database.

### Returns

- Tuple `(obj_type, data)`:
  - `obj_type` (str): The type of the Git object (`commit`, `tree`, `blob`, or `tag`).
  - `data` (bytes): The raw content bytes of the object (excluding the Git object header).

### Operation Details

1. **Find Object Path:**
   - Uses `find_object(sha1_prefix)` to locate the file path of the object in the `.git/objects` directory.
   - This function verifies the uniqueness and existence of the object by matching the prefix.

2. **Read and Decompress Data:**
   - Reads the compressed object data from the file via `read_file(path)`.
   - Decompresses the data using `zlib.decompress`.

3. **Parse Header and Content:**
   - Locates the first null byte (`\x00`) which separates the object header from the content.
   - The header contains the object type and size string (e.g., `"blob 14"`).
   - Extracts the object type and object size.
   - Extracts the content bytes following the null byte.

4. **Validate Size:**
   - Asserts the declared size matches the actual data length for integrity.

5. **Return:**
   - Returns a tuple of the object type string and the raw data bytes.

---

## Usage Example

Suppose you want to read and print the contents of a blob object with prefix `"e68f"`:

```python
obj_type, data = read_object("e68f")
if obj_type == "blob":
    print(data.decode())  # Assuming blob stores text content
else:
    print(f"Object type is {obj_type}, not a blob.")
```

This will:

- Locate the object file that matches the SHA-1 prefix "e68f".
- Decompress and parse the object.
- Print the content of the blob as a string.

---

## Related Functions

- **`find_object(sha1_prefix)`**: Finds the full object path corresponding to the given SHA-1 prefix, ensuring uniqueness.
- **`read_file(path)`**: Reads raw bytes from a file at the specified path.
- **`cat_file(mode, sha1_prefix)`**: Uses `read_object` to display the content or information about Git objects in various modes.
- **`read_tree(sha1=None, data=None)`**: Parses tree objects into entries, often called after reading a tree object.
- **`hash_object(data, obj_type, write=True)`**: Creates and stores a new Git object.
- **`find_commit_objects(commit_sha1)`**: Recursively finds all objects referenced by a commit.

---

## ASCII Diagram: Object Storage and Reading Flow

```
.git/
└── objects/
    ├── <first two chars of SHA-1>/
    │   ├── <remaining 38 chars of SHA-1>  # compressed object file
    │   └── ...
    └── ...

read_object("abc123..."):

+-----------------------------+
| find_object("abc123")        |--+
+-----------------------------+  |
                                 | returns file path
+-----------------------------+  |
| read_file(path)              |  | reads compressed bytes
+-----------------------------+  |
                                 | compressed bytes
+-----------------------------+  |
| zlib.decompress(data)        |  |
+-----------------------------+  |
                                 | full_data = "<type> <size>\\0<data>"
+-----------------------------+  |
| parse header and data        |<-+
+-----------------------------+

Returns: (obj_type, data_bytes)
```

---

## Notes

- The SHA-1 prefix must be at least 2 characters to narrow down the object directory and uniquely identify one object.
- The function raises `ValueError` if the prefix is ambiguous or the object does not exist.
- The object data returned excludes Git's internal header but includes the raw content bytes, which vary by object type.
- This read operation is critical for many Git commands that inspect or manipulate repository data.

---

End of `read_object.md` documentation.