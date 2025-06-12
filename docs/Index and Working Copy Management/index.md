# index.md - Git Index Management

## Overview

This document covers the reading, writing, and manipulation of the Git index file. The Git index (also called the staging area) is a critical component in Git's architecture that tracks changes intended for the next commit. Managing the index involves reading its current contents, adding files to it, and writing changes back to disk. Within the broader documentation tree, this file fits under **Index and Working Copy Management**, closely related to repository status reporting, committing changes, and showing diffs. It plays a key role in bridging the working directory and the commit history by staging snapshots of file contents.

---

## Function Documentation

### `read_index()`

**Purpose:**  
Reads the current Git index file from `.git/index` and returns a list of `IndexEntry` objects representing each staged file.

**Parameters:**  
None

**Returns:**  
`List[IndexEntry]` — a list of index entries describing each staged file.

**Operation:**  
1. Attempts to read the binary index file `.git/index`.
2. Validates the SHA-1 checksum at the end of the file to ensure data integrity.
3. Parses the header, verifying the signature (`DIRC`) and version (expected 2).
4. Iterates through the entries, unpacking fixed-size fields and reading the variable-length file path.
5. Returns all parsed entries.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(f"Staged file: {entry.path}, SHA1: {entry.sha1.hex()}")
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the `.git/index` file, updating the staging area.

**Parameters:**  
- `entries: List[IndexEntry]` — The list of entries to write.

**Returns:**  
None

**Operation:**  
1. Packs each entry's metadata and path into the Git index binary format.
2. Computes a SHA-1 checksum over the entire data.
3. Writes the packed data plus checksum to `.git/index`.

**Example Usage:**
```python
entries = read_index()
# Modify entries as needed, for example, remove one path
entries = [e for e in entries if e.path != "unwanted_file.txt"]
write_index(entries)
```

---

### `add(paths)`

**Purpose:**  
Add one or more files to the Git index, staging them for the next commit.

**Parameters:**  
- `paths: List[str]` — List of file paths to add.

**Returns:**  
None

**Operation:**  
1. Normalizes file paths to use forward slashes.
2. Reads current index entries, excluding those matching new paths (to replace them).
3. For each path to add:
   - Reads file contents.
   - Creates a blob object from the contents and obtains its SHA-1 hash.
   - Retrieves file metadata (timestamps, permissions, size, etc.).
   - Creates a new `IndexEntry`.
4. Adds new entries to the existing ones.
5. Sorts the entries by path.
6. Writes the updated entries back to the index.

**Example Usage:**
```python
add(["src/main.py", "README.md"])
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of a git object (blob, tree, commit) from the provided data and optionally write it to the object store.

**Parameters:**  
- `data: bytes` — Raw object data.
- `obj_type: str` — Type of object (`blob`, `tree`, `commit`).
- `write: bool` — Whether to write the compressed object to `.git/objects/`.

**Returns:**  
`str` — Hexadecimal SHA-1 hash of the object.

**Operation:**  
1. Constructs the object header (`obj_type size\0`).
2. Concatenates header and data.
3. Computes SHA-1 hash of the full object.
4. If `write` is True, compresses and writes the object to the correct path in the `.git/objects` directory, creating directories if necessary.
5. Returns the SHA-1 hash.

**Example Usage:**
```python
data = b"hello world\n"
sha1 = hash_object(data, "blob")
print(f"Created blob object with SHA1: {sha1}")
```

---

### `read_file(path)`

**Purpose:**  
Read the contents of a file as bytes.

**Parameters:**  
- `path: str` — File path.

**Returns:**  
`bytes` — Contents of the file.

**Operation:**  
Opens the file in binary mode and reads all bytes.

**Example Usage:**
```python
contents = read_file("README.md")
print(contents.decode())
```

---

### `write_file(path, data)`

**Purpose:**  
Write bytes to a file at the specified path.

**Parameters:**  
- `path: str` — Destination file path.
- `data: bytes` — Data to write.

**Returns:**  
None

**Operation:**  
Opens the file in binary write mode and writes the data.

**Example Usage:**
```python
write_file(".git/HEAD", b"ref: refs/heads/master\n")
```

---

### `status()`

**Purpose:**  
Display the status of the working directory with respect to the index.

**Parameters:**  
None

**Returns:**  
None

**Operation:**  
1. Calls `get_status()` to retrieve lists of changed, new, and deleted files.
2. Prints these files categorized by their status.

**Example Usage:**
```python
status()
# Output:
# changed files:
#     src/main.py
# new files:
#     docs/usage.md
# deleted files:
#     old_notes.txt
```

---

### `get_status()`

**Purpose:**  
Get the status of the working directory compared to the index.

**Parameters:**  
None

**Returns:**  
Tuple[List[str], List[str], List[str]] — Lists of changed, new, and deleted file paths.

**Operation:**  
1. Walks the working directory (excluding `.git`).
2. Builds sets of paths in the working directory and index.
3. Determines changed files by comparing object hashes.
4. Identifies new files existing in the working directory but not in the index.
5. Identifies deleted files missing from the working directory but present in the index.

**Example Usage:**
```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### `diff()`

**Purpose:**  
Show the diff of files changed between the index and the working copy.

**Parameters:**  
None

**Returns:**  
None

**Operation:**  
1. Identifies changed files via `get_status()`.
2. For each changed file:
   - Reads the blob object from the index.
   - Reads the current file content in the working directory.
   - Uses `difflib.unified_diff` to generate the diff.
3. Prints the unified diff.

**Example Usage:**
```python
diff()
# Output:
# --- src/main.py (index)
# +++ src/main.py (working copy)
# @@ -1,4 +1,5 @@
# -print("Hello World")
# +print("Hello, World!")
# +print("Additional line")
```

---

## ASCII Diagram: Git Index Role

```
+-----------------+           +--------------------+            +------------------+
| Working Directory| --read--> |       Git Index    | --commit--> |  Git Object Store |
+-----------------+           +--------------------+            +------------------+
       |                             ^    |                                ^
       |                             |    |                                |
       |                             +----+                                |
       |                               add files                            |
       +----------------------------------------------------->              |
```

---

## Notes

- The index file format is binary and includes metadata such as timestamps, permissions, and SHA-1 hashes.
- Adding files to the index involves creating blob objects and updating the index entries.
- The index acts as the staging area for commits.
- Integrity checks (e.g., SHA-1 checksum validation) are critical when reading and writing the index.
- The current implementation supports only top-level directory entries (no nested directories in the index).

---

This concludes the technical reference for managing the Git index file within the repository. For related operations, refer to the `status.md` for working copy status, `commit.md` for committing staged changes, and `diff.md` for file differences.