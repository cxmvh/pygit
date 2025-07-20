---
sidebar_position: 3
---

# Git Index Operations

## Overview

This document provides a comprehensive guide to Git index operations within the pygit project. It covers the reading and writing of the Git index file, the structure of index entries, hashing objects, locating objects within the object store, and listing tracked files with detailed output options. Positioned under the "Core Git Features" section, this file complements other modules such as repository management, object handling, and status operations, offering developers a detailed reference for managing and interacting with the Git index.

---

## Reading the Index File

### Purpose

The function to read the Git index file parses the `.git/index` file and loads its contents into memory as structured index entries. This allows pygit to track the state of files staged for commit.

### How It Works

1. Open and read the binary index file from the `.git` directory.
2. Verify the index file signature and version for compatibility.
3. Read the number of entries stored in the index.
4. For each entry:
   - Parse the metadata fields (ctime, mtime, device, inode, mode, uid, gid, file size).
   - Extract the SHA-1 object hash.
   - Read the file path name.
5. Store entries in a list or dictionary for efficient access.

### Example Usage

```python
index_entries = read_index('.git/index')
for entry in index_entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

## Writing the Index File

### Purpose

The write operation serializes the in-memory index entries back into the `.git/index` file, updating the Git index state on disk.

### How It Works

1. Sort index entries by path name to maintain Git ordering.
2. Serialize each index entry's metadata and object hash into binary format.
3. Write the index header including signature, version, and entry count.
4. Append serialized entries.
5. Write an SHA-1 checksum of the entire index file content to ensure integrity.

### Example Usage

```python
write_index('.git/index', index_entries)
```

---

## Index Entry Structure

### Purpose

The index entry structure defines how each tracked file's data is stored within the index, encompassing file system metadata and object information.

### Structure Fields

- **ctime (creation time):** Timestamp of last inode change.
- **mtime (modification time):** Timestamp of last file modification.
- **dev:** Device number.
- **ino:** Inode number.
- **mode:** File mode and permissions.
- **uid:** User ID of file owner.
- **gid:** Group ID of file owner.
- **file_size:** Size of the file in bytes.
- **sha1:** SHA-1 hash of the file contents.
- **flags:** Miscellaneous flags including path length.
- **path:** Relative file path.

### ASCII Diagram of Index Entry Layout

```
+----------------+----------------+----------------+
|  ctime (8 B)   |  mtime (8 B)   |   dev (4 B)    |
+----------------+----------------+----------------+
|   ino (4 B)    |  mode (4 B)    |    uid (4 B)   |
+----------------+----------------+----------------+
|    gid (4 B)   | file_size (4 B)|   sha1 (20 B)  |
+----------------+----------------+----------------+
|      flags (2 B)         |     path (variable length)    |
+--------------------------+-------------------------------+
```

---

## Hashing Objects

### Purpose

Hashing objects involves calculating the SHA-1 hash of an object's content combined with its Git object header. This hash uniquely identifies objects such as blobs, trees, and commits.

### How It Works

1. Prepare the object header: `<type> <size>\0`.
2. Concatenate the header with the object's raw content.
3. Compute the SHA-1 hash of the concatenated data.
4. Return the computed SHA-1 digest.

### Example Usage

```python
sha1_hash = hash_object(b'blob', file_content)
print(f"Object SHA-1: {sha1_hash.hex()}")
```

---

## Locating Objects in the Object Store

### Purpose

Locating objects involves finding the stored Git object by its SHA-1 hash within the `.git/objects` directory structure.

### How It Works

1. Split the SHA-1 hash into a two-character directory prefix and the remaining 38 characters as the filename.
2. Construct the path `.git/objects/xx/yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy`.
3. Check if the object file exists.
4. If found, decompress and read the object data.

### ASCII Diagram of Object Store Layout

```
.git/objects/
  ├── xx/                 # First two hex chars of SHA-1
      ├── yyyyyyyyyyyyyyy # Remaining 38 chars as filename
```

### Example Usage

```python
obj_data = find_object('.git/objects', sha1_hash)
if obj_data:
    print("Object found and loaded.")
else:
    print("Object not found.")
```

---

## Listing Tracked Files with Detailed Output

### Purpose

This function lists all files tracked by Git, optionally displaying detailed information such as stage numbers, object hashes, and pathnames, similar to `git ls-files`.

### How It Works

1. Read the index file to retrieve all entries.
2. For each entry:
   - Extract relevant metadata.
   - Format output depending on detailed flags (e.g., show stage, SHA-1, file mode).
3. Output the list to the user.

### Example Usage

```python
ls_files(detailed=True)
# Output:
# 100644 9a4e3b2... 0   src/main.py
# 100755 1f2d3c4... 0   bin/script.sh
```

---

This documentation provides a detailed reference for developers working with the Git index in pygit, facilitating understanding and manipulation of Git index internals and workflows. For related topics such as repository management, object handling, and status checks, refer to the adjacent documentation files within the "Core Git Features" section.