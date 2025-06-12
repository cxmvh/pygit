# write_tree.md

# Writing Tree Objects from Index Entries

---

## Overview

This document describes the process and functionality of writing Git tree objects from the current index entries. In a Git repository, tree objects represent directories and contain references to blobs (file contents) or other trees (subdirectories). The `write_tree` function plays a crucial role in translating the state of the Git index—a staging area for changes—into a tree object that can be committed. This file fits into the broader repository documentation tree under **Object Handling and Git Objects**, specifically focusing on how tree objects are created from index entries as part of commit creation workflows.

The main relevance of this functionality lies in its role during commit operations, where the directory state must be captured and stored efficiently as a tree object. This document provides a detailed breakdown of the key function `write_tree()`, its operation, usage, and its relationship with the index and Git objects.

---

## Function Documentation

### `write_tree()`

---

#### Purpose

The `write_tree()` function generates a Git tree object from the current index entries. This tree object captures the state of the repository's tracked files at a given point in time, representing them as a directory listing with mode, filename, and SHA-1 hash references to blob objects (file contents).

This function currently supports only a flat directory structure (no nested subdirectories), i.e., it assumes all index entries are files in the top-level directory.

---

#### Parameters

- None

---

#### Return Value

- Returns the SHA-1 hash (hexadecimal string) of the newly written tree object.

---

#### Preconditions

- The Git index must exist and be readable.
- All files referenced by the index entries must have valid SHA-1 blob hashes.
- Index entries should not include files in subdirectories (no path separators `/` in `entry.path`).

---

#### Detailed Operation

1. **Read the Index Entries**  
   The function calls `read_index()` to obtain a list of `IndexEntry` objects representing the currently staged files.

2. **Process Each Index Entry**  
   For each entry in the index:
   - It checks that the path does not contain a slash (`'/'`) to enforce single-level directory support.
   - It formats the entry into the Git tree entry format:  
     ```
     <file_mode> <file_name>\0<20-byte SHA-1 binary>
     ```
     where:
     - `<file_mode>` is the file mode in octal notation (e.g., `100644` for a regular file).
     - `<file_name>` is the filename (path).
     - The null byte (`\0`) separates the mode and file name from the SHA-1.
     - The SHA-1 is the 20-byte binary representation of the object's hash (not hex).

3. **Concatenate All Tree Entries**  
   All formatted entries are concatenated together into a single byte string representing the content of the tree object.

4. **Hash and Store the Tree Object**  
   The concatenated byte string is passed to `hash_object(data, 'tree')`, which:
   - Prepends the Git object header (`tree <size>\0`),
   - Computes the SHA-1 hash,
   - Compresses and writes the object under `.git/objects/` if it does not already exist.
   
5. **Return the SHA-1 Hash**  
   The function returns the SHA-1 hash string of the created tree object.

---

#### Example Usage

```python
from pygit import write_tree

# Write the current index as a tree object and get its SHA-1 hash
tree_sha1 = write_tree()

print(f"Created tree object with SHA-1: {tree_sha1}")
```

This example assumes the index is already populated with entries (e.g., after staging files with `add()`), and it writes a tree object representing those entries.

---

#### ASCII Diagram: Tree Object Structure

```
+----------------------------------------------+
| Tree Object Content                           |
+----------------------------------------------+
| <mode> <filename>\0<20-byte SHA-1 binary>   |  <- Entry 1 (e.g., a file)
| <mode> <filename>\0<20-byte SHA-1 binary>   |  <- Entry 2
| ...                                          |
+----------------------------------------------+

Stored as a Git object with header:

  "tree <content_size>\0" + <content_bytes>

Hash computed over this whole data blob.
```

This illustrates how each entry is encoded and concatenated within the tree object's data.

---

## Related Functions and Workflow Context

- **`read_index()`**: Reads the index file and returns all staged entries.
- **`hash_object(data, obj_type)`**: Creates and stores a Git object by hashing and compressing data.
- **`commit()`**: Uses `write_tree()` to create a tree object from the index before making a commit.
- **`read_tree()`**: Parses a tree object back into its entries for inspection or traversal.

---

## Summary

The `write_tree()` function is a core utility that converts the staged file states recorded in the Git index into a tree object. This object represents a snapshot of the directory contents at commit time. Understanding and using `write_tree()` is essential for anyone implementing or extending Git-like functionality in Python, particularly for commit creation and tree management workflows.