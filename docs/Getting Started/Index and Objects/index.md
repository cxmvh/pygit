# index.md - Git Index Reading and Writing Documentation

## Overview

This document provides detailed information on handling the Git index within the `pygit` project, focusing on reading from and writing to the index file. The Git index is a critical component that tracks the state of the working directory in preparation for commits. This file covers the two primary functions involved in index manipulation:

- `read_index()`: Reads and parses the Git index file, returning a list of index entries representing tracked files.
- `write_index(entries)`: Writes a list of index entries back to the Git index file, ensuring the index is updated correctly.

Located under the "Index and Objects" section of the broader pygit documentation, this documentation connects with repository initialization (`pygit.init`), diff computation (`pygit.diff`), and status reporting (`pygit.status`) by providing foundational support for index management.

---

## Function: read_index()

### Purpose
Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the files currently staged in the index.

### Parameters
- None

### Returns
- `List[IndexEntry]`: A list of `IndexEntry` named tuples or objects containing metadata for each file entry in the index.

### Preconditions
- The `.git/index` file must exist and be properly formatted.
- The index file checksum must be valid to ensure data integrity.

### Description
The `read_index` function opens the index file and reads its entire contents as bytes. It validates the SHA-1 checksum at the end of the file to confirm the index has not been corrupted. The header is parsed to verify the signature (`DIRC`) and version (expected 2).

Then, the function iterates over the index entries stored sequentially. Each entry consists of fixed-length metadata fields followed by a variable-length file path string. The function carefully calculates the length of each entry, considering padding to align entries to an 8-byte boundary. Each entry is unpacked using `struct` to extract fields such as timestamps, device and inode numbers, file mode, user and group IDs, file size, SHA-1 object hash, flags, and the file path.

After processing all entries, the function returns the list of parsed `IndexEntry` objects.

### Step-by-Step Operation
1. Attempt to read the `.git/index` file bytes.
2. Extract and verify the SHA-1 checksum to ensure file integrity.
3. Unpack the header to confirm file format signature and version.
4. Iterate through the entries section:
   - Unpack fixed metadata fields.
   - Locate the null-terminated file path.
   - Decode the path and create an `IndexEntry`.
   - Calculate padded entry length and move to the next entry.
5. Assert the number of parsed entries equals the header count.
6. Return the list of index entries.

### Example Usage

```python
from pygit import read_index

index_entries = read_index()
for entry in index_entries:
    print(f'File: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}')
```

### ASCII Diagram: Index File Structure

```
+----------------+----------+------------+-----------------+
| Header (12 B)  | Entries  | Padding    | SHA-1 Checksum  |
| "DIRC", ver=2, | Fixed+Var| Align to 8B| (20 bytes)      |
| num_entries    | length   | bytes      |                 |
+----------------+----------+------------+-----------------+

Each Entry:
+-------------------------------+
| 62 bytes fixed metadata fields |
+-------------------------------+
| Variable length path string    |
+-------------------------------+
| Padding to 8-byte boundary     |
+-------------------------------+
```

---

## Function: write_index(entries)

### Purpose
Writes a list of `IndexEntry` objects to the Git index file (`.git/index`), updating the index to reflect the current staged files.

### Parameters
- `entries (List[IndexEntry])`: The list of index entries to write to the index.

### Returns
- None

### Preconditions
- The entries must be valid and sorted by file path.
- Each entry must have correctly formatted metadata and path information.

### Description
The `write_index` function serializes each `IndexEntry` into the binary format required by Git. It packs fixed metadata fields using `struct.pack` and appends the UTF-8 encoded file path, padding the entry to an 8-byte boundary as per the Git index specification.

After serializing all entries, the function concatenates them with a header containing the signature (`DIRC`), version number (2), and the total number of entries. It then computes a SHA-1 checksum of the entire data block (excluding the checksum field) to append at the end of the index file.

Finally, the complete binary data is written to `.git/index` atomically.

### Step-by-Step Operation
1. Initialize a list to collect packed entries.
2. For each `IndexEntry` in `entries`:
   - Pack all fixed fields (timestamps, device, inode, mode, uid, gid, size, SHA-1, flags).
   - Encode the file path to bytes.
   - Calculate padded length to next multiple of 8 bytes.
   - Append null bytes to pad entry to aligned length.
   - Add packed entry to the list.
3. Pack the index header with signature, version, and entry count.
4. Concatenate the header and all packed entries.
5. Calculate SHA-1 digest of the data.
6. Append the digest to the data.
7. Write the complete data to `.git/index`.

### Example Usage

```python
from pygit import write_index, IndexEntry

# Assume 'entries' is a list of IndexEntry objects obtained from read_index or constructed manually
write_index(entries)
print("Index updated with {} entries.".format(len(entries)))
```

---

## Additional Context

The `read_index` and `write_index` functions form the backbone of index management in pygit. They are used extensively by commands that modify the index, such as `add()`, and are prerequisites for building trees (`write_tree()`), creating commits (`commit()`), and computing diffs (`diff()`). The index file format and these functions conform closely to the Git specification for index version 2.

---

## Summary

| Function Name | Purpose                         | Key Points                                     |
|---------------|--------------------------------|------------------------------------------------|
| `read_index`  | Parse the `.git/index` file and return entries | Validates checksum, parses entries sequentially |
| `write_index` | Serialize entries and write to `.git/index`   | Packs entries with padding, appends SHA-1 checksum |

Together, these functions enable pygit to accurately track and manipulate the staging area, essential for version control operations.

---

# End of index.md Documentation