# ls_files.md

# Documentation for `ls_files` Function

---

## Overview

The `ls_files` function is a utility within the `pygit` project designed to list the files currently registered in the Git index. It plays an important role in inspecting the staging area before commits are made. This function offers a simple interface to view either just the file paths or extended details including file mode, SHA-1 object hashes, and stage numbers. This documentation is part of the "Index and Working Copy Management" section of the pygit documentation tree, which covers management and inspection of the Git index and working copy files.

`ls_files` provides users and developers a quick way to verify the contents of the index, aiding in debugging, status checks, and scripting around Git operations. It relies on the `read_index` function to parse the `.git/index` file and extract the necessary index entries.

---

## Function Documentation

### Function: `ls_files(details=False)`

#### Purpose

Lists the files currently recorded in the Git index. By default, it prints only the relative file paths. When the `details` parameter is set to `True`, it prints a detailed listing including:

- File mode (in octal)
- SHA-1 object hash (hexadecimal)
- Stage number (indicates merge conflict stages)
- File path

This detailed view is similar to the output of `git ls-files -s`.

#### Parameters

- `details` (bool): Optional; if `True`, prints detailed information about each file; if `False`, prints only the paths.

#### Preconditions

- The `.git/index` file must exist and be valid.
- The `read_index` function must successfully read and parse the index entries.
  
#### Operation

1. Calls `read_index()` to retrieve a list of `IndexEntry` objects representing the current Git index.
2. Iterates over each `IndexEntry`.
3. If `details` is `False`, prints only the `path` attribute of each entry.
4. If `details` is `True`:
   - Extracts the stage number by shifting and masking the `flags` attribute.
   - Prints the file mode in octal form, the SHA-1 hash as a hex string, the stage number, and the file path in a formatted string.

#### Example Usage

```python
# Basic usage: list only file paths in the index
ls_files()

# Detailed listing with mode, SHA-1, stage, and path
ls_files(details=True)
```

**Sample output (details=False):**
```
README.md
src/pygit.py
tests/test_basic.py
```

**Sample output (details=True):**
```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	README.md
100644 3b18e0e92c5b8a6c1a9e47d3a4f1c8d0a8c3ba1f 0	src/pygit.py
100644 b6fc4c620b67d95f953a5c1c1230aaab5db5a1b0 0	tests/test_basic.py
```

---

### Supporting Function: `read_index()`

#### Summary

Reads and parses the `.git/index` file, returning a list of `IndexEntry` objects representing the files currently staged.

#### Operation Highlights

- Reads `.git/index` as a binary file.
- Validates the SHA-1 checksum of the index file.
- Parses the header for signature and version.
- Iteratively unpacks each index entry according to Git index format version 2.
- Extracts path names from null-terminated strings following each entry.
- Returns a list of `IndexEntry` instances.

---

## Additional Context and Details

### Git Index Structure (Simplified ASCII Diagram)

```
+-------------------------------------------------------------+
| Signature ('DIRC') | Version (2) | Number of Entries (N)    |
+-------------------------------------------------------------+
| Entry 1: (fixed 62 bytes) + path + padding (to 8-byte align)|
| Entry 2: ...                                                |
| ...                                                        |
| Entry N: ...                                                |
+-------------------------------------------------------------+
| SHA-1 checksum of all preceding bytes                      |
+-------------------------------------------------------------+
```

Each index entry includes metadata such as creation time, modification time, device, inode, mode, user/group IDs, file size, SHA-1 hash of the blob, flags, and the file path.

### Stage Number Extraction

The stage number is stored as bits 12 and 13 of the `flags` field in each index entry:

```python
stage = (entry.flags >> 12) & 3
```

This number is useful for indicating the version of a file during a merge conflict:

- `0`: Normal entry
- `1`, `2`, `3`: Different stages of conflicted entries.

---

## Summary

The `ls_files` function is a straightforward but essential tool for inspecting the Git index contents within the `pygit` implementation. Its ability to provide detailed file information assists developers in understanding the staging area state and supports higher-level commands and tooling built on top of the index.

---

## Appendix: Code Snippet for `ls_files`

```python
def ls_files(details=False):
    """Print list of files in index (including mode, SHA-1, and stage number
    if "details" is True.
    """
    for entry in read_index():
        if details:
            stage = (entry.flags >> 12) & 3
            print('{:6o} {} {}\t{}'.format(
                    entry.mode, entry.sha1.hex(), stage, entry.path))
        else:
            print(entry.path)
```

---

# End of `ls_files.md` Documentation