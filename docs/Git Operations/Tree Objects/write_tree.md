# write_tree.md

# Writing Tree Objects from Index Entries

---

## Overview

The `write_tree.md` document details the process of creating Git tree objects from the current Git index entries within the `pygit` project. This functionality is fundamental to Git’s internal storage mechanism, enabling the representation of directory contents (file modes, paths, and SHA-1 hashes) as tree objects.

In the overall documentation hierarchy, this file belongs to the **"Tree Objects"** section under **"Tree and Commit Object Management"**. It provides the technical explanation and usage of the `write_tree()` function, which serializes the current state of the Git index into a tree object and stores it in the object database. This process is critical as it links the file snapshots captured in the index to a Git commit.

---

## Function Documentation

### `write_tree()`

**Purpose:**  
Generate and write a tree object representing the current contents of the Git index. This function reads index entries (which represent files staged for commit), formats them according to Git's tree object specification, and writes the serialized tree object into the Git object store. It returns the SHA-1 hash of the newly created tree object.

**Parameters:**  
- None.

**Returns:**  
- `str`: The SHA-1 hash (hexadecimal string) of the written tree object.

**Preconditions:**  
- The Git index file (`.git/index`) must exist and contain valid `IndexEntry` objects.
- Currently, the function only supports a flat directory structure — i.e., no nested directories in file paths.

**Detailed Operation:**  
1. Call `read_index()` to obtain the current list of `IndexEntry` objects. Each entry corresponds to a staged file with associated metadata including mode (permissions), path, and SHA-1 hash of the blob content.
2. For each index entry:
   - Assert that the path does not contain `/` (no subdirectories supported).
   - Format a tree entry string as `"<mode as octal> <path>\0<sha1 bytes>"`.
     - The mode is formatted in octal notation (e.g., `100644` for normal files).
     - The SHA-1 is included as raw bytes (20 bytes).
3. Concatenate all tree entries into a single bytes object.
4. Pass the concatenated entries to `hash_object()` with the type `'tree'`, which:
   - Prepares the tree object header.
   - Computes the SHA-1 hash.
   - Compresses and stores the object in the `.git/objects` directory.
5. Return the SHA-1 hash of the stored tree object.

**Example Usage:**

```python
from pygit import write_tree

tree_sha1 = write_tree()
print(f"Tree object written with SHA-1: {tree_sha1}")
```

**Notes:**  
- This function is typically called during the commit process to snapshot the current index state.
- Future enhancements may include recursive support for nested directories, constructing a tree hierarchy.

---

## ASCII Diagram: Tree Object Structure in Git

```
Current Index Entries
+---------------------------+
| Entry 1: mode, path, sha1 |
| Entry 2: mode, path, sha1 |
| Entry 3: mode, path, sha1 |
| ...                       |
+---------------------------+
          |
          v
Concatenate entries as:
[mode path\0sha1][mode path\0sha1][...]
          |
          v
Create tree object:
+----------------------------------------+
| Header: "tree <size>\0"                 |
| Data: concatenated entries              |
+----------------------------------------+
          |
          v
Compute SHA-1 hash and store compressed object in .git/objects
          |
          v
Return SHA-1 hash of tree object
```

---

## Related Functions (Contextual Reference)

- `read_index()` — Reads and parses the Git index file, returning a list of `IndexEntry` objects.
- `hash_object(data, obj_type)` — Hashes and stores Git objects, including trees.
- `commit()` — Calls `write_tree()` internally to record the tree object for a new commit.
- `read_tree()` — Parses a tree object to extract its entries (inverse operation).

---

## Summary

The `write_tree()` function is an essential utility in pygit that translates the staged snapshot of files (index entries) into a Git tree object. It serves as the bridge between the index and commits by producing the tree structure Git requires to track directory contents efficiently. This document provides a clear explanation and example to help developers understand and utilize this function within the pygit project.