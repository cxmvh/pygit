# objects.md

## Overview

This document provides detailed technical reference for functions related to Git object handling within the `pygit` project. It covers the core capabilities of hashing objects, reading and locating objects in the Git object store, encoding objects for pack files, and recursively discovering objects within trees and commits. These functions are fundamental to Git’s storage mechanism, enabling efficient content-addressable storage, object retrieval, and transfer operations. This file fits within the "Git Object Handling" section of the documentation tree and supports key commands like `status`, `init`, and `commit`.

---

## Function Documentation

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of the given object data with its Git object header and optionally writes the compressed object to the Git object store.

**Parameters:**  
- `data` (`bytes`): Raw bytes of the object content (e.g., file content for blobs).  
- `obj_type` (`str`): Type of the object, one of `"blob"`, `"tree"`, `"commit"`.  
- `write` (`bool`): If `True`, write the compressed object to `.git/objects`. Defaults to `True`.

**Operation Steps:**  
1. Construct the Git object header: `"<obj_type> <size>"` encoded as bytes.  
2. Concatenate header, a null byte, and the object data to form `full_data`.  
3. Calculate the SHA-1 hash of `full_data`.  
4. If `write` is `True`, check if the object is already stored; if not, compress `full_data` with zlib and write to `.git/objects/xx/yyyy...` where `xx` are the first two hex digits and `yyyy...` the rest.  
5. Return the SHA-1 hash string.

**Example Usage:**
```python
content = b'Hello, World!\n'
sha1_hash = hash_object(content, 'blob')
print("Object hash:", sha1_hash)
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object from the object store given a SHA-1 prefix and returns its type and data.

**Parameters:**  
- `sha1_prefix` (`str`): A prefix of the SHA-1 hash of the object to read.

**Operation Steps:**  
1. Use `find_object` to locate the exact object file path in `.git/objects`.  
2. Read and decompress the object file content.  
3. Parse the header to extract object type and size.  
4. Validate the size matches the length of the data portion.  
5. Return a tuple `(obj_type, data_bytes)`.

**Example Usage:**
```python
obj_type, data = read_object('3a5f7b')
print("Object type:", obj_type)
print("Data:", data)
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Finds the file path of an object in the Git object store matching the given SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (`str`): Prefix (at least 2 characters) of the SHA-1 object hash.

**Operation Steps:**  
1. Validate prefix length (minimum 2 characters).  
2. List all files in `.git/objects/<first two chars of prefix>`.  
3. Filter files starting with the remainder of the prefix.  
4. If none or multiple matches found, raise `ValueError`.  
5. Return full path to the object file.

**Example Usage:**
```python
path = find_object('3a5f7b')
print("Object path:", path)
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encodes a single Git object for inclusion in a pack file using Git’s variable-length header encoding.

**Parameters:**  
- `obj` (`str`): SHA-1 hash string of the object to encode.

**Operation Steps:**  
1. Read the object type and data using `read_object`.  
2. Map the object type to a numeric code.  
3. Prepare the variable-length size header using 7-bit encoding with continuation bits.  
4. Compress the object data with zlib.  
5. Return concatenated header bytes and compressed data.

**Example Usage:**
```python
packed_bytes = encode_pack_object('3a5f7b...')
with open('object.pack', 'wb') as f:
    f.write(packed_bytes)
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds all object hashes referenced by a tree object, including nested trees and blobs.

**Parameters:**  
- `tree_sha1` (`str`): SHA-1 hash of the root tree object.

**Operation Steps:**  
1. Initialize a set with the root `tree_sha1`.  
2. Read the tree entries via `read_tree`.  
3. For each entry, if it is a tree (directory), recursively add all objects from that subtree.  
4. For blobs, add their SHA-1 hashes directly.  
5. Return the accumulated set of object hashes.

**Example Usage:**
```python
all_objects = find_tree_objects('d670460b4b4aece5915caf5c68d12f560a9fe3e4')
print("Objects in tree:", all_objects)
```

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively finds all object hashes reachable from a commit object, including its tree, parents, and all nested objects.

**Parameters:**  
- `commit_sha1` (`str`): SHA-1 hash of the commit object.

**Operation Steps:**  
1. Initialize a set with the commit SHA-1.  
2. Read the commit object and parse its `tree` and `parent` references.  
3. Use `find_tree_objects` to find all objects in the commit’s tree.  
4. Recursively process each parent commit to include their objects.  
5. Return the combined set of all reachable object hashes.

**Example Usage:**
```python
commit_objects = find_commit_objects('a1b2c3d4e5f6...')
print("All commit-related objects:", commit_objects)
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a tree object and returns a list of entries consisting of mode, path, and SHA-1 for each item.

**Parameters:**  
- `sha1` (`str`, optional): SHA-1 hash of the tree object to read.  
- `data` (`bytes`, optional): Raw data of the tree object. Must specify either `sha1` or `data`.

**Operation Steps:**  
1. If `sha1` is given, read and verify the object type is `tree`.  
2. Parse the tree data by locating null bytes separating mode/path and SHA-1 binary.  
3. Convert mode from octal string to integer, and SHA-1 to hex string.  
4. Collect entries until no more entries are found.  
5. Return a list of `(mode, path, sha1)` tuples.

**Example Usage:**
```python
entries = read_tree(sha1='d670460b4b4aece5915caf5c68d12f560a9fe3e4')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### `ls_files(details=False)`

**Purpose:**  
Lists files currently staged in the Git index.

**Parameters:**  
- `details` (`bool`): If `True`, prints file mode, SHA-1, stage number, and path. Otherwise, prints paths only.

**Operation Steps:**  
1. Read index entries using `read_index()`.  
2. For each entry, print path or detailed info depending on `details`.  
3. Stage number is extracted from flags.

**Example Usage:**
```python
ls_files(details=True)
```

Output:
```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100644 9daeafb9864cf43055ae93beb0afd6c7d144bfa4 0    main.py
```

---

### `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries and writes it to the object store.

**Operation Steps:**  
1. Read index entries.  
2. For each entry, assert it is top-level (no slash in path).  
3. Format each entry as `<mode> <path>\0<sha1-bytes>`.  
4. Concatenate all entries and hash as a tree object via `hash_object`.  
5. Return SHA-1 of the tree object.

**Example Usage:**
```python
tree_sha1 = write_tree()
print("Tree object created:", tree_sha1)
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Determines which objects in the local commit are missing from the remote repository.

**Parameters:**  
- `local_sha1` (`str`): SHA-1 of the local commit.  
- `remote_sha1` (`str` or `None`): SHA-1 of the remote commit or `None` if remote has no commits.

**Operation Steps:**  
1. Retrieve all objects reachable from the local commit using `find_commit_objects`.  
2. If the remote commit is `None`, all local objects are missing.  
3. Otherwise, retrieve remote objects and compute difference.  
4. Return the set of missing SHA-1 hashes.

**Example Usage:**
```python
missing = find_missing_objects('localsha1...', 'remotesha1...')
print(f"Objects missing from remote: {len(missing)}")
```

---

### `push(git_url, username=None, password=None)`

**Purpose:**  
Pushes the local `master` branch to the remote Git repository.

**Parameters:**  
- `git_url` (`str`): URL of the remote Git repository.  
- `username` (`str`, optional): Username for authentication.  
- `password` (`str`, optional): Password for authentication.

**Operation Steps:**  
1. Obtain remote master SHA-1 via `get_remote_master_hash`.  
2. Obtain local master SHA-1 via `get_local_master_hash`.  
3. Determine missing objects to push.  
4. Build Git protocol command lines and create a pack file with missing objects.  
5. Send HTTP POST request with the pack data to the remote `git-receive-pack` endpoint.  
6. Verify remote response for successful unpack and update.  
7. Return remote SHA-1 and missing objects set.

**Example Usage:**
```python
push('https://github.com/user/repo.git', username='user', password='pass')
```

---

## ASCII Diagram: Git Object Storage and Lookup

```
.git/
└── objects/
    ├── 3a/
    │   └── 5f7b...  (compressed object file)
    ├── d6/
    │   └── 7046...  (compressed object file)
    └── ...         (other object folders)

hash_object() flow:
+------------+       compress       +------------+
| raw object | -----------------> | .git/objects/ |
| (header + data) |               | <sha1[:2]>/<sha1[2:]> |
+------------+                   +------------+

read_object() flow:
+------------+       decompress     +------------+
| .git/objects/ | --------------> | raw object  |
+------------+                    +------------+
```

---

This documentation provides a comprehensive reference to the critical functions that implement Git’s object storage and manipulation within the `pygit` project, enabling users and developers to understand and extend the internal object handling mechanisms.