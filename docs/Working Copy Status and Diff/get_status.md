# get_status.md

# Overview

The `get_status` module is responsible for determining the status of files in the working copy relative to the Git index. Specifically, it identifies which files have been changed, which are new (untracked), and which have been deleted since the last commit or index update. This functionality is crucial for providing users with an accurate snapshot of their repository's state, enabling commands like `git status` to work correctly. Within the broader documentation tree, `get_status.md` resides under the "Working Copy Status and Diff" section, complementing files such as `status.md` and `diff.md`. It leverages core index reading functions and file hashing utilities to compare the current filesystem state with the tracked Git index.

---

# Function Documentation

## `get_status()`

### Purpose

The `get_status` function scans the working directory and compares its contents against the Git index to categorize files into three groups:

- **Changed files**: Files tracked by Git whose contents have been modified.
- **New files**: Files present in the working directory but not tracked by Git.
- **Deleted files**: Files tracked by Git that are missing from the working directory.

This enables higher-level commands to report the current state of the working copy accurately.

### Parameters

This function takes no parameters.

### Returns

A tuple of three lists:

```python
(changed_paths, new_paths, deleted_paths)
```

- `changed_paths`: Sorted list of paths that have been modified since last indexing.
- `new_paths`: Sorted list of paths newly added but untracked.
- `deleted_paths`: Sorted list of paths removed from the working directory but still tracked.

### Preconditions

- The function assumes the current working directory is the root of the Git repository.
- The `.git/index` file exists and is readable (or empty if no files are tracked).
- The repository uses POSIX-style forward slashes `/` in paths for consistency.

### How it Operates (Step-by-Step)

1. **Walk the Working Directory:**  
   Recursively traverse the current directory (`.`), ignoring `.git` directories, to collect all file paths present in the working copy.
   
2. **Normalize Paths:**  
   Ensure paths use forward slashes and remove leading `./` prefixes for uniformity.
   
3. **Read Index Entries:**  
   Load the current Git index entries using `read_index()`, which returns a list of `IndexEntry` objects containing metadata and SHA-1 hashes for tracked files.
   
4. **Build Path Sets:**  
   Create a set of file paths from both the working copy and the index for efficient set operations.
   
5. **Detect Changed Files:**  
   For files present in both working copy and index, compute the SHA-1 hash of the current file contents (without writing to the object store). Compare it against the stored SHA-1 in the index entry. If different, mark as changed.
   
6. **Identify New Files:**  
   Files present in the working copy but not in the index are considered new.
   
7. **Identify Deleted Files:**  
   Files present in the index but missing from the working copy are considered deleted.
   
8. **Return Results:**  
   Return sorted lists of changed, new, and deleted paths.

### Example Usage

```python
from pygit import get_status

changed_files, new_files, deleted_files = get_status()

print("Changed files:")
for f in changed_files:
    print("  ", f)

print("New files:")
for f in new_files:
    print("  ", f)

print("Deleted files:")
for f in deleted_files:
    print("  ", f)
```

### Code Snippet

```python
def get_status():
    """Get status of working copy, return tuple of (changed_paths, new_paths,
    deleted_paths).
    """
    paths = set()
    for root, dirs, files in os.walk('.'):
        # Skip .git directory to avoid indexing Git internal files
        dirs[:] = [d for d in dirs if d != '.git']
        for file in files:
            path = os.path.join(root, file)
            path = path.replace('\\', '/')
            if path.startswith('./'):
                path = path[2:]
            paths.add(path)
    entries_by_path = {e.path: e for e in read_index()}
    entry_paths = set(entries_by_path)
    
    # Files tracked in both index and working tree, check for changes
    changed = {p for p in (paths & entry_paths)
               if hash_object(read_file(p), 'blob', write=False) !=
                  entries_by_path[p].sha1.hex()}
    
    # Files in working directory but not in index
    new = paths - entry_paths
    
    # Files in index but missing in working directory
    deleted = entry_paths - paths
    
    return (sorted(changed), sorted(new), sorted(deleted))
```

---

# ASCII Diagram: Status Determination Workflow

```
+----------------------+        +-------------------+
| Working Directory    |        |      Git Index     |
| (files on disk)      |        | (tracked files)    |
+----------+-----------+        +----------+--------+
           |                                |
           | Collect all file paths         | Read all index entries
           |                                |
           +---------------+----------------+
                           |
           +---------------v----------------+
           |      Set of working copy files  |
           +---------------+----------------+
                           |
           +---------------v----------------+
           |      Set of index tracked files |
           +---------------+----------------+
                           |
           +---------------v----------------+
           | Compare sets:                   |
           | - Intersection: check if file  |
           |   contents differ -> Changed    |
           | - In working copy only -> New   |
           | - In index only -> Deleted      |
           +---------------+----------------+
                           |
           +---------------v----------------+
           |   Output sorted lists of paths  |
           +--------------------------------+
```

---

# Summary

The `get_status` function is a cornerstone for reflecting the current state of the working copy relative to the Git index. It integrates file system traversal, index reading, and hashing to precisely classify files as changed, new, or deleted. This mechanism supports commands like `git status` and underpins Git's ability to track project changes efficiently.