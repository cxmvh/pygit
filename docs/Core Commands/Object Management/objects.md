# objects.md

# Hashing Objects, Writing Files, Reading Objects, and Creating Tree Objects

---

## Overview

This document provides comprehensive technical reference for handling Git objects within the `pygit` project. It covers critical operations such as hashing objects, writing data to the object store, reading objects by their SHA-1 hashes, and creating tree objects from the current index. These functions form the backbone of Git's object storage mechanism and underpin key workflows like committing changes, managing file contents, and building repository history.

Within the broader documentation tree, this file resides inside the **Object Handling** section, which focuses on reading, writing, hashing, and managing Git objects such as blobs, trees, and commits. It directly supports higher-level Git commands and flows, including `pygit.commit`, where these object functions are heavily utilized to record repository state.

---

## Function Documentation

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of a Git object constructed from given raw data and its type (e.g., `'blob'`, `'tree'`, `'commit'`). Optionally writes the compressed object data to the Git object store under `.git/objects/`.

**Parameters:**  
- `data` (`bytes`): The raw content of the object.  
- `obj_type` (`str`): The type of object, such as `'blob'`, `'tree'`, or `'commit'`.  
- `write` (`bool`, default `True`): Whether to store the object in the Git object database.

**Operation:**  
1. Create the object header in the format: `"<obj_type> <len(data)>"` encoded as bytes.  
2. Concatenate the header, a null byte (`\x00`), and the data to form the full object bytes.  
3. Compute the SHA-1 hash of these full object bytes.  
4. If `write` is `True`, compress the full data using zlib and write it to the path `.git/objects/xx/yyyy...` where `xx` and `yyyy...` are derived from the SHA-1 digest.  
5. Return the SHA-1 hash as a hex string.

**Example Usage:**

```python
blob_data = b"Hello, Git!"
sha1 = hash_object(blob_data, "blob")
print(f"Stored blob object with SHA-1: {sha1}")
```

**ASCII Diagram:**

```
+----------------+      +-----------+      +--------------------------+
| Raw Data bytes | ---> | Header    | ---> | Full Object Bytes        |
| (e.g. "Hello") |      | "<type> N" |      | = header + \x00 + data   |
+----------------+      +-----------+      +--------------------------+
                                                |
                                                v
                                      +---------------------+
                                      | SHA-1 Hash (hex)    |
                                      +---------------------+
                                                |
                                     Write compressed to:
               .git/objects/xx/yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

---

### `write_file(path, data)`

**Purpose:**  
Writes bytes data to a file at the specified path.

**Parameters:**  
- `path` (`str`): The filesystem path to write to.  
- `data` (`bytes`): The data to write.

**Operation:**  
- Opens the file in binary write mode and writes the provided bytes.

**Example Usage:**

```python
write_file(".git/refs/heads/master", b"abc123def456\n")
```

---

### `read_file(path)`

**Purpose:**  
Reads the contents of a file at a given path and returns it as bytes.

**Parameters:**  
- `path` (`str`): The file path to read.

**Returns:**  
- `bytes`: The content of the file.

**Operation:**  
- Opens the file in binary read mode and reads all bytes.

**Example Usage:**

```python
data = read_file(".git/HEAD")
print(data.decode())
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object given its SHA-1 prefix (partial or full hash), decompresses it, parses its header, and returns its type and raw data.

**Parameters:**  
- `sha1_prefix` (`str`): The SHA-1 hash prefix identifying the object.

**Returns:**  
- Tuple of `(obj_type: str, data: bytes)`.

**Preconditions:**  
- The object must exist in the object store, identified uniquely by `sha1_prefix`.

**Operation:**  
1. Locate the object file path using the prefix (via `find_object`).  
2. Read and decompress the stored zlib data.  
3. Split the header (`<type> <size>`) and raw content at the null byte.  
4. Verify the content length matches the size specified in the header.  
5. Return the parsed object type and data bytes.

**Example Usage:**

```python
obj_type, data = read_object("a1b2c3")
print(f"Object type: {obj_type}")
print(f"Content:\n{data.decode()}")
```

---

### `write_tree()`

**Purpose:**  
Creates a Git tree object from the current index entries, representing a snapshot of the directory contents at the time of commit.

**Returns:**  
- SHA-1 hex string of the newly created tree object.

**Operation:**  
1. Read all entries from the Git index.  
2. For each entry, format a tree entry as:  
   `"<mode> <path>\x00<sha1_bytes>"`  
3. Concatenate all tree entries into a single byte string.  
4. Hash and write the tree object using `hash_object` with type `"tree"`.  
5. Return the SHA-1 hash of this tree object.

**Preconditions:**  
- The index must exist and contain entries representing files staged for commit.  
- Currently only supports entries at the top directory level (no nested directories).

**Example Usage:**

```python
tree_sha1 = write_tree()
print(f"Tree object SHA-1: {tree_sha1}")
```

**ASCII Diagram:**

```
Index Entries:
+----------------------------+
| mode | path       | sha1    |
+----------------------------+
       |
       v
For each entry:
"mode path\0sha1"
       |
       v
Concatenate all entries
       |
       v
Hash as tree object -> SHA-1
       |
       v
Write to .git/objects/...
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieves the current commit SHA-1 hash pointed to by the local `master` branch.

**Returns:**  
- SHA-1 hex string of the current `master` commit, or `None` if no commit exists yet.

**Operation:**  
- Reads the contents of `.git/refs/heads/master` file.  
- Returns the stripped commit hash string.

**Example Usage:**

```python
master_sha1 = get_local_master_hash()
if master_sha1:
    print(f"Local master commit: {master_sha1}")
else:
    print("No commits on master yet.")
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds all Git object hashes (trees and blobs) referenced by a given tree object, including the tree itself.

**Parameters:**  
- `tree_sha1` (`str`): SHA-1 hash of the tree object to explore.

**Returns:**  
- `set` of SHA-1 strings representing all objects reachable from this tree.

**Operation:**  
1. Initialize a set with the starting tree SHA-1.  
2. Read tree entries (mode, path, sha1) from the tree object.  
3. For each entry:  
   - If it is a directory (mode indicates directory), recurse into it and add all returned objects.  
   - Otherwise, add the blob SHA-1.  
4. Return the accumulated set.

**Example Usage:**

```python
all_objects = find_tree_objects(tree_sha1)
print(f"Objects in tree: {all_objects}")
```

---

### `commit(message, author=None)`

**Purpose:**  
Creates a new commit object representing the current state of the index, recording the commit message, author, timestamp, and parent commit.

**Parameters:**  
- `message` (`str`): Commit message to record.  
- `author` (`str`, optional): Author string in the format `"Name <email>"`. Defaults to environment variables.

**Returns:**  
- SHA-1 hex string of the created commit object.

**Operation:**  
1. Call `write_tree()` to write the current index as a tree object.  
2. Retrieve the current master commit hash as parent (if any).  
3. Compose commit metadata lines: tree, parent, author, committer, timestamps, and message.  
4. Encode and hash the commit object using `hash_object` with type `"commit"`.  
5. Update `.git/refs/heads/master` with the new commit SHA-1.  
6. Print confirmation and return the SHA-1.

**Example Usage:**

```python
commit_sha1 = commit("Initial commit")
print(f"Committed with SHA-1: {commit_sha1}")
```

**ASCII Diagram:**

```
+-----------------+      +-----------------+      +-----------------+
| Current Index   | ---> | write_tree()     | ---> | Tree Object SHA-1|
+-----------------+      +-----------------+      +-----------------+
                                                      |
                                                      v
                                             +-----------------+
                                             | Compose commit   |
                                             | metadata lines   |
                                             +-----------------+
                                                      |
                                                      v
                                             +-----------------+
                                             | hash_object(...) |
                                             +-----------------+
                                                      |
                                                      v
                                             +-----------------+
                                             | Update refs/heads/master |
                                             +-----------------+
```

---

## Additional Notes

- The functions `write_file` and `read_file` are utility functions used throughout object handling for interacting with the filesystem.
- The object storage layout follows the standard Git pattern `.git/objects/xx/yyyy...` derived from the first two and remaining 38 characters of the SHA-1 hash.
- Tree objects link blobs and other trees, representing directory structure snapshots.
- Commit objects reference trees and parent commits, storing author and committer metadata, enabling Git history traversal.
- The `hash_object` function is fundamental for ensuring data integrity and efficient storage of Git objects.
- Functions in this file are often invoked by higher-level commands like `pygit.commit` and support repository state management.

---

This documentation serves as the core reference for understanding how `pygit` manages Git objects at a low level, enabling developers to extend or troubleshoot object storage and retrieval operations effectively.