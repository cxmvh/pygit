# diff.md

## Overview

The `diff.md` document describes the functionality for displaying differences ("diffs") between the Git index and the working copy files in the `pygit` repository. It explains how to use diff commands to identify changes made in the working directory relative to the index, focusing on changed files. This file is part of the **Status and Diff** section in the documentation tree, which covers working copy status, index manipulation, and diffing operations.

The diff functionality is crucial for users and developers to understand what modifications have been made locally before committing or staging changes. It leverages other core components such as reading the Git index, reading Git objects, and file IO utilities.

---

## Important Functions

### 1. `diff()`

#### Purpose

Shows line-by-line diffs of files that have changed between the Git index and the working copy. It highlights the differences to help users review modifications before staging or committing.

#### Parameters

- None

#### Preconditions

- The Git index file (`.git/index`) must exist and be readable.
- The working copy files should be accessible.
- The repository should be initialized.

#### How It Operates

1. Calls `get_status()` to retrieve sets of changed, new, and deleted files.
2. Reads the Git index entries into a dictionary keyed by file path.
3. Iterates over each changed file path.
4. For each changed file:
   - Retrieves the SHA-1 hash of the file's blob from the index.
   - Reads the corresponding blob object from the Git object store.
   - Decodes the blob data into lines representing the index version.
   - Reads the current working copy file contents and decodes into lines.
   - Uses Python's `difflib.unified_diff` to generate a unified diff between index and working copy content.
   - Prints the diff output to standard output.
5. Prints a separator line between diffs of multiple changed files.

#### Example Usage

```bash
$ pygit diff
```

This command will print diffs for all files changed but not yet staged.

#### Code Snippet

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

### 2. `get_status()`

#### Purpose

Determines the current status of files in the working directory relative to the Git index. Returns lists of changed, new, and deleted files.

#### Parameters

- None

#### Returns

- Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`

#### How It Operates

1. Walks the working directory recursively, skipping `.git` directory.
2. Collects all file paths in the working directory.
3. Reads the Git index entries and maps them by path.
4. Compares working copy file contents (hashed as blobs) to the index blobs:
   - Files present in both but with different hashes → changed.
   - Files in working copy but not in index → new.
   - Files in index but missing in working copy → deleted.
5. Returns sorted lists of changed, new, and deleted paths.

#### Example Usage

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

#### Code Snippet

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

### 3. `read_index()`

#### Purpose

Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the entries in the index.

#### Parameters

- None

#### Returns

- List of `IndexEntry` objects.

#### How It Operates

1. Reads the raw `.git/index` file data.
2. Validates the index file signature and version.
3. Validates the SHA-1 checksum of the index contents.
4. Parses individual index entries from the binary data using struct unpacking.
5. Extracts file paths from the entry data.
6. Returns a list of fully parsed `IndexEntry` objects.

#### Example Usage

```python
entries = read_index()
for entry in entries:
    print(f"{entry.path} - {entry.sha1.hex()}")
```

#### Code Snippet

```python
def read_index():
    """Read git index file and return list of IndexEntry objects."""
    try:
        data = read_file(os.path.join('.git', 'index'))
    except FileNotFoundError:
        return []
    digest = hashlib.sha1(data[:-20]).digest()
    assert digest == data[-20:], 'invalid index checksum'
    signature, version, num_entries = struct.unpack('!4sLL', data[:12])
    assert signature == b'DIRC', \
            'invalid index signature {}'.format(signature)
    assert version == 2, 'unknown index version {}'.format(version)
    entry_data = data[12:-20]
    entries = []
    i = 0
    while i + 62 < len(entry_data):
        fields_end = i + 62
        fields = struct.unpack('!LLLLLLLLLL20sH', entry_data[i:fields_end])
        path_end = entry_data.index(b'\x00', fields_end)
        path = entry_data[fields_end:path_end]
        entry = IndexEntry(*(fields + (path.decode(),)))
        entries.append(entry)
        entry_len = ((62 + len(path) + 8) // 8) * 8
        i += entry_len
    assert len(entries) == num_entries
    return entries
```

---

### 4. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object by its SHA-1 prefix from the object store, decompresses it, and returns its type and raw data.

#### Parameters

- `sha1_prefix` (str): The prefix of the SHA-1 hash identifying the object.

#### Returns

- Tuple `(object_type: str, data: bytes)`

#### How It Operates

1. Finds the full object file path corresponding to the SHA-1 prefix.
2. Reads the compressed object file and decompresses it using zlib.
3. Parses the object header to extract type and size.
4. Extracts the object data payload.
5. Verifies that the data size matches the header size.
6. Returns the object type and data bytes.

#### Example Usage

```python
obj_type, data = read_object('a1b2c3')
print(f"Object type: {obj_type}")
print(data)
```

#### Code Snippet

```python
def read_object(sha1_prefix):
    """Read object with given SHA-1 prefix and return tuple of
    (object_type, data_bytes), or raise ValueError if not found.
    """
    path = find_object(sha1_prefix)
    full_data = zlib.decompress(read_file(path))
    nul_index = full_data.index(b'\x00')
    header = full_data[:nul_index]
    obj_type, size_str = header.decode().split()
    size = int(size_str)
    data = full_data[nul_index + 1:]
    assert size == len(data), 'expected size {}, got {} bytes'.format(
            size, len(data))
    return (obj_type, data)
```

---

### 5. `read_file(path)`

#### Purpose

Reads the contents of a file from the filesystem as bytes.

#### Parameters

- `path` (str): The file path to read.

#### Returns

- `bytes`: The raw contents of the file.

#### How It Operates

- Opens the file in binary read mode.
- Reads all bytes from the file.
- Returns the bytes.

#### Example Usage

```python
content = read_file('README.md')
print(content.decode())
```

#### Code Snippet

```python
def read_file(path):
    """Read contents of file at given path as bytes."""
    with open(path, 'rb') as f:
        return f.read()
```

---

## ASCII Diagram: Diffing Process Overview

```
+-----------------+          +--------------------+          +----------------------+
| Git Index       |          | Working Copy       |          | Diff Output          |
| (.git/index)    |          | Filesystem         |          | (unified diff lines) |
+--------+--------+          +---------+----------+          +-----------+----------+
         |                             |                               ^
         |                             |                               |
         |                             |                               |
         |                             |                               |
         |          Read index entries |                               |
         |---------------------------->|                               |
         |                             |                               |
         |       Read file contents    |                               |
         |<----------------------------|                               |
         |                             |                               |
         |                             |                               |
         |                             |                               |
         |   Read blob object data     |                               |
         |<-------------------------------------------------------------|
         |                             |                               |
         |                             |                               |
         |          Generate diff      |------------------------------>|
         |                             |                               |
         |                             |                               |
         |                             |                               |
         +-----------------------------+-------------------------------+
```

This diagram shows that the diff command reads the Git index entries and their corresponding blob objects, reads the current working copy files, and then generates a unified diff output comparing the two.

---

## Summary

The `diff.md` file documents the core diff functionality within the `pygit` project, explaining how to show textual differences between files staged in the index and the current working copy. The key function `diff()` combines status detection, object reading, and file reading to produce unified diffs line-by-line. This is vital for users to inspect local modifications before committing them to the repository. The documentation also references related functions used to read the index, objects, and files, ensuring users understand the underlying mechanisms.