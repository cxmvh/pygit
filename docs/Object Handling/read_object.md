# read_object.md

# Overview

The `read_object` function is a core utility in the pygit project that reads and parses Git objects identified by their SHA-1 hash prefixes. It is part of the **Object Handling** section of the documentation, which focuses on operations related to Git objects such as commits, trees, blobs, and their storage and retrieval. This function plays a fundamental role in retrieving the raw data and type of a Git object, enabling higher-level commands and utilities like `cat_file`, `status`, and `push` to operate on Git objects correctly.

Located within the broader context of the pygit repository, the `read_object` function interacts closely with functions that locate objects on the filesystem (`find_object`), decompress and interpret their contents, and validate their integrity. Understanding `read_object` is essential for comprehending how pygit accesses and manipulates the Git object database.

---

# Function Documentation

## `read_object(sha1_prefix)`

Reads a Git object using a SHA-1 hash prefix, returning its type and raw data bytes. Raises an error if the object cannot be found or if multiple objects match the prefix.

### Purpose

Git objects (commits, trees, blobs) are stored under `.git/objects` using their SHA-1 hashes split into directories and files. This function accepts a partial or full SHA-1 hex string (`sha1_prefix`), finds the corresponding object file, decompresses its zlib-compressed content, parses its header to extract the object type and size, and returns the object type and data bytes.

### Parameters

- `sha1_prefix` (str): The prefix (partial or full) of the SHA-1 hash identifying the Git object to read.

### Returns

- Tuple `(obj_type, data)`:
  - `obj_type` (str): The type of the Git object (`"commit"`, `"tree"`, `"blob"`, etc.).
  - `data` (bytes): The raw data content of the object.

### Raises

- `ValueError`: If the object is not found or if multiple objects match the given prefix.
- `AssertionError`: If the declared size in the object header does not match the actual data length.

### Operation Details

1. Uses `find_object(sha1_prefix)` to locate the filesystem path to the compressed object file.
2. Reads the compressed file content as bytes.
3. Decompresses the data using `zlib.decompress`.
4. Splits the decompressed data into a header and content at the null byte (`\x00`).
5. Parses the header (e.g., `"blob 14"`) to get the object type and size.
6. Extracts the data portion after the null byte.
7. Verifies that the size declared in the header matches the length of the data.
8. Returns the object type and data bytes.

### Code Example

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

### Usage Example

Suppose you want to read the object with SHA-1 prefix `"a1b2c3"`:

```python
obj_type, data = read_object("a1b2c3")
print(f"Object type: {obj_type}")
print(f"Data (first 100 bytes): {data[:100]!r}")
```

This would print the type of the object (e.g., `"blob"`) and the first 100 bytes of its content.

---

# Related Functions Overview

To fully utilize `read_object`, understanding related functions is helpful:

- `find_object(sha1_prefix)`: Locates the object file path on disk for the given SHA-1 prefix.
- `read_file(path)`: Reads a file's contents as bytes.
- `cat_file(mode, sha1_prefix)`: Uses `read_object` to display information or contents of an object.
- `read_tree(sha1=None, data=None)`: Parses tree objects to list their entries.
- `hash_object(data, obj_type, write=True)`: Hashes and optionally writes objects to the object store.

---

# ASCII Diagram: Git Object File Storage Layout

```
.git/
 |
 +-- objects/
       |
       +-- aa/          # First two characters of SHA-1 hash
       |    +-- bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb  # Remaining 38 chars of SHA-1 hash
       +-- ...
```

The `read_object` function uses the SHA-1 prefix to navigate this directory structure to locate the compressed object file.

---

# Summary

The `read_object` function is a critical building block for accessing Git objects in pygit. It ensures that any object identified by a SHA-1 prefix can be read, decompressed, and parsed into a usable form, enabling other Git operations such as displaying object contents, showing diffs, and pushing commits. Its robust error checking and integration with other utility functions make it essential for maintaining Git's object integrity and accessibility within pygit.