# objects.md

# Handling Git Objects: Hashing, Reading, and Packing

---

## Overview

This document covers the handling of Git objects within the repository, focusing on their core operations such as hashing, reading, locating, and packing. It is part of the broader **Git Object Handling** section, which provides a comprehensive guide on managing Git objects, crucial for storing commits, trees, blobs, and other Git entities.

The functions documented here form the backbone of the Git object management system implemented in `pygit`. They facilitate converting data into Git objects by hashing and compressing, reading stored objects, finding objects by SHA-1 prefixes, and creating pack files for efficient data transfer. These operations are foundational for commands like `commit`, `status`, and `push` and underpin repository consistency and performance.

---

## Function Documentation

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of the given object data for a specified Git object type (`commit`, `tree`, `blob`, etc.). Optionally writes the compressed object to the Git object store under `.git/objects`. Returns the SHA-1 hash as a hexadecimal string.

**Parameters:**  
- `data` (`bytes`): Raw data of the Git object.  
- `obj_type` (`str`): Type of Git object (e.g., 'blob', 'tree', 'commit').  
- `write` (`bool`, optional): Whether to write the compressed object to the object store (default `True`).

**Operation:**  
1. Create the Git object header in the form: `<obj_type> <data length>\0`.  
2. Concatenate the header and data.  
3. Compute the SHA-1 hash of this full data.  
4. If `write` is `True`, compress the full data using zlib and write it under `.git/objects/<first two sha1 chars>/<remaining sha1 chars>`. Create directories if missing.  
5. Return the SHA-1 hex digest.

**Example Usage:**
```python
data = b'Hello, World!\n'
sha1_hash = hash_object(data, 'blob')
print(f"Object stored with SHA-1: {sha1_hash}")
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads and decompresses a Git object identified by its SHA-1 prefix from the object store, returning the object's type and raw data.

**Parameters:**  
- `sha1_prefix` (`str`): A prefix of the SHA-1 hash identifying the object.

**Operation:**  
1. Locate the full object path by expanding the prefix using `find_object`.  
2. Read the compressed object file and decompress it.  
3. Parse the header to extract the object type and size.  
4. Extract the object data following the header.  
5. Verify the size matches the expected length.  
6. Return a tuple `(object_type, data_bytes)`.

**Example Usage:**
```python
obj_type, data = read_object('9daeafb')
print(f"Object type: {obj_type}")
print(f"Object data:\n{data.decode()}")
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Finds the object file path corresponding to a given SHA-1 prefix in the `.git/objects` directory. Raises an error if the object is not found or the prefix is ambiguous.

**Parameters:**  
- `sha1_prefix` (`str`): The beginning of the SHA-1 hash to locate.

**Operation:**  
- Search the `.git/objects/<prefix first two chars>/` directory for files starting with the remaining prefix.  
- If exactly one match is found, return the full path; otherwise, raise an error.

**Example Usage:**
```python
path = find_object('9daeafb')
print(f"Object file path: {path}")
```

---

### `create_pack(objects)`

**Purpose:**  
Generates a Git pack file containing all provided objects. Pack files are used to efficiently transfer multiple objects in a compressed format.

**Parameters:**  
- `objects` (`set` of `str`): Set of SHA-1 hashes of Git objects to include.

**Operation:**  
1. Create a pack header: ASCII "PACK", version number (2), and number of objects.  
2. Encode each object using `encode_pack_object` (not detailed here).  
3. Concatenate header and encoded objects.  
4. Compute SHA-1 checksum of the pack contents and append it.  
5. Return the complete pack file as bytes.

**Example Usage:**
```python
objects_to_pack = {'9daeafb...', '1a410ef...'}
pack_data = create_pack(objects_to_pack)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds and returns all SHA-1 hashes of objects contained in a tree object, including nested trees and blobs.

**Parameters:**  
- `tree_sha1` (`str`): SHA-1 hash of the root tree object.

**Operation:**  
1. Initialize a set containing the root `tree_sha1`.  
2. Read the entries of the tree object.  
3. For each entry:  
   - If it is a directory (tree), recursively collect objects from it.  
   - Otherwise, add the blob's SHA-1 to the set.  
4. Return the complete set of object hashes.

**Example Usage:**
```python
objects = find_tree_objects('4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for obj in objects:
    print(obj)
```

---

### ASCII Diagram: Git Object Storage Structure

```
.git/
  objects/
    aa/
      bbccddeeff...   # Object file named after SHA-1 suffix
    9d/
      aeafb123456...  # Example compressed Git object file
  refs/
    heads/
      master          # Reference to commit SHA-1
  index               # Git index file
```

- The first two hex digits of the SHA-1 form a directory under `.git/objects/`.
- The remaining 38 hex digits form the filename in that directory.
- Objects are stored compressed.

---

### Additional Related Functions (Brief Descriptions)

- **`write_file(path, data)`**: Writes raw bytes to a file at the specified path.
- **`read_file(path)`**: Reads raw bytes from a file at the specified path.
- **`write_tree()`**: Creates a tree object from the current index entries by hashing their concatenated modes, paths, and SHA-1s.
- **`get_local_master_hash()`**: Reads the current commit SHA-1 hash from the local master branch reference.
- **`write_index(entries)`**: Serializes and writes the index entries to the `.git/index` file.
- **`read_index()`**: Reads and parses the `.git/index` file, returning a list of index entries.
- **`hash_object()` and `write_file()`** are used extensively in the commit process to create and store Git objects.

---

## Summary

The functions in this file underpin the core mechanisms of Git's object storage model: creating objects from data, storing them efficiently, retrieving and interpreting them, and preparing sets of objects for transfer via pack files. Mastery of these functions is essential for understanding the internal workings of Git and for implementing Git operations like commits, pushes, and status checks.

---

# End of `objects.md` documentation file content.