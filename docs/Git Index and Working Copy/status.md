# Working Copy Status Functions Documentation

This document describes the working copy status functions of the `pygit` repository. It covers how the system detects changed, new, and deleted files in the working directory relative to the Git index and provides tools to inspect and display these status changes. This file fits under the **"Git Index and Working Copy"** section of the documentation tree, alongside related files like `index.md` and `diff.md`. It is significant because understanding the working copy status is fundamental for managing changes during the development process, enabling users to see what files have been modified, added, or removed before committing or pushing changes.

---

## Overview

The status subsystem provides functionality to:

- Scan the working directory to detect files that have changed compared to the Git index.
- Identify newly added files that are not yet tracked.
- Detect deleted files that were tracked but no longer exist in the working directory.
- Present these changes clearly to the user.
- Provide diff views of changes between the index and the working copy.

The primary interfaces include `get_status()` for retrieving file lists by type of change and `status()` for printing a summary. Additional support functions include reading the index, hashing file contents, and comparing stored blobs with working files.

---

# Function Documentation

---

### `get_status()`

**Purpose:**  
Determines the status of the working copy by comparing files in the working directory against the Git index. Returns three lists of file paths categorized as changed, new, or deleted.

**Parameters:**  
None.

**Returns:**  
A tuple of three lists `(changed_paths, new_paths, deleted_paths)`, each list sorted alphabetically.

**Operation Details:**  
1. Walks the working directory excluding `.git` subdirectory, collecting all file paths.
2. Reads the Git index entries and maps them by file path.
3. Determines:
   - **Changed files:** Files present in both working directory and index but whose content hashes differ.
   - **New files:** Files present in working directory but missing in the index.
   - **Deleted files:** Files present in the index but missing in the working directory.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

**Code Snippet:**

```python
def get_status():
    """Get status of working copy, return tuple of (changed_paths, new_paths,
    deleted_paths).
    """
    paths = set()
    for root, dirs, files in os.walk('.'):
        dirs[:] = [d for d in dirs if d != '.git']
        for file in files:
            path = os.path.join(root, file)
            path = path.replace('\\', '/')
            if path.startswith('./'):
                path = path[2:]
            paths.add(path)
    entries_by_path = {e.path: e for e in read_index()}
    entry_paths = set(entries_by_path)
    changed = {p for p in (paths & entry_paths)
               if hash_object(read_file(p), 'blob', write=False) !=
                  entries_by_path[p].sha1.hex()}
    new = paths - entry_paths
    deleted = entry_paths - paths
    return (sorted(changed), sorted(new), sorted(deleted))
```

---

### `status()`

**Purpose:**  
Prints a human-readable summary of the working copy status, listing changed, new, and deleted files.

**Parameters:**  
None.

**Returns:**  
None. Outputs are printed to standard output.

**Operation Details:**  
- Calls `get_status()` to obtain file status.
- Prints sections for changed, new, and deleted files if any exist.

**Example Usage:**

```python
status()
```

*Output Example:*

```
changed files:
    README.md
    src/main.py
new files:
    docs/usage.md
deleted files:
    old_script.py
```

**Code Snippet:**

```python
def status():
    """Show status of working copy."""
    changed, new, deleted = get_status()
    if changed:
        print('changed files:')
        for path in changed:
            print('   ', path)
    if new:
        print('new files:')
        for path in new:
            print('   ', path)
    if deleted:
        print('deleted files:')
        for path in deleted:
            print('   ', path)
```

---

### `diff()`

**Purpose:**  
Displays unified diffs between the files in the index and their counterparts in the working directory for all changed files.

**Parameters:**  
None.

**Returns:**  
None. Outputs diff lines to standard output.

**Operation Details:**  
1. Calls `get_status()` to get changed files.
2. Reads the blob data from the index for each changed file.
3. Reads the current file content from the working directory.
4. Uses Python’s `difflib.unified_diff()` to generate and print the diff.

**Example Usage:**

```python
diff()
```

*Sample Output:*

```
--- README.md (index)
+++ README.md (working copy)
@@ -1,4 +1,5 @@
 # Project Title
-
+Added a new introduction paragraph.
 ...
----------------------------------------------------------------------
```

**Code Snippet:**

```python
def diff():
    """Show diff of files changed (between index and working copy)."""
    changed, _, _ = get_status()
    entries_by_path = {e.path: e for e in read_index()}
    for i, path in enumerate(changed):
        sha1 = entries_by_path[path].sha1.hex()
        obj_type, data = read_object(sha1)
        assert obj_type == 'blob'
        index_lines = data.decode().splitlines()
        working_lines = read_file(path).decode().splitlines()
        diff_lines = difflib.unified_diff(
                index_lines, working_lines,
                '{} (index)'.format(path),
                '{} (working copy)'.format(path),
                lineterm='')
        for line in diff_lines:
            print(line)
        if i < len(changed) - 1:
            print('-' * 70)
```

---

### `add(paths)`

**Purpose:**  
Adds one or more file paths to the Git index, hashing their contents and updating the index entries.

**Parameters:**  
- `paths`: List of file paths (strings) to add.

**Returns:**  
None.

**Operation Details:**  
- Normalizes paths to use forward slashes.
- Reads current index entries and excludes any matching the paths to add.
- For each new path:
  - Reads the file content and hashes it as a blob.
  - Collects metadata from `os.stat()`.
  - Creates a new `IndexEntry` and appends it.
- Sorts entries by path and writes back to the index.

**Example Usage:**

```python
add(['src/new_feature.py', 'README.md'])
```

**Code Snippet:**

```python
def add(paths):
    """Add all file paths to git index."""
    paths = [p.replace('\\', '/') for p in paths]
    all_entries = read_index()
    entries = [e for e in all_entries if e.path not in paths]
    for path in paths:
        sha1 = hash_object(read_file(path), 'blob')
        st = os.stat(path)
        flags = len(path.encode())
        assert flags < (1 << 12)
        entry = IndexEntry(
                int(st.st_ctime), 0, int(st.st_mtime), 0, st.st_dev,
                st.st_ino, st.st_mode, st.st_uid, st.st_gid, st.st_size,
                bytes.fromhex(sha1), flags, path)
        entries.append(entry)
    entries.sort(key=operator.attrgetter('path'))
    write_index(entries)
```

---

### `read_index()`

**Purpose:**  
Reads the Git index file and returns a list of `IndexEntry` objects representing the tracked files.

**Parameters:**  
None.

**Returns:**  
List of `IndexEntry` objects.

**Operation Details:**  
- Opens `.git/index` and reads raw bytes.
- Checks SHA-1 checksum for data integrity.
- Parses header and entries according to Git index format version 2.
- Extracts metadata and file paths, creates `IndexEntry` instances.
- Returns the list of entries.

**Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file.

**Parameters:**  
- `entries`: List of `IndexEntry` objects to write.

**Returns:**  
None.

**Operation Details:**  
- Serializes each entry into the binary format expected by Git.
- Computes SHA-1 checksum of the data.
- Writes the data and checksum to `.git/index`.

---

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object by SHA-1 prefix, decompresses and parses it, returning its type and raw data.

**Parameters:**  
- `sha1_prefix`: String prefix of the SHA-1 hash of the object.

**Returns:**  
Tuple `(object_type, data_bytes)`.

**Raises:**  
`ValueError` if the object cannot be found or the prefix is ambiguous.

---

### `read_file(path)`

**Purpose:**  
Reads a file's contents as bytes.

**Parameters:**  
- `path`: File system path.

**Returns:**  
Bytes of file content.

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Hashes object data of a given type, optionally writes it to the object store, and returns the SHA-1 hex string.

**Parameters:**  
- `data`: Bytes of object data.
- `obj_type`: Object type string (`'blob'`, `'tree'`, `'commit'`, etc.).
- `write`: Boolean, whether to write the object file.

**Returns:**  
SHA-1 hex string of the object.

---

## ASCII Diagram: Status Detection Flow

```
+----------------+
| Working Dir    |       (os.walk excludes .git)
| All Files      |------------------+
+----------------+                  |
                                    v
                            +-----------------+
                            | Git Index       |   (read_index)
                            | Entries         |
                            +-----------------+
                                    |
                +-------------------+---------------------+
                |                   |                     |
                v                   v                     v
         Changed Files         New Files            Deleted Files
   (paths in both but content  (in working dir      (in index but not
    hashes differ)              but not in index)    in working dir)

```

---

# Summary

The `status.md` file documents the core working copy status functionality in pygit. Together, functions like `get_status()`, `status()`, and `diff()` enable users to detect and inspect file changes, additions, and deletions relative to the Git index. These are foundational operations that precede committing changes or preparing to push to a remote repository. The file also describes supporting functions for reading the index, hashing files, and managing index entries, providing a comprehensive reference for the working copy status subsystem.