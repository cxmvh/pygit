```mdx
# Working Copy and Status Management: `status.md`

This document details the functionality of the `status()` and `get_status()` functions, which are central to determining the current state of the working directory in relation to the git index and repository history. These functions enable detecting file changes such as modifications, additions, and deletions, thus providing the foundation for showing status information similar to `git status`.

They are part of the broader "Working Copy and Status Management" area, which handles interactions between the working directory, the staging index, and the repository object database. Understanding these functions is critical for developers implementing or extending git-like status reporting.

---

## Function: `status()`

### Purpose

`status()` serves as the primary interface to compute the working directory status. It identifies files that have been modified, newly added, or deleted compared to the last committed state recorded in the index. This function consolidates information from the working directory, the index, and the repository to present a comprehensive status overview.

### Parameters

- None explicitly required; implicitly operates on the current repository context, accessing the working directory, index, and `.git` metadata.

### Operation Steps

1. **Load Index and Repository Metadata**  
   Reads the current git index entries to obtain a baseline of tracked files and their recorded states.

2. **Walk Working Directory Files**  
   Scans the working directory recursively to locate all files, collecting their metadata (such as modification times, sizes, and content hashes).

3. **Compare Working Directory to Index**  
   For each file found, compares its current state to the corresponding index entry (if any):
   - If the file exists in the index:
     - Check if the content has changed since the last index update.
   - If the file is new (not in index):
     - Mark as "new file".
   - If a file exists in the index but is missing from the working directory:
     - Mark as "deleted file".

4. **Produce Status Summary**  
   Aggregates results into categories: changed, new, deleted files.

### Example Usage

```python
from pygit import status

# Retrieve current working directory status
current_status = status()

print("Changed files:", current_status.changed)
print("New files:", current_status.new)
print("Deleted files:", current_status.deleted)
```

---

## Function: `get_status(path)`

### Purpose

`get_status(path)` is a helper function invoked by `status()` to evaluate the status of a specific file or directory path. It determines whether the target is unchanged, modified, new, or deleted relative to the index, facilitating fine-grained status detection.

### Parameters

- `path` (string): The filesystem path of the file or directory to check.

### Operation Steps

1. **Check Index for Path**  
   Determines if `path` is recorded in the index.

2. **Check Working Directory for Path**  
   Verifies existence and state of `path` in the working directory.

3. **Compare Content and Metadata**  
   If present in both index and working directory, compares file metadata and contents to detect modifications.

4. **Classify Status**  
   Returns one of:
   - `unchanged`: file matches index entry.
   - `modified`: file differs from index.
   - `new`: file exists only in working directory.
   - `deleted`: file exists only in index.

### Example Usage

```python
from pygit import get_status

file_path = "src/main.py"
file_status = get_status(file_path)

if file_status == "modified":
    print(f"{file_path} has been modified.")
elif file_status == "new":
    print(f"{file_path} is a new file.")
elif file_status == "deleted":
    print(f"{file_path} has been deleted.")
else:
    print(f"{file_path} is unchanged.")
```

---

## ASCII Diagram: Status Determination Flow

```
+----------------------+
|   Working Directory   |
|  (files & metadata)   |
+----------+-----------+
           |
           | compare
           v
+----------------------+          +-----------------+
|        Index         |<---------|  Repository     |
| (tracked files &     |          | (commits &      |
|  recorded metadata)  |          |  object data)   |
+----------+-----------+          +-----------------+
           |
           | status()
           v
+---------------------------+
|  Status Summary Output    |
|  (changed, new, deleted)  |
+---------------------------+
```

This diagram illustrates how `status()` uses information from the working directory and index, referencing the repository as needed, to produce a summary of file status changes.

---

For related details on differences between files in the working directory and the index, see [diff.md](./diff.md).

For broader context on index and commit management, refer to [index_and_commit.md](../index_and_commit.md).
```