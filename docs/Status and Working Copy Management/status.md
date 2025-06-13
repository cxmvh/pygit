# status.md

# Status Command and Related Functions Documentation

---

## Overview

The `status.md` document provides comprehensive documentation for the `status` command and its supporting functions within the `pygit` project. Located under the **Status and Working Copy Management** section of the overall documentation tree, this file explains how `pygit` inspects the current working directory to report changes relative to the Git index. The key functions covered include `status()`, which displays the working copy status summary, and auxiliary functions such as `get_status()`, `read_file()`, and `hash_object()` that underpin its operation. Understanding these functions is crucial for grasping how `pygit` tracks file changes, new files, and deletions in the working directory.

---

## Function Documentation

---

### Function: `status()`

#### Purpose

The `status()` function provides a user-friendly summary of the working copy's current state. It identifies files that have been changed, newly added, or deleted since the last index update, and prints these lists with clear headings.

#### Parameters

- None

#### Operation Details

1. Calls `get_status()` to retrieve three lists:
   - `changed`: files modified since last commit/index.
   - `new`: files newly added but not yet tracked.
   - `deleted`: files tracked but now missing from working directory.
2. Prints each category if non-empty:
   - "changed files:" followed by each changed file path.
   - "new files:" followed by each new file path.
   - "deleted files:" followed by each deleted file path.

#### Usage Example

```python
>>> status()
changed files:
    README.md
    src/main.py
new files:
    docs/usage.md
deleted files:
    old_script.py
```

---

### Function: `get_status()`

#### Purpose

The `get_status()` function computes the working directory status by comparing the current files on disk against the Git index entries. It returns three sorted lists of file paths representing changed, new, and deleted files.

#### Parameters

- None

#### Operation Details

1. Recursively walks the current directory excluding `.git` to collect all file paths (`paths`).
2. Reads Git index entries with `read_index()` and creates a map `entries_by_path`.
3. Determines:
   - `changed`: intersection of working directory and index files where blob hashes differ.
   - `new`: files present in working directory but not in index.
   - `deleted`: files present in index but missing from working directory.
4. Returns these as sorted lists.

#### Usage Example

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### Function: `read_file(path)`

#### Purpose

Reads and returns the contents of a file at the specified path as bytes.

#### Parameters

- `path` (str): The file system path to read.

#### Operation Details

- Opens the file in binary mode.
- Reads all data and returns it as bytes.

#### Usage Example

```python
data = read_file('README.md')
print(data.decode())
```

---

### Function: `hash_object(data, obj_type, write=True)`

#### Purpose

Computes the SHA-1 hash of a Git object (blob, tree, commit) given its raw data and type. Optionally writes the compressed object data into the Git object store.

#### Parameters

- `data` (bytes): Raw content of the object.
- `obj_type` (str): Type of object (`'blob'`, `'tree'`, `'commit'`).
- `write` (bool, optional): Whether to write the object to the `.git/objects` directory. Defaults to `True`.

#### Operation Details

1. Constructs the Git object header in the format: `<obj_type> <size>\0`.
2. Concatenates header and data.
3. Calculates SHA-1 hash of the combined data.
4. If `write` is `True`, compresses and writes the object to `.git/objects/<first_two_sha1>/<remaining_sha1>`.
5. Returns the SHA-1 hash as a hexadecimal string.

#### Usage Example

```python
content = read_file('README.md')
sha1 = hash_object(content, 'blob')
print(f"Blob SHA-1: {sha1}")
```

---

## ASCII Diagram: Status Checking Workflow

```
+----------------+
| Working Copy   |
| (filesystem)   |
+----------------+
        |
        | read_file()
        |
        v
+----------------+
| File Contents  |
+----------------+
        |
        | hash_object(data, 'blob')
        |
        v
+----------------+
| Object Store   |
| (.git/objects) |
+----------------+

Meanwhile:

+----------------+
| Git Index      |
| (read_index()) |
+----------------+

       | Compare hashes
       |
       v

+----------------------------------------+
| get_status() returns (changed, new, del)|
+----------------------------------------+

       |
       v

+----------------+
| status() prints|
+----------------+
```

---

# Summary

This documentation file serves as a detailed technical reference for the `status` command and its core supporting functions within the `pygit` system. It explains how `pygit` efficiently identifies changes in the working directory against the Git index by hashing file contents and comparing object hashes. Together, these functions form the basis for status reporting, a fundamental Git operation that enables users to track repository changes before committing.