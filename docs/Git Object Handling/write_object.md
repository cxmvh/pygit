# write_object.md

# Hashing and Writing Objects to the Git Object Store

---

## Overview

This document details the core functionality for hashing data into Git objects and writing these objects to the Git object store. It covers two critical functions: `hash_object` and `write_file`, which underpin the storage of Git objects (blobs, trees, commits, etc.) in the `.git/objects` directory. These functions are foundational within the "Git Object Handling" section of the documentation tree, enabling the creation and persistence of Git objects that represent the content and history of a repository.

The `hash_object` function computes a SHA-1 hash for given data combined with its type and optionally writes the compressed object to disk, while `write_file` handles the low-level byte writing to files. Together, they ensure the integrity and efficient storage of Git objects, supporting operations such as committing, diffing, and pushing changes.

---

## Function Documentation

### `hash_object(data, obj_type, write=True)`

#### Purpose

`hash_object` generates a SHA-1 hash for the provided data tagged with its Git object type (`blob`, `tree`, `commit`, etc.) and optionally writes the compressed object to the Git object store. This function is essential for creating new Git objects and storing them in the repository.

#### Parameters

- `data` (`bytes`): The raw data content of the object (e.g., file contents for blobs).
- `obj_type` (`str`): The type of Git object (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (`bool`, optional): Whether to write the compressed object to the object store. Defaults to `True`.

#### Preconditions

- The `.git/objects` directory structure exists or can be created.
- `data` is a byte string representing the object content accurately.

#### Operation Steps

1. Construct an object header string of the form: `"<obj_type> <len(data)>"`.
2. Concatenate the header, a null byte (`b'\x00'`), and the data bytes to form the full object representation.
3. Compute the SHA-1 hash of this full data blob; this hash uniquely identifies the object.
4. If `write` is `True`:
   - Determine the storage path based on the SHA-1 hash (first two characters as directory, rest as filename).
   - Create the directory if it does not exist.
   - Compress the full data blob using `zlib`.
   - Write the compressed data to the object file path.
5. Return the SHA-1 hash as a hexadecimal string.

#### Example Usage

```python
# Example: Hash a blob object and write it to the object store.

file_contents = b'Hello, Git!'
obj_type = 'blob'

sha1_hash = hash_object(file_contents, obj_type)
print(f'Object stored with SHA-1: {sha1_hash}')
```

#### ASCII Diagram: Object Storage Path

```
.git/
└── objects/
    └── aa/
        └── bbcdef1234567890abcdef1234567890abcdef  # Object stored here
```

The SHA-1 hash `aabbcdef1234567890abcdef1234567890abcdef` is split into:

- Directory: `aa`
- File: `bbcdef1234567890abcdef1234567890abcdef`

---

### `write_file(path, data)`

#### Purpose

`write_file` writes raw byte data to a file at the specified path. This utility is used primarily for saving compressed Git object data to the object store or writing other Git-related files.

#### Parameters

- `path` (`str`): The file system path where data should be written.
- `data` (`bytes`): The byte content to write to the file.

#### Preconditions

- The directory for `path` exists or can be created by the caller.
- The caller has write permissions to the target location.

#### Operation Steps

1. Open the file at `path` in binary write mode (`'wb'`).
2. Write the provided `data` bytes into the file.
3. Close the file after writing completes.

#### Example Usage

```python
# Example: Write compressed data to a Git object file

compressed_data = zlib.compress(b'blob 11\x00Hello Git!')
object_path = '.git/objects/ab/cdef1234567890abcdef1234567890abcdef'

write_file(object_path, compressed_data)
print(f'Data written to {object_path}')
```

---

## Additional Notes

- The `hash_object` function uses `write_file` internally to perform the actual file writing.
- The Git object store is structured to avoid too many files in one directory by splitting the SHA-1 into a directory and file name.
- Writing an object that already exists does not overwrite the file, ensuring content addressability and immutability.
- These functions are leveraged by higher-level Git operations such as committing changes, updating trees, and pushing objects to remotes.

---

This documentation complements other files in the "Git Object Handling" section, such as `read_object.md` (for reading objects) and `objects.md` (for related hashing and encoding operations), creating a comprehensive reference for Git object manipulation within the `pygit` project.