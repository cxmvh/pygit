---
sidebar_position: 3
---

# Object Handling and Storage

## Overview

This document details the management of Git objects within the repository, focusing on the processes of creating, reading, writing, finding, and representing objects. It covers the core functions involved in object handling such as `hash_object`, `read_object`, and `write_object`, and explains how these operations underpin essential Git features like compression, pack file creation, and object lookup. This file integrates insights from related code flows including `pygit.cat_file`, `pygit.status`, `pygit.commit`, and `pygit.diff`, making it a central reference for understanding the object model and storage mechanisms in the repository core operations.

---

## hash_object

### Purpose

`hash_object` is responsible for creating a Git object from given data. It computes the SHA-1 hash of the object content, prepares the object header, optionally writes the object to the object store, and returns the object’s hash ID. This function is fundamental to adding new content to the repository in the form of blobs, trees, commits, or tags.

### Parameters

- `data` (bytes): The raw content of the object to be hashed.
- `obj_type` (str): The type of Git object being created. Common types include `'blob'`, `'tree'`, `'commit'`, and `'tag'`.
- `write` (bool, optional): Whether to write the object to the object database. Defaults to `True`.

### Operation Details

1. **Prepare Object Header:** Constructs a header string with the format `"{obj_type} {len(data)}\0"`.
2. **Concatenate Header and Data:** Combines the header and the data bytes.
3. **Compute SHA-1 Hash:** Calculates the SHA-1 hash of the combined bytes.
4. **Write Object (Optional):** If `write` is `True`, compresses the content and stores it under the `.git/objects` directory using the hash as the filename.
5. **Return Hash:** Returns the SHA-1 hash string as the object ID.

### Example Usage

```python
# Create and store a blob object from file content
file_content = b"Hello, Git!"
blob_hash = hash_object(file_content, obj_type='blob', write=True)
print(f"Stored new blob object with hash: {blob_hash}")
```

---

## read_object

### Purpose

`read_object` retrieves and decompresses a Git object from the object database given its SHA-1 hash. It returns the object’s type and its uncompressed content. This function is essential for accessing the stored data representing blobs, trees, commits, or tags.

### Parameters

- `sha` (str): The SHA-1 hash of the object to read.

### Operation Details

1. **Locate Object File:** Finds the object file in `.git/objects/` based on the SHA-1 hash prefix and suffix.
2. **Read and Decompress:** Reads the compressed content and decompresses it (typically using zlib).
3. **Parse Header:** Extracts the object type and size from the header portion of the decompressed data.
4. **Extract Data:** Returns the object type and the raw content bytes.

### Example Usage

```python
# Read a blob object by its hash
obj_type, content = read_object(blob_hash)
print(f"Object type: {obj_type}")
print(f"Content: {content.decode()}")
```

---

## write_object

### Purpose

`write_object` saves raw object data into the Git object store. It compresses the data and writes it to the appropriate location on disk, ensuring the object is stored in the canonical Git format.

### Parameters

- `sha` (str): The SHA-1 hash identifying the object.
- `data` (bytes): The raw object data including header.
- `git_dir` (str): Path to the `.git` directory where objects are stored.

### Operation Details

1. **Determine Object Path:** Based on the SHA-1 hash, the object is stored in a subdirectory named by the first two characters of the hash, with the remainder as the filename.
2. **Create Directory If Needed:** Ensures the subdirectory exists.
3. **Compress Data:** Compresses the raw object data using zlib.
4. **Write Compressed Data:** Writes the compressed bytes to the object file.
5. **Handle Existing Objects:** If the object already exists, writing may be skipped to avoid duplication.

### Example Usage

```python
# Assuming we have raw object data with header
raw_data = b"blob 12\0Hello, Git!"
sha = 'f572d396fae9206628714fb2ce00f72e94f2258f'  # Example SHA-1
write_object(sha, raw_data, git_dir='/path/to/repo/.git')
```

---

## Object Finding and Representation

### Overview

Finding objects involves locating objects by their SHA-1 hashes or partial prefixes within the object database. Representation covers converting these objects into human-readable forms, such as displaying file contents (blobs), directory trees (trees), commit logs, or tag annotations.

### Concepts

- **Object Database Layout:**

  Git stores objects in the `.git/objects` directory using a two-level folder structure based on the hash:

  ```
  .git/
    objects/
      ab/
        cd1234...  # Object file for hash starting with 'abcd1234...'
      ...
  ```

- **Object Lookup Process:**

  1. The first two characters of the SHA-1 hash determine the subdirectory.
  2. The remaining characters form the filename.
  3. The file is read and decompressed to retrieve the object.

- **Representation Example:**

  - **Blob:** Raw file content.
  - **Tree:** List of filenames and their associated object hashes.
  - **Commit:** Metadata including author, committer, commit message, and parent commits.
  - **Tag:** Annotated tag information.

### ASCII Diagram: Object Storage Layout

```
.git/objects/
├── 1a/
│   ├── 2b3c4d5e6f7g8h9i0jklmnopqrstuvwx  # Object file for SHA-1 1a2b3c4d5e6f7g8h9i0jklmnopqrstuvwx
├── 5f/
│   ├── a1b2c3d4e5f67890123456789abcdef012  # Object file for SHA-1 5fa1b2c3d4e5f67890123456789abcdef012
...
```

---

## Integration with Core Git Operations

The object handling functions form the foundation for several core Git operations:

- **pygit.cat_file:** Uses `read_object` to display object contents.
- **pygit.status:** Reads objects to compare index and working directory states.
- **pygit.commit:** Creates commit objects via `hash_object`, and reads them for history traversal.
- **pygit.diff:** Reads object contents to generate diffs between commits or the index.

These integrations highlight the importance of reliable object storage and retrieval in maintaining repository integrity and supporting Git workflows.

---

This documentation serves as a comprehensive reference for developers working with Git's object storage mechanics and their interactions within the broader repository core operations. For detailed explanations of related topics such as tree and commit handling, indexing, and diff generation, consult the linked documentation files in the [Repository Core Operations section](./).