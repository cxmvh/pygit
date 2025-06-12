# get_status.md

# Determining Changed, New, and Deleted Files in the Working Copy

---

## Overview

The `get_status.md` document explains the mechanisms used to determine the status of files in the working copy of a Git repository managed by `pygit`. Specifically, it focuses on identifying files that have been **changed**, **added (new)**, or **deleted** relative to the current Git index. This functionality is critical for status reporting, which informs users about modifications pending commit, and forms the foundation for other Git operations like committing, diffing, and adding files.

Within the overall documentation tree, this file resides under the **Status and Diff** section, which covers working copy status detection, index manipulation, and diffing. The functions described here interoperate closely with index management (`read_index`), file reading utilities (`read_file`), and object hashing (`hash_object`), enabling accurate tracking of changes.

---

## Key Functions

### `get_status()`

#### Purpose

`get_status()` inspects the current working directory and compares it against the Git index to determine three categories of file changes:

- **Changed files:** Files present in both the working copy and the index but whose content has changed.
- **New files:** Files present in the working copy but not tracked in the index.
- **Deleted files:** Files tracked in the index but missing from the working copy.

It returns a tuple of three sorted lists representing these file paths.

#### Parameters

- None

#### Returns

- `changed_paths` (List[str]): Sorted list of file paths that have changed.
- `new_paths` (List[str]): Sorted list of new (untracked) file paths.
- `deleted_paths` (List[str]): Sorted list of deleted file paths.

#### Preconditions

- Must be run inside a valid Git repository with an existing `.git/index` file.
- The Git index must be readable and valid.

#### Operation Details

1. **Collect Working Copy Paths:**
   - Walk the working directory recursively using `os.walk`, excluding the `.git` directory.
   - Normalize file paths to use forward slashes and strip leading `./`.
   - Collect all file paths into a set.

2. **Read Index Entries:**
   - Use `read_index()` to load the current Git index entries.
   - Build a dictionary mapping file paths to their corresponding index entries.
   - Extract the set of indexed file paths.

3. **Determine Changed Files:**
   - For files present in both working copy and index, read file contents.
   - Compute the blob SHA-1 hash of the working copy file (without writing).
   - Compare the computed SHA-1 with the SHA-1 stored in the index entry.
   - If hashes differ, the file is marked as changed.

4. **Identify New and Deleted Files:**
   - New files are those in the working copy but not in the index.
   - Deleted files are those in the index but missing from the working copy.

5. **Return Results:**
   - Return the sorted lists of changed, new, and deleted files.

#### Example Usage

```python
from pygit import get_status

changed, new, deleted = get_status()

print("Changed files:")
for path in changed:
    print("  ", path)

print("New files:")
for path in new:
    print("  ", path)

print("Deleted files:")
for path in deleted:
    print("  ", path)
```

---

## Supporting Functions (Contextual Overview)

While `get_status()` is the central function for determining file changes, it relies on several supporting functions which are briefly described here for context:

### `read_index()`

Reads and parses the Git index file `.git/index` and returns a list of `IndexEntry` objects representing the tracked files.

### `read_file(path)`

Reads the contents of a file at `path` as bytes.

### `hash_object(data, obj_type, write=False)`

Computes the SHA-1 hash of an object (e.g., a blob for a file) without writing it to the object store if `write=False`. Used for comparing working copy file content to index blobs.

---

## ASCII Diagram: Status Determination Workflow

```
+------------------+
| Working Directory |
+------------------+
          |
          |  (os.walk, gather file paths)
          v
   +------------------+       +-----------------+
   | Set of file paths |       | Git Index entries|
   +------------------+       +-----------------+
          |                           |
          | Intersection & Difference|
          +------------+--------------+
                       |
         +-------------+--------------+
         |                            |
+------------------+         +------------------+
| Changed files    |         | New files        |
| (content differs)|         | (not in index)   |
+------------------+         +------------------+
                       |
                +------------------+
                | Deleted files    |
                | (in index missing)|
                +------------------+
```

---

## Integration with Other Commands

- The output of `get_status()` is consumed by commands like `status()` which prints the current repository status to the user.
- It also underpins the `diff()` command which shows detailed differences for changed files.
- Adding files to the index (`add()`) and committing changes (`commit()`) depend on accurate status information to function correctly.

---

# Full `get_status` Function Source for Reference

```python
def get_status():
    """Get status of working copy, return tuple of (changed_paths, new_paths,
    deleted_paths).
    """
    paths = set()
    for root, dirs, files in os.walk('.'):
        # Exclude .git directory from walk
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

# Summary

The `get_status()` function provides a concise, efficient way to detect changes in the working tree relative to the Git index, categorizing files into changed, new, or deleted. This capability forms a cornerstone of typical Git workflows and enables users to understand what modifications are staged or pending staging, facilitating effective version control operations.