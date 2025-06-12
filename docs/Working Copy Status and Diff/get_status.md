# get_status.md

# Utility to Get Paths Categorized by Changed, New, and Deleted

---

## Overview

The `get_status` utility is a core component within the "Working Copy Status and Diff" section of the pygit project. Its primary purpose is to analyze the current working directory against the Git index (staging area) and categorize files into three groups: changed, new, and deleted. This categorization is essential for various Git operations, including showing diffs, preparing commits, and updating the index. By providing a clear distinction between these file states, `get_status` enables other commands and utilities to accurately track modifications in the working copy.

---

## Functions

### `get_status()`

#### Purpose

`get_status()` inspects the working directory and compares the current state of files against the Git index. It returns three sorted lists containing file paths that are:

- **Changed:** Files tracked by Git whose contents differ from the index version.
- **New:** Files present in the working directory but not tracked in the index.
- **Deleted:** Files tracked in the index but missing from the working directory.

#### Parameters

- None

#### Returns

```python
(changed_paths: List[str], new_paths: List[str], deleted_paths: List[str])
```

- `changed_paths`: List of file paths with modifications.
- `new_paths`: List of untracked file paths.
- `deleted_paths`: List of tracked file paths that have been removed.

#### Preconditions

- Assumes the current working directory is a valid Git repository with an initialized `.git` directory.
- Requires access to the Git index (`.git/index`) to read tracked files and metadata.
- Uses auxiliary functions `read_index()`, `read_file()`, and `hash_object()`.

#### Operation Details

1. **Gather Working Directory Paths:**
   - Walk through the current directory tree recursively.
   - Exclude the `.git` directory to avoid scanning Git internals.
   - Normalize file paths to use forward slashes and remove leading `./`.

2. **Read Git Index Entries:**
   - Load index entries via `read_index()`, mapping each entry by its file path.

3. **Determine File States:**
   - **Changed:** Intersection of working directory files and index entries where the SHA-1 hash of the current file content differs from the index.
   - **New:** Files present in the working directory but absent from the index.
   - **Deleted:** Files present in the index but missing in the working directory.

4. **Return Sorted Lists:**
   - Sort each category alphabetically before returning.

#### Example Usage

```python
from pygit import get_status

changed_files, new_files, deleted_files = get_status()

print("Changed files:")
for path in changed_files:
    print("  -", path)

print("New files:")
for path in new_files:
    print("  -", path)

print("Deleted files:")
for path in deleted_files:
    print("  -", path)
```

---

### Supporting Functions Used by `get_status()`

> For completeness, here are brief descriptions of some key functions `get_status()` relies on:

#### `read_index()`

Reads the Git index file and returns a list of `IndexEntry` objects representing tracked files and their metadata.

#### `read_file(path)`

Reads the contents of the file at `path` as bytes.

#### `hash_object(data, obj_type, write=False)`

Computes the SHA-1 hash of the given `data` with specified `obj_type` (usually `'blob'`), optionally writing the object to the Git object store. Here, `write=False` is used to avoid writing during status checks.

---

## ASCII Diagram: File State Categorization Flow

```
+-------------------+       +------------------+       +------------------+
| Working Directory  |       |     Git Index    |       |  Categorization  |
| (Current Files)    |       | (Tracked Files)  |       |                  |
+-------------------+       +------------------+       +------------------+
          |                           |                           |
          | Set of paths (W)          | Set of paths (I)          |
          |                           |                           |
          +-----------+---------------+                           |
                      |                                           |
                      v                                           v
           +--------------------+                   +---------------------------+
           | Intersection (W ∩ I)| <----------------| Compare file SHA-1 hashes  |
           +--------------------+                   +---------------------------+
                      |                                           |
                      v                                           v
          +---------------------+                  +-------------------+
          |   Changed files     |                  |   Deleted files   |
          |  (diff SHA-1s)      |                  |  (I - W)          |
          +---------------------+                  +-------------------+
                      |
                      v
          +---------------------+
          |    New files        |
          |    (W - I)          |
          +---------------------+
```

---

## Summary

The `get_status` utility is foundational for determining the working copy state in pygit. It bridges the physical file system and Git's internal tracking, enabling accurate detection of modifications, additions, and removals. This functionality underpins commands like `diff` and `status`, helping users understand what changes exist before committing or further operations.