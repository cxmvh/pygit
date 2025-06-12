# add.md

# Adding files to the git index

---

## Overview

The `add.md` document covers the functionality for adding files to the Git index (also called the staging area). This is a crucial step in the Git workflow where changes in the working directory are marked for inclusion in the next commit. Within the broader documentation tree, this file is part of the **Index Management** section, which includes reading, writing, and managing the index file. The `add` command bridges the working copy and the index by hashing file contents as blobs, creating index entries, and updating the index file to reflect the staged files.

Understanding how files are added to the index is fundamental to grasping how Git tracks changes and prepares commits. The documented `add(paths)` function demonstrates how files are read, hashed, and recorded in the index, ensuring accurate tracking of file versions.

---

## Function Documentation

### `add(paths)`

#### Purpose

The `add` function stages one or more files by adding them to the Git index. It updates the index to include new or modified files so that these changes will be included in the next commit.

#### Parameters

- **paths** (`list[str]`): A list of file paths (strings) to be added to the index. Paths can be relative and use either forward or backward slashes; backward slashes will be converted to forward slashes.

#### Preconditions

- The paths must correspond to files that exist in the working directory.
- The `.git/index` file may or may not exist; if it does, it will be read and updated.
- File system metadata (e.g., modification times, permissions) is accessible for the specified files.

#### Operation Details

1. **Normalize paths:**  
   Converts all file paths to use forward slashes (`/`), ensuring consistency across platforms (important for Windows compatibility).

2. **Read existing index entries:**  
   The function reads the current index entries using `read_index()`. If the index does not exist, it assumes an empty list of entries.

3. **Filter out old entries for given paths:**  
   Any existing index entries corresponding to the files being added are removed to avoid duplicates.

4. **Hash file contents:**  
   For each file path:
   - Read the file contents as bytes.
   - Hash the contents using `hash_object(data, "blob")`, which creates a Git blob object and returns its SHA-1 hash.

5. **Gather file metadata:**  
   Using `os.stat(path)`, it collects metadata such as creation time, modification time, device, inode, mode (permissions), user ID, group ID, and file size.

6. **Create new index entries:**  
   Constructs an `IndexEntry` for each file with the gathered metadata, SHA-1 hash, and a flags field that encodes the path length.

7. **Sort all entries:**  
   The combined index entries are sorted by file path lexicographically to maintain index order.

8. **Write updated index:**  
   The updated list of entries is written back to `.git/index` using `write_index(entries)`.

#### Example Usage

```python
from pygit import add

# Stage a single file
add(['README.md'])

# Stage multiple files
add([
    'src/main.py',
    'docs/add.md',
    'tests/test_add.py'
])
```

---

## Supporting Concepts and Flow

### Interaction with `hash_object`

When adding files, their contents are hashed and stored as blob objects in the Git object store. This ensures the content is tracked independently of the file system.

```
+-----------------+       +-----------------+       +-------------------+
| Working Copy    |  -->  | hash_object()   |  -->  | Git Object Store   |
| (file contents) |       | (blob)          |       | (.git/objects/)    |
+-----------------+       +-----------------+       +-------------------+
```

### Index Entry Structure (Simplified)

Each index entry contains:

- File metadata (ctime, mtime, dev, ino, mode, uid, gid, size)
- SHA-1 hash of the blob object
- Flags (including path length)
- File path

The entries are stored in `.git/index` in a binary format with a checksum footer.

---

## ASCII Diagram: Adding Files to the Index

```
Working Directory
+-------------------+
| file1.txt         |    +------------------+
| file2.py          | -> | Read file content |  -> hash_object(blob)
| README.md         |    +------------------+        |
+-------------------+                                  v
                                                +-------------------+
                                                | Git Object Store   |
                                                +-------------------+

Updated Index:
+------------------------------------------------------------+
| Index Entry 1: file1.txt, sha1=abc123..., metadata          |
| Index Entry 2: file2.py, sha1=def456..., metadata           |
| Index Entry 3: README.md, sha1=789abc..., metadata          |
+------------------------------------------------------------+
              |
              v
          write_index()
              |
         .git/index file updated
```

---

## Related Functions

While this document focuses on `add(paths)`, the following functions are integral to its operation and understanding:

- **`read_index()`**: Reads the current index file entries.
- **`write_index(entries)`**: Writes the updated entries back to the index file.
- **`hash_object(data, obj_type)`**: Hashes file data and creates a Git object.
- **`read_file(path)`**: Reads file content as bytes.

For detailed documentation on these, refer to their respective files in the **Index Management** and **Object Handling** sections.

---

## Summary

The `add` command stages files by reading their contents, hashing them as Git blob objects, and updating the Git index with new entries that record the file metadata and blob SHA-1. This prepares the changes for the next commit and forms the foundation of Git’s version tracking mechanism.