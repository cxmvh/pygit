---
sidebar_position: 2
---

# Object Storage and Handling

## Overview

This document provides a comprehensive reference for managing Git object storage and handling within the pygit library. It covers essential operations such as reading and writing Git objects—including commits, trees, and blobs—retrieving objects by SHA-1 prefix, hashing, compressing object data, and reading tree structures. Additionally, it details the `cat_file` functionality, which allows inspection of object contents.

A significant focus of this documentation is on the creation and encoding of pack files, which are crucial for efficient transfer and storage of Git objects. Pack files optimize network usage and storage by compressing and consolidating multiple objects.

This file fits within the broader Git Library Documentation as a core module responsible for low-level object manipulation, supporting higher-level operations like commits, repository state management, and remote interactions.

---

## Functions

### `cat_file`

**Purpose:**  
The `cat_file` function is used to display the content or type of a Git object identified by its SHA-1 hash. It can output the raw content or object metadata, which is helpful for inspecting repository objects.

**Parameters:**  
- `object_hash` (str): The full SHA-1 hash of the object to display.  
- `mode` (str, optional): The display mode; common modes include `'type'` (object type) and `'content'` (raw object content).

**Operation:**  
1. Locate the object in the object database using the provided SHA-1 hash.  
2. Decompress the object data if it is stored in compressed form.  
3. Depending on the mode, either return the object's type (e.g., commit, tree, blob) or its raw content.

**Example Usage:**

```python
# Display the type of the object
print(pygit.cat_file('a1b2c3d4e5f6g7h8i9j0k', mode='type'))  # Output: 'commit'

# Display the content of the object
content = pygit.cat_file('a1b2c3d4e5f6g7h8i9j0k', mode='content')
print(content)
```

---

### `read_object`

**Purpose:**  
Reads and decompresses a Git object from the object database using its SHA-1 hash, returning both its type and content.

**Parameters:**  
- `object_hash` (str): The SHA-1 hash of the desired object.

**Operation:**  
1. Locate the object file in the `.git/objects` directory, using the first two characters of the hash as a directory and the remaining characters as the filename.  
2. Read the compressed object file.  
3. Decompress the data using zlib.  
4. Parse the decompressed data into object type and raw content.

**Example Usage:**

```python
obj_type, obj_content = pygit.read_object('4f5e6d7c8b9a0e1d2c3b')
print(f"Object type: {obj_type}")
print(f"Content:\n{obj_content.decode('utf-8')}")
```

---

### `find_object`

**Purpose:**  
Resolves a partial SHA-1 prefix to its full hash if it uniquely identifies an object in the database.

**Parameters:**  
- `prefix` (str): The SHA-1 prefix to resolve (must be at least a few characters).

**Operation:**  
1. Search the `.git/objects` directory for objects whose SHA-1 hashes start with the given prefix.  
2. If exactly one match is found, return the full object hash.  
3. If no matches or multiple matches are found, raise an error or return `None`.

**Example Usage:**

```python
full_hash = pygit.find_object('a1b2c3')
if full_hash:
    print(f"Full hash resolved: {full_hash}")
else:
    print("No unique object found for prefix.")
```

---

### `read_tree`

**Purpose:**  
Reads a tree object and returns a list of entries representing files and subdirectories contained in the tree.

**Parameters:**  
- `tree_hash` (str): The SHA-1 hash of the tree object.

**Operation:**  
1. Use `read_object` to load the tree content.  
2. Parse the tree data, which consists of multiple entries, each containing mode, filename, and SHA-1 hash of the object.  
3. Return the entries as a structured list or dictionary for further processing.

**Example Usage:**

```python
entries = pygit.read_tree('abc123def456...')
for entry in entries:
    print(f"{entry.mode} {entry.name} {entry.sha1}")
```

**ASCII Diagram:**

```
Tree Object Structure:

+----------------------+----------------+---------------------+
| Mode (e.g., 100644)  | Filename       | Object SHA-1 (20 B) |
+----------------------+----------------+---------------------+
| 6 ASCII bytes + null | Variable bytes | 20 bytes binary     |
+----------------------+----------------+---------------------+

Multiple such entries are concatenated within the tree object.
```

---

### `hash_object`

**Purpose:**  
Creates a new Git object by hashing and optionally writing content to the object database.

**Parameters:**  
- `data` (bytes): The content to hash and store.  
- `object_type` (str): The type of the object (e.g., `'blob'`, `'commit'`, `'tree'`).  
- `write` (bool, optional): Whether to write the object to the database (default: `True`).

**Operation:**  
1. Prepare the object header: `"<object_type> <length>\0"`.  
2. Concatenate the header and data.  
3. Compute the SHA-1 hash of the concatenated bytes.  
4. If `write` is `True`, compress the object data and write it to the appropriate location in `.git/objects`.  
5. Return the SHA-1 hash.

**Example Usage:**

```python
content = b"Hello, Git!"
hash = pygit.hash_object(content, 'blob')
print(f"Created object with hash: {hash}")
```

---

### `encode_pack_object`

**Purpose:**  
Encodes a single Git object for inclusion in a pack file, applying compression and delta encoding if appropriate.

**Parameters:**  
- `object_hash` (str): The SHA-1 hash of the object to encode.  
- `base_object` (optional): The base object for delta encoding (can be None if no delta).  
- `output_stream`: A writable stream to output the encoded data.

**Operation:**  
1. Read the object content and type.  
2. If a base object is provided, compute the delta between the base and current object.  
3. Compress the data using zlib.  
4. Write the pack object header and compressed data to the output stream.

**Example Usage:**

```python
with open('packfile.pack', 'wb') as pack_stream:
    pygit.encode_pack_object('deadbeef...', base_object=None, output_stream=pack_stream)
```

---

### `create_pack`

**Purpose:**  
Creates a pack file consisting of multiple Git objects for efficient transfer or storage.

**Parameters:**  
- `object_hashes` (list of str): List of SHA-1 hashes of the objects to include in the pack.  
- `output_path` (str): File path to write the resulting pack file.

**Operation:**  
1. Open the output file for writing.  
2. Write the pack file header, which includes the signature and version number.  
3. Iterate over the list of objects, encoding each with `encode_pack_object`.  
4. Compute and append the SHA-1 checksum of the entire pack file.  
5. Close the file.

**Example Usage:**

```python
objects_to_pack = ['a1b2c3...', 'd4e5f6...', '789abc...']
pygit.create_pack(objects_to_pack, 'objects.pack')
print("Pack file created successfully.")
```

**ASCII Diagram:**

```
Pack File Structure:

+----------------+----------------+----------------+-----------------+
| Signature (4B) | Version (4B)   | Object Count   | Encoded Objects |
+----------------+----------------+----------------+-----------------+
| "PACK"         | 2 (network ord)| N (network ord)| Compressed data |
+----------------+----------------+----------------+-----------------+
                                            |
                                            +--> [Object 1: header + compressed data]
                                            +--> [Object 2: header + compressed data]
                                            +--> ...
+----------------+
| SHA-1 Checksum |
+----------------+
```

---

This document provides the foundational details for handling Git objects within the pygit library, enabling developers to build upon or integrate object storage and transfer functionality effectively. For related topics such as repository initialization or commit management, see the corresponding documentation files in the Git Library Documentation section.