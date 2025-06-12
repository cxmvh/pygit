# object_handling.md

# Object Handling in Git: Reading, Writing, Hashing, and Pack Encoding

---

## Overview

This document covers the core functionalities related to Git object handling within the repository. It focuses on reading and writing Git objects (blobs, trees, commits), hashing data to generate object IDs (SHA-1 hashes), locating objects by their SHA-1 prefix, and encoding objects for storage in Git packfiles. These operations are fundamental to Git's data integrity, content-addressable storage, and efficient object transmission.

Situated within the "Object Handling and Git Objects" section of the documentation tree, this file serves as a technical reference for how objects are manipulated at a low level, bridging user-facing commands and the internal Git storage mechanisms. It is closely related to index management, tree and commit management, and packfile encoding.

---

## Detailed Function Descriptions

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of a given data blob, prepended with a Git object header specifying its type and size. If `write` is `True`, the object is compressed and written into the Git object store under `.git/objects/`.

**Parameters:**  
- `data` (`bytes`): Raw bytes of the object's content.  
- `obj_type` (`str`): Type of the object, e.g., `'blob'`, `'tree'`, `'commit'`.  
- `write` (`bool`): Whether to write the compressed object to the object store.

**Operation:**  
1. Create a header string: `"<obj_type> <len(data)>"` encoded as bytes.  
2. Concatenate header, a null byte (`\x00`), and the data to form `full_data`.  
3. Compute SHA-1 hash of `full_data`.  
4. If `write` is `True`, store the compressed `full_data` in `.git/objects/<first two hex chars>/<remaining 38 chars>`. The directory is created if it does not exist.  
5. Return the SHA-1 hash as a hex string.

**Example Usage:**

```python
data = b'Hello Git!'
sha1 = hash_object(data, 'blob')
print(f'Object stored with hash: {sha1}')
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Retrieve and decompress a Git object given its SHA-1 hash prefix, returning its type and raw data.

**Parameters:**  
- `sha1_prefix` (`str`): Hex prefix of the SHA-1 hash identifying the object.

**Operation:**  
1. Use `find_object` to locate the object file path from the prefix.  
2. Read and decompress the object data.  
3. Parse the header (e.g., `'blob 12'`) to extract object type and data size.  
4. Return a tuple `(obj_type, data_bytes)`, verifying the size matches.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d4')
print(f'Object type: {obj_type}, Data length: {len(data)}')
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Locate the full path of the object file in `.git/objects/` directory given a SHA-1 prefix. Ensures the prefix uniquely identifies one object.

**Parameters:**  
- `sha1_prefix` (`str`): At least two hex characters of the SHA-1 hash.

**Operation:**  
1. Check that prefix length ≥ 2.  
2. List files in `.git/objects/<first two chars>`.  
3. Filter filenames starting with the remaining prefix characters.  
4. Raise exceptions if no or multiple matches found.  
5. Return full path to the matching object file.

**Example Usage:**

```python
path = find_object('a1b2c3')
print(f'Object path: {path}')
```

---

### `write_file(path, data)`

**Purpose:**  
Write raw data bytes to the file at the specified path, creating directories as needed.

**Parameters:**  
- `path` (`str`): File path to write to.  
- `data` (`bytes`): Data to write.

**Operation:**  
Open the file in binary write mode and write the data in full.

**Example Usage:**

```python
write_file('.git/objects/ab/cdef1234', b'compressed data bytes')
```

---

### `read_file(path)`

**Purpose:**  
Read the entire contents of a file as bytes.

**Parameters:**  
- `path` (`str`): Path to the file.

**Operation:**  
Open file in binary mode and read all contents.

**Example Usage:**

```python
data = read_file('README.md')
print(f'Read {len(data)} bytes')
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parse a tree object either from its SHA-1 hash or raw data bytes, returning a list of entries describing files and subtrees.

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 hash of the tree object.  
- `data` (`bytes`, optional): Raw tree object data.

**Operation:**  
1. If `sha1` is provided, read the object data for that tree.  
2. Parse the data by iteratively extracting mode, path, and SHA-1 of each entry until all entries are processed.  
3. Return a list of `(mode, path, sha1)` tuples.

**Example Usage:**

```python
entries = read_tree(sha1='4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f'{oct(mode)} {path} {sha1}')
```

---

### `write_tree()`

**Purpose:**  
Create a tree object from the current Git index entries (staged files), and store it as a Git object.

**Operation:**  
1. Read index entries.  
2. For each entry, format a tree entry as `<mode> <path>\0<sha1>`.  
3. Concatenate all entries and hash them as a `tree` object.  
4. Return the SHA-1 hash of the new tree.

**Example Usage:**

```python
tree_sha1 = write_tree()
print(f'Written tree object {tree_sha1}')
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively find all objects reachable from a given tree object, including subtrees and blobs.

**Parameters:**  
- `tree_sha1` (`str`): SHA-1 of the root tree.

**Operation:**  
1. Initialize a set containing the tree itself.  
2. For each entry in the tree, if it is a subtree, recursively add its objects.  
3. Add all blob SHA-1s.  
4. Return the complete set.

**Example Usage:**

```python
objects = find_tree_objects(tree_sha1)
print(f'Objects in tree: {len(objects)}')
```

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively find all objects reachable from a commit: the commit itself, its tree, and all parent commits.

**Parameters:**  
- `commit_sha1` (`str`): SHA-1 hash of the commit.

**Operation:**  
1. Add the commit SHA-1 to the set.  
2. Read commit object and extract its tree and parent commits.  
3. Add all tree objects via `find_tree_objects`.  
4. Recursively add all parent commit objects.  
5. Return the set.

**Example Usage:**

```python
commit_objects = find_commit_objects(commit_sha1)
print(f'Total objects reachable from commit: {len(commit_objects)}')
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encode a single Git object for inclusion in a packfile. The encoding includes a variable-length header describing the object type and size, followed by the zlib-compressed object data.

**Parameters:**  
- `obj` (`str`): SHA-1 hash of the object to encode.

**Operation:**  
1. Read the object type and data.  
2. Convert the object type to a numeric code.  
3. Construct the header with the object type and size using a variable-length encoding scheme.  
4. Compress the data with zlib.  
5. Return the concatenated header and compressed data bytes.

**Example Usage:**

```python
pack_bytes = encode_pack_object('a1b2c3d4e5f6...')
print(f'Pack object size: {len(pack_bytes)} bytes')
```

---

### ASCII Diagram: Git Object Storage Layout

```
.git/
├── objects/
│   ├── ab/
│   │   └── cdef1234567890...  # Object with SHA-1 starting 'abcdef...'
│   ├── 12/
│   │   └── 3456789abcde...
│   └── ...
├── refs/
│   └── heads/
│       └── master             # Reference to latest commit SHA-1
└── index                     # Staging area entries
```

- Objects are stored in `.git/objects/` in subdirectories named by their first two hex characters.
- The remaining 38 hex characters form the filename.
- Objects are stored compressed.
- The index stores metadata about files staged for commit.

---

# Summary

This document provides a comprehensive technical reference for key Git object handling functions. Together, they enable low-level manipulation of the Git object database, supporting data hashing, storage, retrieval, and packfile encoding. Understanding these functions is critical for grasping how Git manages its content-addressable storage system efficiently and securely.