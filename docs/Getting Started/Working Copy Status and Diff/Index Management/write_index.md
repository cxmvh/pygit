# write_index.md

# Writing IndexEntry Objects Back to the Git Index File

## Overview

The `write_index.md` file documents the process and function involved in writing `IndexEntry` objects back to the Git index file (`.git/index`) in the `pygit` repository. The Git index is a critical data structure that tracks the state of the working directory in preparation for commits. Modifying the index properly is essential for staging changes, committing, and synchronizing repository state.

This document fits within the **"Working Copy Status and Diff"** section of the overall pygit documentation tree, under the subdirectory **"Index Management"**. It complements related files such as `read_index.md` (which focuses on reading and interpreting the index) and `add.md` (which covers adding files to the index). The `write_index` function is a pivotal part of the git index handling workflow, used internally by commands like `add()` to persist staged file information.

---

## Function: `write_index(entries)`

### Purpose

`write_index` serializes a list of `IndexEntry` objects and writes them into the `.git/index` file in the Git repository. This operation updates the index file, representing the staging area of the repository. The function ensures the index file conforms to the Git index file format version 2, including correct headers and SHA-1 checksum verification.

### Signature

```python
def write_index(entries):
    """Write list of IndexEntry objects to git index file."""
```

### Parameters

- `entries`: A list of `IndexEntry` objects representing files staged in the index. Each `IndexEntry` contains metadata such as timestamps, device/inode info, file mode, user/group IDs, file size, object SHA-1, flags, and the file path.

### Description

The function performs the following steps:

1. **Packing Entries**:  
   Each `IndexEntry` is packed into a binary format according to the Git index specification. This includes fixed-size fields (timestamps, device, inode, mode, uid, gid, size, SHA-1, flags) followed by the variable-length path string. The function calculates padding so that each entry is aligned to an 8-byte boundary.

2. **Constructing Index Header**:  
   The index file starts with a 12-byte header consisting of:
   - Signature: `b'DIRC'` (4 bytes)
   - Version number: 2 (4 bytes)
   - Number of entries (4 bytes)

3. **Concatenating Entries**:  
   The packed entries are concatenated after the header.

4. **SHA-1 Checksum**:  
   A SHA-1 checksum is computed over the entire index file contents excluding the checksum itself. This checksum is appended at the end of the file to validate integrity.

5. **Writing to Disk**:  
   The complete index data (header + entries + checksum) is written atomically to `.git/index` using the utility `write_file`.

### Example Usage

```python
# Assume `entries` is a list of IndexEntry objects obtained by reading the current index,
# modifying or adding entries as needed, and then writing them back.

from pygit import write_index, read_index

# Read current index entries
entries = read_index()

# Modify entries as needed (e.g., add new or updated files)
# entries.append(new_entry)
# entries = sorted(entries, key=lambda e: e.path)

# Write updated index back to disk
write_index(entries)
```

### Code Snippet

```python
def write_index(entries):
    """Write list of IndexEntry objects to git index file."""
    packed_entries = []
    for entry in entries:
        # Pack fixed fields according to Git index format
        entry_head = struct.pack('!LLLLLLLLLL20sH',
                entry.ctime_s, entry.ctime_n, entry.mtime_s, entry.mtime_n,
                entry.dev, entry.ino, entry.mode, entry.uid, entry.gid,
                entry.size, entry.sha1, entry.flags)
        path = entry.path.encode()
        # Align entry length to next multiple of 8 bytes
        length = ((62 + len(path) + 8) // 8) * 8
        packed_entry = entry_head + path + b'\x00' * (length - 62 - len(path))
        packed_entries.append(packed_entry)
    # Construct header: signature, version, number of entries
    header = struct.pack('!4sLL', b'DIRC', 2, len(entries))
    all_data = header + b''.join(packed_entries)
    # Append SHA-1 checksum of all preceding bytes
    digest = hashlib.sha1(all_data).digest()
    # Write to .git/index file
    write_file(os.path.join('.git', 'index'), all_data + digest)
```

---

## Additional Context and Related Functions

- `read_index()`: Reads `.git/index` and returns a list of `IndexEntry` objects.
- `add(paths)`: Adds files to the index by creating new `IndexEntry` objects and calling `write_index`.
- `write_file(path, data)`: Utility function to write bytes data to a file.
- `hash_object(data, obj_type)`: Computes SHA-1 hash of object data and optionally writes it to the object store.

### Example Workflow Diagram

```
+-------------------+
| Read current index | <--- read_index()
+-------------------+
          |
          v
+-------------------+
| Modify entries or  |
| add new IndexEntry |
+-------------------+
          |
          v
+-------------------+
| Serialize entries  | <--- write_index()
+-------------------+
          |
          v
+-------------------+
| Write .git/index   | <--- write_file()
+-------------------+
```

---

## Summary

The `write_index` function is essential for persisting the staged state of files in a Git repository managed by `pygit`. By correctly formatting and checksumming the index file, it ensures that staged changes are stored reliably, enabling subsequent operations like committing or diffing to function correctly. This documentation provides both the conceptual understanding and practical details necessary to use or extend this functionality.