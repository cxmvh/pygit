# cat_file.md

# cat_file — Displaying Git Objects

## Overview

The `cat_file` function is a utility for displaying the contents or metadata of Git objects stored in a repository's object database. It supports various display modes to show raw data, object types, sizes, or a human-readable "pretty" format. This function is integral to the *Object Handling* section of the repository, complementing functions like `read_object` and `read_tree` by providing flexible output tailored to different Git object types (blobs, trees, commits).

Situated within the *Object Handling* documentation, `cat_file` serves as a bridge between low-level object retrieval and user-facing display commands, similar in spirit to the `git cat-file` command in Git itself. It helps users and developers inspect the exact contents or structure of objects identified by their SHA-1 hashes.

---

## Function Documentation

### `cat_file(mode, sha1_prefix)`

**Purpose:**  
Outputs the contents or information of a Git object identified by a SHA-1 prefix, according to the specified display mode.

**Parameters:**  
- `mode` (`str`): Specifies the type of output. Supported modes are:
  - `'commit'`, `'tree'`, `'blob'`: Output raw object data only if object type matches.
  - `'size'`: Prints the size in bytes of the object's content.
  - `'type'`: Prints the type of the object (e.g., `'commit'`, `'tree'`, `'blob'`).
  - `'pretty'`: Prints a formatted representation of the object, which varies by type.
- `sha1_prefix` (`str`): The SHA-1 hash prefix of the object to display. The prefix must be long enough to uniquely identify the object.

**Preconditions:**  
- The SHA-1 prefix must correspond to exactly one object in the repository.
- For modes `'commit'`, `'tree'`, or `'blob'`, the actual object type must match the mode parameter; otherwise, a `ValueError` is raised.

**Operation:**  
1. Uses `read_object(sha1_prefix)` to obtain the object's type and raw data.
2. Depending on `mode`:
   - If `'commit'`, `'tree'`, or `'blob'`: verifies the object type matches `mode` and writes raw data bytes to standard output.
   - If `'size'`: prints the byte length of the object's data.
   - If `'type'`: prints the object's type string.
   - If `'pretty'`:
       - For `'commit'` or `'blob'`, writes raw data to standard output.
       - For `'tree'`, parses the tree entries and prints each entry's mode, type (tree/blob), SHA-1, and path in a formatted line.
   - Raises `ValueError` if `mode` is unrecognized.

**Example Usage:**

```python
# Display raw contents of a blob object
cat_file('blob', 'a1b2c3d')

# Show type of object for given SHA-1 prefix
cat_file('type', 'a1b2c3d')

# Show pretty formatted tree object
cat_file('pretty', 'a1b2c3d')
```

**Sample Output for `cat_file('pretty', tree_sha1)`**:

```
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    README.md
040000 tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904    src
```

---

### Related Functions

To fully understand `cat_file`, familiarity with the following related functions is recommended:

- **`read_object(sha1_prefix)`**  
  Reads a Git object by SHA-1 prefix and returns its type and data bytes.

- **`read_tree(sha1=None, data=None)`**  
  Parses tree object data into a list of `(mode, path, sha1)` entries.

- **`find_object(sha1_prefix)`**  
  Locates the object file path in the `.git/objects` directory corresponding to the given SHA-1 prefix.

---

## Internal Workflow Diagram

```
+-----------------+       +-----------------+       +-----------------------------+
|  cat_file()     |       |  read_object()  |       |  read_tree() (if tree + pretty) |
+-------+---------+       +--------+--------+       +-------------+---------------+
        |                          |                              |
        | 1. Calls read_object()   |                              |
        +------------------------>+                              |
        |                          |                              |
        |                          | 2. Finds object path, reads |
        |                          |    and decompresses raw data |
        |                          |                              |
        |                          +----------------------------->+
        |                          |                              |
        |                          |            3. If tree, parse |
        |                          |               raw tree data  |
        |                          |                              |
        |                          |                              |
        | 4. Depending on mode, prints raw data, size, type or formatted tree |
        |                                                                       
        +------------------------------------------------------------------------+
```

---

# Detailed Function Reference

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object based on a SHA-1 prefix and returns a tuple `(object_type, data_bytes)`.

**Parameters:**  
- `sha1_prefix` (`str`): SHA-1 prefix identifying the object.

**Operation:**  
1. Locates the object file with `find_object(sha1_prefix)`.
2. Reads and decompresses the object file content.
3. Parses the header to extract object type and declared size.
4. Verifies the actual data size matches the header size.
5. Returns the object type and data.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d')
print(f'Object type: {obj_type}')
print(f'Raw data length: {len(data)} bytes')
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a tree object from either a SHA-1 hash or raw data bytes, returning a list of entries.

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 hash of the tree object.
- `data` (`bytes`, optional): Raw data of the tree object.

**Preconditions:**  
One of `sha1` or `data` must be provided.

**Operation:**  
1. If `sha1` is provided, reads the object data using `read_object`.
2. Parses the tree binary format to extract `(mode, path, sha1)` for each entry.
3. Returns the list of entries.

**Example Usage:**

```python
entries = read_tree(sha1='a1b2c3d4e5f6...')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Finds the file path of the Git object corresponding to the provided SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (`str`): SHA-1 prefix to locate.

**Operation:**  
1. Checks the `.git/objects` directory matching the SHA-1 prefix.
2. Ensures exactly one object matches the prefix.
3. Returns the full file path of the object.

**Example Usage:**

```python
object_path = find_object('a1b2c3d')
print(f'Object file located at: {object_path}')
```

---

# Summary

The `cat_file` function provides a versatile interface to inspect Git objects stored in the repository. By leveraging lower-level functions like `read_object` and `read_tree`, it supports multiple output modes, including raw data dump, size, type info, and pretty-printing. This makes it an essential tool for both debugging and understanding the internal structure of Git repositories.

For a comprehensive understanding, users should also refer to the `read_object`, `read_tree`, and `find_object` functions documented in this and related files under the *Object Handling* section.