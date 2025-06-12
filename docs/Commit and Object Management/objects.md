# objects.md

# Handling of Git Objects: Hashing, Reading, and Packing

---

## Overview

This document details the handling of Git objects within the repository, focusing on the core operations of hashing data into Git objects, reading those objects from storage, and packing multiple objects for efficient transmission or storage. The file `objects.md` fits within the **Commit and Object Management** section of the documentation tree, which covers committing changes and managing Git objects such as commits, trees, and blobs.

The operations described here are foundational to Git's internal mechanics, enabling the creation, retrieval, and efficient transport of repository data. Functions such as `hash_object`, `read_object`, and `create_pack` are central to how Git stores content-addressed data and manages repository state.

---

## Important Functions

### 1. `hash_object(data, obj_type, write=True)`

#### Purpose

Computes the SHA-1 hash of the given data as a Git object of the specified type (e.g., `blob`, `tree`, `commit`). Optionally writes the compressed object data to the Git object store under `.git/objects`.

#### Parameters

- `data` (`bytes`): Raw content to be hashed.
- `obj_type` (`str`): The type of Git object (`blob`, `tree`, `commit`, etc.).
- `write` (`bool`, optional): Whether to write the object to the object store. Defaults to `True`.

#### Operation

1. Construct the Git object header: `"<obj_type> <len(data)>\0"`.
2. Concatenate the header and data.
3. Compute the SHA-1 hash of the concatenated bytes.
4. If writing is enabled:
    - Create the directory path `.git/objects/xx/` where `xx` are the first two hex digits of the SHA-1.
    - Compress the full object data using zlib.
    - Write the compressed data to `.git/objects/xx/yy...` where `yy...` are the remaining SHA-1 hex digits.
5. Return the SHA-1 hash as a hex string.

#### Example Usage

```python
data = b'Hello, Git!'
sha1 = hash_object(data, 'blob')
print(f'Object SHA-1: {sha1}')
```

---

### 2. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object from the object store using the specified SHA-1 prefix and returns its type and raw data.

#### Parameters

- `sha1_prefix` (`str`): A full or partial SHA-1 hex string identifying the object.

#### Preconditions

- The SHA-1 prefix must uniquely identify a single object in the object store.

#### Operation

1. Locate the object file path using the SHA-1 prefix.
2. Read and decompress the object data.
3. Parse the header to extract the object type and size.
4. Verify the decompressed data size matches the header size.
5. Return a tuple `(obj_type, data)` where:
   - `obj_type` is a string (e.g., `'blob'`, `'commit'`).
   - `data` is the raw bytes of the object's contents.

#### Example Usage

```python
obj_type, data = read_object('a1b2c3d4')
print(f'Object type: {obj_type}')
print(f'Object data:\n{data.decode()}')
```

---

### 3. `write_file(path, data)`

#### Purpose

Writes raw bytes to a file at the specified path.

#### Parameters

- `path` (`str`): Path to the file.
- `data` (`bytes`): Data to write.

#### Operation

- Opens the file in binary write mode and writes the data.

#### Example Usage

```python
write_file('.git/refs/heads/master', b'abc1234def\n')
```

---

### 4. `create_pack(objects)`

#### Purpose

Creates a Git packfile containing all objects identified by their SHA-1 hashes for efficient storage or transfer.

#### Parameters

- `objects` (`set[str]`): Set of SHA-1 hex strings representing objects to include.

#### Operation

1. Create a packfile header containing:
   - Signature: `'PACK'`
   - Version number (2)
   - Number of objects
2. Encode each object using `encode_pack_object` and concatenate.
3. Append a SHA-1 checksum of the entire packfile content.
4. Return the full packfile as bytes.

#### Example Usage

```python
objects_to_pack = {'a1b2c3d4...', '5f6e7d8c...'}
pack_data = create_pack(objects_to_pack)
write_file('.git/pack/pack-example.pack', pack_data)
```

#### ASCII Diagram: Packfile Structure

```
+-------------------------------+
| "PACK" signature (4 bytes)    |
+-------------------------------+
| Version number (4 bytes)       |
+-------------------------------+
| Number of objects (4 bytes)    |
+-------------------------------+
| Encoded objects (variable)     |
|  - Object header               |
|  - Compressed object data     |
+-------------------------------+
| SHA-1 checksum (20 bytes)      |
+-------------------------------+
```

---

### 5. `encode_pack_object(obj)`

#### Purpose

Encodes an individual Git object into the packfile format, including a variable-length header and compressed data.

#### Parameters

- `obj` (`str`): SHA-1 hex string of the object to encode.

#### Operation

1. Read the object type and data via `read_object`.
2. Map the object type to a type number.
3. Encode the object's size in a variable-length format.
4. Construct the packfile header byte(s) combining type and size.
5. Compress the object data with zlib.
6. Return concatenated header and compressed data as bytes.

#### Example Usage

```python
encoded = encode_pack_object('a1b2c3d4...')
# Use encoded bytes to build packfile body
```

---

### 6. `find_object(sha1_prefix)`

#### Purpose

Resolves a SHA-1 prefix to a specific object file path in the object store.

#### Parameters

- `sha1_prefix` (`str`): Partial or full SHA-1 hex string.

#### Operation

1. Validate prefix length (minimum 2 characters).
2. List files in `.git/objects/xx` where `xx` are first two chars of prefix.
3. Match files starting with the remaining prefix.
4. Ensure exactly one match exists.
5. Return the full path to the object file.

#### Example Usage

```python
path = find_object('a1b2c3')
print(f'Object file path: {path}')
```

---

### 7. `cat_file(mode, sha1_prefix)`

#### Purpose

Displays the contents or metadata of a Git object identified by SHA-1 prefix, supporting multiple output modes.

#### Parameters

- `mode` (`str`): One of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, `'pretty'`.
- `sha1_prefix` (`str`): SHA-1 prefix of the object.

#### Operation

- Reads the object.
- Depending on mode:
  - `commit/tree/blob`: Writes raw object data to stdout (must match mode).
  - `size`: Prints size of object data.
  - `type`: Prints object type.
  - `pretty`: Nicely formats output; for trees, lists entries.

#### Example Usage

```bash
pygit cat_file pretty 4a5e6f7
```

---

### 8. `read_tree(sha1=None, data=None)`

#### Purpose

Parses a Git tree object and returns a list of entries describing files and subtrees.

#### Parameters

- `sha1` (`str`, optional): SHA-1 of the tree object.
- `data` (`bytes`, optional): Raw tree data (if already loaded).

#### Preconditions

- Must provide either `sha1` or `data`.

#### Operation

1. If `sha1` is provided, read and decompress the tree object.
2. Parse the binary tree data into entries:
   - Each entry has mode, path, and SHA-1.
3. Return list of tuples `(mode, path, sha1)`.

#### Example Usage

```python
entries = read_tree(sha1='a1b2c3...')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### 9. `find_tree_objects(tree_sha1)`

#### Purpose

Recursively collects all object SHA-1 hashes contained within a tree object.

#### Parameters

- `tree_sha1` (`str`): SHA-1 hash of the tree object.

#### Operation

- Add the tree SHA-1 to a set.
- For each entry in the tree:
  - If entry is a subtree, recurse.
  - Else add blob SHA-1.
- Return the complete set.

#### Example Usage

```python
objects = find_tree_objects('a1b2c3...')
print(f'Objects in tree: {objects}')
```

---

### 10. `find_commit_objects(commit_sha1)`

#### Purpose

Recursively collects all object SHA-1 hashes reachable from a commit, including its tree and parent commits.

#### Parameters

- `commit_sha1` (`str`): SHA-1 hash of the commit object.

#### Operation

- Add commit SHA-1.
- Extract tree SHA-1 and find tree objects recursively.
- For each parent commit, recurse.
- Return set of all reachable objects.

#### Example Usage

```python
objs = find_commit_objects('f1e2d3c4...')
print(f'Objects in commit: {objs}')
```

---

## Summary ASCII Diagram: Git Object Relationships

```
Commit Object (commit_sha1)
|
|-- references --> Tree Object (tree_sha1)
|                   |
|                   |-- contains --> Blob Objects (files)
|                   |-- contains --> Tree Objects (subdirectories)
|
|-- references --> Parent Commits (optional)
```

This hierarchy allows efficient storage and retrieval of repository state, where objects are content-addressed by SHA-1 hashes.

---

# End of objects.md Documentation