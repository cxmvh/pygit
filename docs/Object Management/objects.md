# objects.md

# Git Object Management: Reading, Writing, Finding, Encoding, and Hashing

## Overview

This document provides comprehensive technical reference material on handling Git objects within the repository. It covers the life cycle and manipulation of Git objects including blobs, trees, commits, and pack files. This file plays a critical role in the "Object Management" section of the overall documentation tree, focusing on low-level Git internals related to objects stored in the `.git/objects` directory.

The functionality documented here enables reading objects from the object store, writing new objects, locating objects by SHA-1 prefixes, encoding objects for pack files, and traversing commit and tree objects recursively. These operations underpin higher-level commands like commit creation, diff display, and push operations.

---

## Important Functions

---

### `read_object(sha1_prefix)`

#### Purpose
Reads a Git object identified by a SHA-1 hash prefix from the local object store and returns its type and raw data.

#### Parameters
- `sha1_prefix` (str): The prefix of the SHA-1 hash identifying the object.

#### Operation
1. Uses `find_object` to locate the file path corresponding to the full SHA-1 hash matched by the prefix.
2. Reads and decompresses the object file contents with zlib.
3. Parses the header to extract the object type (e.g., `blob`, `tree`, `commit`) and size.
4. Extracts the raw object data.
5. Validates that the size in the header matches the actual data length.
6. Returns a tuple `(obj_type, data)`.

#### Example
```python
obj_type, data = read_object('9daeafb')  # SHA-1 prefix of a blob object
print(f'Object type: {obj_type}')
print(f'Data (first 100 bytes): {data[:100]}')
```

---

### `find_object(sha1_prefix)`

#### Purpose
Locate the file path of a Git object in the `.git/objects` directory given a SHA-1 prefix.

#### Parameters
- `sha1_prefix` (str): The SHA-1 prefix string (minimum 2 characters).

#### Operation
1. Validates prefix length is at least 2 characters.
2. Constructs the directory path based on first two characters of the SHA-1 prefix.
3. Lists files in that directory that match the remaining prefix.
4. Raises an error if no or multiple objects match.
5. Returns the full filesystem path to the object file.

#### Example
```python
path = find_object('9daeafb')
print(f'Object file path: {path}')
```

---

### `hash_object(data, obj_type, write=True)`

#### Purpose
Compute the SHA-1 hash of object data with its Git header, optionally write the compressed object to the object store.

#### Parameters
- `data` (bytes): Raw object data.
- `obj_type` (str): Type of the object (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (bool, default `True`): Whether to write the object to storage.

#### Operation
1. Constructs a Git object header: `"<obj_type> <data_length>\0"`.
2. Concatenates header and data.
3. Computes SHA-1 hash of this full data.
4. If `write` is True, stores the compressed data in `.git/objects/<first2>/<remaining>` path.
5. Returns the SHA-1 hash as a hex string.

#### Example
```python
with open('README.md', 'rb') as f:
    blob_data = f.read()
sha1 = hash_object(blob_data, 'blob')
print(f'Object SHA-1: {sha1}')
```

---

### `read_tree(sha1=None, data=None)`

#### Purpose
Parse a Git tree object from either a SHA-1 hash or raw data and return a list of entries.

#### Parameters
- `sha1` (str, optional): SHA-1 hash of the tree object.
- `data` (bytes, optional): Raw data of the tree object.

#### Preconditions
- Exactly one of `sha1` or `data` must be provided.

#### Operation
1. If `sha1` is specified, calls `read_object` to get raw tree data.
2. Iterates over tree entries formatted as `<mode> <path>\0<20-byte SHA-1>`.
3. Decodes mode and path, converts mode to integer.
4. Extracts SHA-1 hash of the entry object.
5. Returns a list of tuples `(mode, path, sha1_hex)`.

#### Example
```python
entries = read_tree(sha1='4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### `write_tree()`

#### Purpose
Create and write a tree object representing the current state of the Git index.

#### Operation
1. Reads the index entries.
2. For each entry, constructs a tree entry: `<mode> <path>\0<sha1>`.
3. Joins these entries and hashes them as a `tree` object.
4. Writes the tree object to the object store.
5. Returns the SHA-1 hash of the tree.

#### Example
```python
tree_sha1 = write_tree()
print(f'Written tree object: {tree_sha1}')
```

---

### `find_commit_objects(commit_sha1)`

#### Purpose
Recursively find all objects reachable from a commit, including its tree, parents, and the commit itself.

#### Parameters
- `commit_sha1` (str): SHA-1 hash of the commit object.

#### Operation
1. Reads the commit object.
2. Extracts the tree SHA-1 and parent commit SHA-1(s).
3. Recursively collects all tree objects and their entries.
4. Recursively collects parent commit objects.
5. Returns a set of all SHA-1 hashes involved.

#### Example
```python
objects = find_commit_objects('9daeafb9864cf43055ae93beb0afd6c7d144bfa4')
print(f'All objects in commit: {objects}')
```

---

### `encode_pack_object(obj)`

#### Purpose
Encode a single Git object into the pack file format (variable-length header + compressed data).

#### Parameters
- `obj` (str): SHA-1 hash of the object to encode.

#### Operation
1. Reads the object type and data.
2. Maps object type to numeric code.
3. Encodes object size and type in a variable-length header.
4. Compresses the object data.
5. Returns the concatenated header and compressed data bytes.

#### Example
```python
pack_bytes = encode_pack_object('9daeafb9864cf43055ae93beb0afd6c7d144bfa4')
print(f'Pack object size: {len(pack_bytes)} bytes')
```

---

### `create_pack(objects)`

#### Purpose
Create a full Git pack file from a set of object SHA-1 hashes.

#### Parameters
- `objects` (set of str): SHA-1 hashes of objects to include.

#### Operation
1. Writes a pack header including version and object count.
2. Encodes each object using `encode_pack_object`.
3. Concatenates all encoded objects.
4. Appends SHA-1 checksum of all contents.
5. Returns the full pack file bytes.

#### Example
```python
objects = {'9daeafb9864cf43055ae93beb0afd6c7d144bfa4'}
pack_data = create_pack(objects)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
```

---

### `cat_file(mode, sha1_prefix)`

#### Purpose
Prints the content or information about an object given its SHA-1 prefix in various modes.

#### Parameters
- `mode` (str): One of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, or `'pretty'`.
- `sha1_prefix` (str): SHA-1 prefix of the object.

#### Operation
- For `'commit'`, `'tree'`, `'blob'`: prints raw object data, verifying type.
- For `'size'`: prints size of object data.
- For `'type'`: prints object type.
- For `'pretty'`: prints prettified version (e.g., formatted tree entries).
- Raises error for unexpected modes.

#### Example
```python
cat_file('pretty', '9daeafb')
```

---

### `read_index()`

#### Purpose
Reads the Git index file into a list of `IndexEntry` objects.

#### Operation
1. Reads `.git/index` file data.
2. Validates header signature, version, and checksum.
3. Parses each index entry (metadata + path).
4. Returns a list of entries.

#### Example
```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### `write_index(entries)`

#### Purpose
Writes a list of `IndexEntry` objects back to the Git index file.

#### Parameters
- `entries` (list of `IndexEntry`): Entries to write.

#### Operation
1. Packs each entry into binary format.
2. Concatenates entries with index header.
3. Computes SHA-1 checksum.
4. Writes to `.git/index`.

---

### `get_status()`

#### Purpose
Determines the status of the working directory relative to the index.

#### Returns
- Tuple `(changed_paths, new_paths, deleted_paths)`: lists of file paths.

#### Operation
1. Walks the working directory, ignoring `.git`.
2. Compares file SHA-1 hashes to index entries.
3. Classifies files as changed, new, or deleted.

---

### `add(paths)`

#### Purpose
Add specified files to the Git index.

#### Parameters
- `paths` (list of str): File paths to add.

#### Operation
1. Reads current index.
2. Hashes and creates new index entries for specified paths.
3. Writes updated index.

---

### `commit(message, author=None)`

#### Purpose
Create a new commit object from the current index state.

#### Parameters
- `message` (str): Commit message.
- `author` (str, optional): Author identity string.

#### Operation
1. Writes a tree object from index.
2. Retrieves parent commit SHA-1.
3. Constructs commit metadata with timestamps.
4. Hashes and writes commit object.
5. Updates `refs/heads/master`.

#### Example
```python
commit_sha1 = commit("Initial commit")
print(f"Commit created: {commit_sha1}")
```

---

### ASCII Diagram: Git Object Storage Layout

```
.git/objects/
├── <first two hex digits>/
│   ├── <remaining 38 hex digits>  # compressed object file
│   ├── ...
│
├── ...
```

Objects are stored in folders named by the first two characters of their SHA-1 hash with files named by the remaining characters.

---

### ASCII Diagram: Commit Object Structure

```
commit object: "commit <size>\0<data>"

<data> example:
tree <tree_sha1>
parent <parent_sha1>  (optional)
author <author> <timestamp> <timezone>
committer <committer> <timestamp> <timezone>

<blank line>

<commit message>
```

---

# Summary

This document specifies the core functionality for handling Git objects at a low level, including reading, writing, hashing, indexing, and packing. These fundamental capabilities enable higher-level Git commands and workflows within the repository. The consistent use of SHA-1 hashes, compression, and object typing forms the foundation of Git's data integrity and efficient storage model.