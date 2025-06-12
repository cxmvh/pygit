# cat_file.md

# Documentation for the `cat_file` Function and Related Object Reading Utilities

---

## Overview

This document provides detailed technical documentation for the `cat_file` function and its associated utilities within the `pygit` project. Located in the **Git Objects and Packfiles** section of the repository, these functions form the core mechanisms for reading, interpreting, and displaying Git objects such as commits, trees, blobs, and their metadata.

The `cat_file` function mimics the behavior of the Git command `git cat-file`, allowing users or internal processes to output raw object data, object size, type, or a prettified representation. Supporting functions include object retrieval, decompression, and parsing utilities (`read_object`, `read_tree`, `find_object`), which collectively enable efficient access and manipulation of Git’s internal storage format.

This documentation is essential for developers extending or maintaining the object handling and inspection features of the `pygit` repository, bridging low-level object data with user-facing commands.

---

## Function Documentation

---

### `cat_file(mode, sha1_prefix)`

**Purpose:**  
Outputs the contents or metadata of a Git object identified by a SHA-1 prefix, according to the specified mode.

**Parameters:**  
- `mode` (`str`): Determines the output format and content. Possible values:
  - `'commit'`, `'tree'`, `'blob'`: Output raw data bytes for the specified object type.
  - `'size'`: Print the size (in bytes) of the object data.
  - `'type'`: Print the type of the object (`commit`, `tree`, `blob`, etc.).
  - `'pretty'`: Print a prettified version of the object according to its type.
- `sha1_prefix` (`str`): A prefix string (usually the first few hex characters) of the SHA-1 hash identifying the object.

**Operation Steps:**  
1. Use `read_object` to retrieve the full object type and data corresponding to `sha1_prefix`.
2. Based on the `mode`:
   - For `'commit'`, `'tree'`, `'blob'`:
     - Validate the object type matches the mode.
     - Write raw bytes of the object data directly to standard output.
   - For `'size'`:
     - Print the length of the object’s data in bytes.
   - For `'type'`:
     - Print the object type string.
   - For `'pretty'`:
     - If object is `commit` or `blob`, write raw data bytes.
     - If object is `tree`, parse the tree entries using `read_tree` and print each entry in a formatted style showing mode, type, SHA-1, and path.
     - Raise an error if an unknown object type is encountered.
3. Raise a `ValueError` if the `mode` is not recognized.

**Example Usage:**

```python
# Print raw contents of a blob object by prefix
cat_file('blob', 'a1b2c3d4')

# Print size of an object
cat_file('size', 'a1b2c3d4')

# Pretty print a tree object
cat_file('pretty', 'a1b2c3d4')
```

**ASCII Diagram:**

```
+-----------------+          +-------------------+
|  cat_file call  |  ---->   | read_object(sha1) |
+-----------------+          +-------------------+
         |                            |
         |                            v
         |                    (obj_type, data)
         |                            |
         |                    +---------------+
         |                    | Output based  |
         |                    | on mode param |
         |                    +---------------+
         |                            |
         +----------------------------+
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Retrieve and decompress a Git object’s data by its SHA-1 prefix and return its type and raw data bytes.

**Parameters:**  
- `sha1_prefix` (`str`): Prefix of the SHA-1 hash of the object.

**Operation Steps:**  
1. Use `find_object` to locate the file path of the object in the Git object store.
2. Read the compressed object file’s bytes via `read_file`.
3. Decompress the data using `zlib.decompress`.
4. Parse the object header separated by a null byte (`\x00`), extracting the object type and expected size.
5. Extract the object data bytes following the header.
6. Validate that the decompressed data size matches the expected size.
7. Return a tuple `(object_type, data_bytes)`.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d4')
print(f'Object type: {obj_type}')
print(f'Data length: {len(data)} bytes')
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Locate the file path of a Git object in the `.git/objects/` directory given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (`str`): SHA-1 hash prefix (at least 2 characters).

**Operation Steps:**  
1. Verify the prefix length is at least 2.
2. Construct the object directory path using the first two characters of the prefix.
3. List files in the directory and filter those matching the remainder of the SHA-1 prefix.
4. If no matches found, raise an error.
5. If multiple matches found, raise an error indicating ambiguity.
6. Return the full path to the object file.

**Example Usage:**

```python
path = find_object('a1b2c3')
print(f'Object located at: {path}')
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parse a Git tree object to extract its entries as `(mode, path, sha1)` tuples.

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 hash of the tree object. If provided, object data will be read.
- `data` (`bytes`, optional): Raw data of the tree object. Used if `sha1` is not provided.

**Operation Steps:**  
1. If `sha1` is given, call `read_object` to get the tree data.
2. If neither `sha1` nor `data` is provided, raise a `TypeError`.
3. Parse the tree entries from the binary data:
   - Each entry consists of mode and path string, a null byte, then the 20-byte SHA-1 hash.
4. Collect entries until no more null bytes are found.
5. Return a list of tuples: `(mode, path, sha1_hex)`.

**Example Usage:**

```python
entries = read_tree(sha1='e68f7a...')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### `read_file(path)`

**Purpose:**  
Read the contents of a file as bytes.

**Parameters:**  
- `path` (`str`): Filesystem path to the file.

**Operation Steps:**  
1. Open the file in binary read mode.
2. Read and return all bytes from the file.

**Example Usage:**

```python
data = read_file('.git/objects/a1/b2c3...')
print(f'Read {len(data)} bytes')
```

---

## Additional Related Utilities

While the primary focus is `cat_file` and its direct helpers, understanding the overall object reading workflow benefits from familiarity with the following related functions:

- **`diff()`**: Compares file contents in the working copy against the index, showing line-by-line diffs.
- **`find_commit_objects(commit_sha1)`**: Recursively locates all objects linked to a commit.
- **`find_tree_objects(tree_sha1)`**: Recursively locates all objects in a tree.
- **`hash_object(data, obj_type, write=True)`**: Hashes and optionally writes object data to the store.
- **`write_tree()`**: Writes a tree object from current index entries.
- **`read_index()` / `write_index(entries)`**: Manage the Git index file.
- **`get_status()` / `status()`**: Determine and display working copy changes.

---

## Summary Diagram of Object Reading Flow

```
cat_file(mode, sha1_prefix)
        |
        v
read_object(sha1_prefix)
        |
    find_object(sha1_prefix)
        |
    read_file(object_path)
        |
    decompress data -> parse header & content
        |
    return (obj_type, data)
        |
+-------------------------------------------+
| Depending on mode:                         |
| - raw output (commit/tree/blob)           |
| - print size                              |
| - print type                              |
| - pretty print (commit/blob/tree parsed) |
+-------------------------------------------+
```

---

## Conclusion

The `cat_file` function and its associated utilities provide a critical interface for accessing and displaying Git objects stored in the `.git/objects` directory. Their correct implementation ensures that other Git operations like diffing, committing, and pushing can reliably retrieve and interpret repository data.

This module serves as a foundation for understanding Git’s internal object model and can be extended for advanced inspection, debugging, or custom tooling around Git repositories.