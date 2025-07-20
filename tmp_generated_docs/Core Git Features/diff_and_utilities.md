---
sidebar_position: 5
---

# Diff Feature Implementation and Supporting Utilities

## Overview

This document explains the implementation of the diff feature in pygit, as well as the supporting utilities that enable it. The diff feature is responsible for detecting file changes by comparing different states of the repository, such as the working directory, the index, and commit snapshots. It covers how file changes are detected, how Git objects and index entries are read and processed, and how diffs are generated and displayed to the user. This file fits within the broader "Core Git Features" documentation section, complementing index operations, status checks, and object handling, thereby providing a comprehensive understanding of how pygit tracks and presents changes in a Git repository.

---

## Detecting File Changes

The diff feature primarily compares content from two sources to identify changes:

- **Index vs Working Directory:** Detects modifications in files that have not yet been staged.
- **Tree/Object vs Index:** Detects staged changes compared to the last commit or tree.

The diff utilities leverage index entries and object data to perform these comparisons efficiently.

### Key Concepts

- **Index Entry:** Metadata about a tracked file, including its SHA-1 object hash, file mode, size, and modification timestamps.
- **Git Object:** Blobs (file contents), trees (directories), and commits that represent repository state.
- **Diff Output:** A structured textual representation that describes additions, deletions, and modifications.

---

## Important Functions

### `diff_trees(tree1, tree2)`

#### Purpose

Compares two Git tree objects and generates a diff that represents changes between them, such as added, modified, or deleted files.

#### Parameters

- `tree1`: The first tree object (e.g., from a commit or the index).
- `tree2`: The second tree object to compare against (e.g., working directory snapshot).

#### Operation

1. Read both trees and list their entries.
2. For each entry (file or subdirectory), compare presence and SHA-1 hashes.
3. Identify additions (entries in `tree2` but not in `tree1`), deletions (entries in `tree1` but not in `tree2`), and modifications (entries present in both but with different hashes).
4. For modified files, read blob contents and generate line-by-line diff.
5. Aggregate all changes into a unified diff output.

#### Example Usage

```python
diff_output = diff_trees(commit_tree, index_tree)
print(diff_output)
```

---

### `read_index()`

#### Purpose

Reads the Git index file from the repository and parses it into a structured list of index entries.

#### Parameters

- None (operates on the repository's `.git/index` file).

#### Operation

1. Open and read the binary index file.
2. Parse header information (version, entry count).
3. Iterate through entries, extracting file metadata and SHA-1 hashes.
4. Return a list of index entry objects.

#### Example Usage

```python
index_entries = read_index()
for entry in index_entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### `diff_index_to_workdir(index_entries, workdir_path)`

#### Purpose

Generates a diff between the current index entries and the files in the working directory to identify unstaged changes.

#### Parameters

- `index_entries`: List of index entry objects representing staged files.
- `workdir_path`: Path to the working directory.

#### Operation

1. For each index entry, read the corresponding working directory file.
2. Compute SHA-1 hash of the working directory file content.
3. Compare with the index entry's SHA-1.
4. If hashes differ, generate a diff between staged and working directory versions.
5. Collect diffs for all changed files.

#### Example Usage

```python
unstaged_diffs = diff_index_to_workdir(index_entries, repo_workdir)
for diff in unstaged_diffs:
    print(diff)
```

---

### `generate_unified_diff(old_content, new_content, filename)`

#### Purpose

Produces a unified diff string between two versions of file content, suitable for display in patch format.

#### Parameters

- `old_content`: String content of the original file.
- `new_content`: String content of the modified file.
- `filename`: Name of the file being diffed.

#### Operation

1. Split both contents into lines.
2. Use a diff algorithm (e.g., Myers' algorithm) to find line insertions, deletions, and modifications.
3. Format the changes into unified diff text with proper headers and hunk markers.
4. Return the formatted diff string.

#### Example Usage

```python
diff_text = generate_unified_diff(old_file_content, new_file_content, "example.txt")
print(diff_text)
```

---

## ASCII Diagram: Diff Processing Flow

```
+------------------+          +------------------+          +------------------+
|  First Tree/Index |          |  Second Tree/WD  |          |    Diff Output   |
| (e.g., commit)    |          | (e.g., index or  |          | (Unified diff    |
|                  |          |  working dir)     |          |  format)         |
+--------+---------+          +---------+--------+          +---------+--------+
         |                              |                             |
         |                              |                             |
         |      Compare entries         |                             |
         |----------------------------->|                             |
         |                              |                             |
         |          Read blobs          |                             |
         |<-----------------------------|                             |
         |                              |                             |
         | Generate line diffs for files                             |
         |------------------------------------------------------------>|
         |                              |                             |
         +------------------------------------------------------------>+
```

---

## Summary

This file documents the core mechanisms pygit uses to detect and present changes between repository states. By understanding how trees, index entries, and working directory files are read and compared, as well as how diffs are generated and formatted, developers can extend or debug the diff functionality effectively. The described utilities serve as foundational tools for related features such as status reporting and commit preparation.