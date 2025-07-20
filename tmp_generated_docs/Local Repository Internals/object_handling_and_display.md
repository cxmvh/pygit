---
sidebar_position: 2
---

# Object Handling and Display

## Overview

This document details the process of reading, validating, parsing, and displaying Git objects—specifically commits, trees, and blobs—within the local repository internals. It focuses on the core functions `cat_file`, `read_object`, and `read_tree` that interact with the Git object database. These routines are essential for inspecting and manipulating Git objects at a low level, enabling developers to understand the structure and content of objects stored in the `.git/objects` directory. This file complements the broader repository internals documentation by providing practical insights and examples for working directly with Git’s object model.

---

## Functions

### `cat_file`

**Purpose:**  
The `cat_file` function is responsible for retrieving the raw contents and metadata of a Git object, given its SHA-1 hash. It serves as a low-level tool to read any type of object (commit, tree, blob) from the object database, decompress the stored data, and return it for further processing.

**Parameters:**  
- `sha`: The SHA-1 hash string identifying the Git object to read.

**Operation:**  
1. Locate the object file within the `.git/objects` directory using the SHA-1 hash prefix and suffix.
2. Read the compressed object data from the file.
3. Decompress the object data using zlib (or equivalent decompression).
4. Return the raw decompressed data along with the object type (e.g., `commit`, `tree`, `blob`) and size.

**Example Usage:**

```python
obj_type, obj_data = cat_file('9fceb02...')
print(f"Object Type: {obj_type}")
print(f"Object Data:\n{obj_data.decode('utf-8')}")
```

---

### `read_object`

**Purpose:**  
`read_object` builds upon `cat_file` by parsing the raw object data into a structured format depending on the object type. This function validates the object and prepares it for programmatic inspection or manipulation.

**Parameters:**  
- `sha`: The SHA-1 hash of the object to read.

**Operation:**  
1. Call `cat_file` to get the raw object type and data.
2. Validate the object type against known Git object types (`commit`, `tree`, `blob`).
3. Parse the object data:
   - For **commits**, parse the header fields (tree, parents, author, committer, message).
   - For **trees**, parse entries (mode, filename, SHA).
   - For **blobs**, treat data as raw file content.
4. Return a structured object representation (e.g., dictionary or class instance) appropriate to the Git object type.

**Example Usage:**

```python
commit_obj = read_object('9fceb02...')
print(f"Commit tree SHA: {commit_obj['tree']}")
print(f"Commit message: {commit_obj['message']}")
```

---

### `read_tree`

**Purpose:**  
The `read_tree` function parses a tree object to extract its entries, which represent files and subdirectories in a Git repository at a specific commit or tree state. It translates the raw tree object data into a list of entries with detailed metadata.

**Parameters:**  
- `sha`: The SHA-1 hash of the tree object to read.

**Operation:**  
1. Use `read_object` to get the raw tree object data.
2. Parse the tree entries sequentially:
   - Each entry consists of a file mode, filename, and SHA-1 hash of the referenced object.
3. Return a list of entries with fields such as mode, name, and SHA.

**Example Usage:**

```python
tree_entries = read_tree('4b825dc...')
for entry in tree_entries:
    print(f"{entry['mode']} {entry['name']} {entry['sha']}")
```

---

## ASCII Diagram: Git Object Storage and Retrieval

```
+-----------------------------------------------+
|                 .git/objects                   |
|                                               |
|  +----------+     +----------+     +---------+|
|  |  9fceb0  | --> | 02...    |     | (file)  ||
|  +----------+     +----------+     +---------+|
|                                               |
+-----------------------------------------------+
         |                  |                 |
         v                  v                 v
    cat_file()          read_object()      read_tree()
         |                  |                 |
         |   Raw Object Data | Structured Obj  | Parsed Tree Entries
         |                  |                 |
+----------------+   +----------------+  +---------------------+
| Compressed obj  |   | Commit/Blob/   |  | List of tree entries |
| file in object  |-->| Tree object    |  | with mode, name, sha |
| database       |   | representation |  +---------------------+
+----------------+   +----------------+
```

---

This document serves as a foundational reference for developers working with Git internals, offering practical insights into low-level object handling routines critical for implementing or extending Git-related tooling. For further understanding of repository structure and workflows, see [repository_and_object_model.md](./repository_and_object_model.md) and [commit_and_branch.md](./commit_and_branch.md).