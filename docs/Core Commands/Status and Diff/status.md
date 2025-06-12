# status.md - Working Copy Status Detection and Diff Display

---

## Overview

This document covers the mechanisms and commands related to detecting the status of files in the working copy relative to the Git index and repository, as well as displaying diffs of changed files. It provides detailed explanations of status commands and associated functions that identify changed, new, or deleted files, and show differences between the index and working copy. This file is part of the **Status and Diff** section in the documentation tree, which focuses on working copy status, index manipulation, and diffing. The concepts here are integral to understanding how changes are tracked and presented to the user before committing or pushing.

---

## Functions

### 1. `get_status()`

#### Purpose

Detects and returns the current state of the working copy compared to the Git index. It identifies files that have been changed, newly created, or deleted since the last index update.

#### Parameters

- None

#### Returns

- `changed_paths` (list of `str`): Files that exist both in the index and working copy but whose contents differ.
- `new_paths` (list of `str`): Files that exist in the working copy but are not yet tracked in the index.
- `deleted_paths` (list of `str`): Files that are tracked in the index but no longer exist in the working copy.

#### Operation Steps

1. Recursively walks the current directory tree, excluding the `.git` directory, to collect all file paths in the working copy.
2. Reads the Git index to get a list of tracked files and their metadata.
3. Compares the set of working copy file paths and index paths:
   - Files present in both sets are checked for content differences by hashing the working copy file and comparing it to the SHA-1 hash stored in the index.
   - Files present only in the working copy are considered new.
   - Files present only in the index but missing in the working copy are considered deleted.
4. Returns sorted lists of changed, new, and deleted file paths.

#### Example Usage

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 2. `status()`

#### Purpose

Prints a user-friendly summary of the working copy status, displaying lists of changed, new, and deleted files.

#### Parameters

- None

#### Returns

- None (prints output to standard output)

#### Operation Steps

1. Calls `get_status()` to retrieve changed, new, and deleted files.
2. Prints each category with appropriate headers if any files exist in that category.
3. Indents listed files for readability.

#### Example Usage

```python
status()
```

**Sample Output**

```
changed files:
    src/main.py
new files:
    docs/README.md
deleted files:
    old_script.py
```

---

### 3. `diff()`

#### Purpose

Displays line-by-line unified diffs between the contents of files in the index and their counterparts in the working copy for all changed files.

#### Parameters

- None

#### Returns

- None (prints diffs to standard output)

#### Operation Steps

1. Calls `get_status()` to get the list of changed files.
2. Reads the index entries for quick lookup of SHA-1 hashes by file path.
3. For each changed file:
   - Reads the blob object from the Git object store using the SHA-1 from the index.
   - Decodes the blob data into lines representing the index version of the file.
   - Reads the current working copy file and splits into lines.
   - Uses Python's `difflib.unified_diff` to generate a diff between index and working copy file contents.
   - Prints the unified diff output.
4. Prints a separator line of 70 hyphens between diffs of different files for clarity.

#### Example Usage

```python
diff()
```

**Sample Output**

```
--- src/main.py (index)
+++ src/main.py (working copy)
@@ -1,5 +1,6 @@
 def main():
-    print("Hello, World!")
+    print("Hello, Universe!")
+
+    print("Additional line")
----------------------------------------------------------------------
```

---

### 4. `add(paths)`

#### Purpose

Adds specified file paths from the working copy to the Git index, updating the index entries with current file metadata and content hashes.

#### Parameters

- `paths` (list of `str`): File paths to add to the index.

#### Returns

- None

#### Operation Steps

1. Normalizes paths by converting backslashes to forward slashes.
2. Reads the current index entries.
3. Removes any existing entries corresponding to the paths to add (to avoid duplicates).
4. For each path:
   - Reads the file content and hashes it as a blob object.
   - Retrieves filesystem metadata (like timestamps, device, inode, mode, owner, size).
   - Creates a new `IndexEntry` with this information.
5. Adds new entries to the list.
6. Sorts entries by path.
7. Writes updated entries back to the index file.

#### Example Usage

```python
files_to_add = ['src/main.py', 'docs/README.md']
add(files_to_add)
```

---

### 5. `read_index()`

#### Purpose

Reads the Git index file `.git/index` and parses it into a list of `IndexEntry` objects representing tracked files and their metadata.

#### Parameters

- None

#### Returns

- List of `IndexEntry` objects

#### Operation Steps

1. Reads the index file as bytes.
2. Validates the checksum and header signature.
3. Iterates over the index entry data section:
   - Unpacks fixed-size fields with `struct`.
   - Extracts the variable-length file path.
   - Constructs `IndexEntry` objects.
4. Returns the list of entries.

#### Example Usage

```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### 6. `read_object(sha1_prefix)`

#### Purpose

Retrieves and decompresses a Git object from the object store using a SHA-1 hash prefix, returning the object type and raw data.

#### Parameters

- `sha1_prefix` (str): Hexadecimal prefix of the SHA-1 hash of the object (must be at least 2 characters).

#### Returns

- Tuple `(obj_type, data_bytes)` where:
  - `obj_type` (str): Type of the Git object (`'blob'`, `'tree'`, `'commit'`, etc.)
  - `data_bytes` (bytes): Raw content bytes of the object.

#### Operation Steps

1. Uses `find_object()` to locate the object file path in `.git/objects`.
2. Reads and decompresses the object file.
3. Parses the object header (type and size).
4. Verifies the size matches the decompressed data length.
5. Returns the object type and data.

#### Example Usage

```python
obj_type, data = read_object('a1b2c3')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### 7. `read_file(path)`

#### Purpose

Reads the entire contents of a file at the given path as bytes.

#### Parameters

- `path` (str): File system path to read.

#### Returns

- Bytes content of the file.

#### Operation Steps

1. Opens the file in binary read mode.
2. Reads all bytes.
3. Closes the file and returns the bytes.

#### Example Usage

```python
content = read_file('src/main.py')
print(content.decode())
```

---

### 8. `write_index(entries)`

#### Purpose

Writes a list of `IndexEntry` objects back to the Git index file, including recalculating the checksum.

#### Parameters

- `entries` (list of `IndexEntry`): Index entries to write.

#### Returns

- None

#### Operation Steps

1. Packs each entry into binary format with fixed fields and path strings.
2. Constructs the index file header.
3. Concatenates all entries and appends the SHA-1 checksum.
4. Writes the resulting binary data to `.git/index`.

#### Example Usage

```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### 9. `ls_files(details=False)`

#### Purpose

Lists files currently tracked in the Git index. Optionally displays detailed information.

#### Parameters

- `details` (bool): If `True`, prints mode, SHA-1, stage number, and path. Otherwise, prints only paths.

#### Returns

- None (prints to stdout)

#### Operation Steps

1. Reads index entries.
2. For each entry:
   - If `details` is `True`, prints mode, SHA-1, stage, and path.
   - Otherwise, prints only the path.

#### Example Usage

```python
ls_files(details=True)
```

**Sample Output**

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	src/main.py
100644 4b825dc642cb6eb9a060e54bf8d69288fbee4904 0	README.md
```

---

### 10. `cat_file(mode, sha1_prefix)`

#### Purpose

Displays information or content of a Git object specified by SHA-1 prefix, in various modes:

- Raw content (`commit`, `tree`, `blob`)
- Size
- Type
- Pretty-printed representation

#### Parameters

- `mode` (str): Mode of output: `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, `'pretty'`.
- `sha1_prefix` (str): SHA-1 prefix identifying the object.

#### Returns

- None (prints to stdout or writes to buffer)

#### Operation Steps

1. Reads the object via `read_object`.
2. Depending on `mode`:
   - If raw type (`commit`, `tree`, `blob`), verifies object type and writes raw data.
   - If `size`, prints size of data.
   - If `type`, prints object type.
   - If `pretty`:
     - For `commit` and `blob`, prints raw data.
     - For `tree`, iterates entries and prints human-readable lines.
3. Raises error on invalid mode.

#### Example Usage

```python
cat_file('pretty', 'a1b2c3')
```

---

## ASCII Diagram: Status and Diff Workflow

```
+--------------------+       +-------------------+      +-------------------+
|  Working Copy Files |       |      Git Index     |      |    Object Store    |
|  (Filesystem)       |       | (.git/index file)  |      |  (.git/objects/)   |
+----------+---------+       +---------+---------+      +---------+---------+
           |                           |                          |
           |                           |                          |
           |    get_status() compares file states               |
           |<----------------------------------------------------|
           |                           |                          |
           |                           |                          |
           |      status() prints categorized file lists         |
           |                           |                          |
           |                           |                          |
           |                           |                          |
           | diff() reads index blobs and working copy files     |
           |---------------------------------------------------->|
           |                           |                          |
           |          shows unified diff output                   |
           |<----------------------------------------------------|
           |                           |                          |
           +-----------------------------------------------------+
```

---

## Summary

This document outlines the core functionality for detecting changes in the working copy relative to the Git index and displaying those changes to the user. Key functions like `get_status()`, `status()`, and `diff()` form the backbone of status detection and display, while supporting functions (`read_index()`, `add()`, `read_object()`, etc.) manage index and object store interactions. Together, these provide a foundation for the user to understand and visualize their repository's current state before committing or pushing changes.