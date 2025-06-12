# Status.md - Checking and Displaying Working Copy Status

## Overview

This document describes how to check and display the status of the working copy in a Git repository. It covers identifying files that have been changed, added (new), or deleted relative to the Git index (staging area). This functionality is crucial for users to understand what modifications exist in their working directory before committing changes.

Within the broader documentation tree, `status.md` fits under **Index and Working Copy Management**, alongside related topics like index handling, diffs, and staging. It leverages core functions that interact with the Git index and working directory, providing a user-friendly summary of repository state changes.

---

## Important Functions

### 1. `status()`

#### Purpose

Displays the status of the working copy by printing lists of changed, new, and deleted files. This function provides a high-level overview to the user of which files have modifications compared to the Git index.

#### Parameters

- None

#### Operation Steps

1. Calls `get_status()` to retrieve three lists: changed files, new files, and deleted files.
2. Prints each category of files if they are not empty, neatly listing their paths.

#### Example Usage

```python
status()
```

Expected console output if some files changed, some new, and some deleted:

```
changed files:
    src/main.py
    README.md
new files:
    docs/tutorial.md
deleted files:
    old_script.py
```

---

### 2. `get_status()`

#### Purpose

Computes the current status of the working copy, returning tuples of file paths that are changed, new, or deleted compared to the Git index.

#### Returns

- `changed`: Sorted list of files present both in the working directory and index but whose contents differ.
- `new`: Sorted list of files present in the working directory but not in the index.
- `deleted`: Sorted list of files present in the index but missing from the working directory.

#### Operation Steps

1. Recursively walk the working directory (excluding `.git` directory) to collect all file paths.
2. Normalize paths to use forward slashes and strip leading `./`.
3. Read the Git index entries and map paths to their index entries.
4. Determine:
   - **Changed files:** Paths in both working directory and index where file content hash differs.
   - **New files:** Paths present in working directory but not in index.
   - **Deleted files:** Paths present in index but missing from working directory.
5. Return the sorted lists of these paths.

#### Example Usage

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 3. `read_file(path)`

#### Purpose

Reads the content of a file at the given path and returns its raw bytes.

#### Parameters

- `path` (str): File system path to the file.

#### Returns

- Bytes content of the file.

#### Example Usage

```python
data = read_file("README.md")
print(data.decode())
```

---

### 4. `hash_object(data, obj_type, write=True)`

#### Purpose

Computes the SHA-1 hash of the given data as a Git object of the specified type (e.g., blob, tree, commit). Optionally writes the object to the Git object store.

#### Parameters

- `data` (bytes): Raw bytes of object data.
- `obj_type` (str): Git object type (e.g., "blob").
- `write` (bool): Whether to write the object to the `.git/objects` directory.

#### Returns

- SHA-1 object hash as a hex string.

#### Operation Steps

1. Create the object header: `<obj_type> <len(data)>`
2. Concatenate header, a null byte, and the data.
3. Compute SHA-1 hash of the full data.
4. If `write` is True:
   - Compress the full data using zlib.
   - Store it in `.git/objects/` under directory named by first two hex chars of hash.
5. Return the hex SHA-1 string.

#### Example Usage

```python
data = read_file("README.md")
sha1 = hash_object(data, "blob")
print("Object hash:", sha1)
```

---

### 5. `read_index()`

#### Purpose

Reads the Git index file and returns a list of `IndexEntry` objects representing the staged files and their metadata.

#### Returns

- List of `IndexEntry` instances.

#### Operation Steps

1. Read `.git/index` file.
2. Verify checksum and header signature for validity.
3. Parse each index entry including stat info, SHA-1 hash, flags, and path.
4. Return the list of entries.

#### Example Usage

```python
entries = read_index()
for e in entries:
    print(f"{e.path}: {e.sha1.hex()}")
```

---

### 6. `add(paths)`

#### Purpose

Add given file paths to the Git index, updating or creating entries for these files.

#### Parameters

- `paths` (list of str): File paths to add to the index.

#### Operation Steps

1. Normalize paths to use forward slashes.
2. Read existing index entries.
3. Remove any entries matching given paths (to avoid duplicates).
4. For each path:
   - Read file content and compute blob SHA-1.
   - Stat the file to collect metadata.
   - Create a new `IndexEntry` with file metadata and SHA-1.
5. Sort all entries by path.
6. Write updated index back to `.git/index`.

#### Example Usage

```python
add(["README.md", "src/main.py"])
```

---

### ASCII Diagram: Status Workflow Overview

```
+------------------+
| Working Directory |
+------------------+
         |
         | Collect all file paths (excluding .git)
         v
+------------------+       +-----------------+
|   Git Index      |<----->|  File Hashing   |
+------------------+       +-----------------+
         |                          |
         | Compare hashes           |
         +------------+-------------+
                      |
       +--------------+---------------+
       |              |               |
   Changed files   New files     Deleted files
       |              |               |
       v              v               v
   Display lists of changed, new, and deleted files
```

---

### Additional Notes

- The `status()` function depends heavily on `get_status()`, which performs the core comparison logic.
- The file hashing mechanism (`hash_object`) ensures content integrity and comparison accuracy.
- The index plays a pivotal role as the baseline for comparison.
- This status functionality lays the groundwork for further commands like `git add`, `git commit`, and `git diff`.

---

# Summary

This documentation provides a clear understanding of how the working copy status is determined and displayed. By combining directory traversal, file hashing, and index inspection, the status functionality enables users to effectively track changes before committing or staging. The functions described here are foundational utilities within the larger `pygit` project for repository management.