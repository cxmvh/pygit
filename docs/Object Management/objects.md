# Object Management

This document provides comprehensive reference information on the creation, hashing, reading, writing, and retrieval of Git objects by their SHA-1 hashes within the repository. It also covers handling object data, which is foundational to Git's internal storage and version control mechanisms. This file fits within the broader "Object Management" section of the documentation tree, which focuses on manipulating Git objects, encoding, and packfile creation. Understanding the functions detailed here is essential for developers working on features related to object storage, retrieval, and integrity verification in the Git system.

---

## Function: `hash_object`

### Purpose

Computes the SHA-1 hash of given data after optionally writing it as a Git object to the repository. This function is fundamental for creating Git objects (blobs, trees, commits) and ensuring their integrity and uniqueness based on content.

### Parameters

- `data` (`bytes`): The raw data to be hashed.
- `fmt` (`str`): The type of Git object (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (`bool`): Optional flag indicating whether to store the object in the repository. Defaults to `False`.

### Operation

1. Constructs the Git object header in the format: `<fmt> <len(data)>\0`.
2. Concatenates the header and the data.
3. Computes the SHA-1 hash over this combined byte string.
4. If `write` is `True`, compresses and writes the object to the `.git/objects` directory under a path derived from the hash.
5. Returns the SHA-1 hash as a hex string.

### Example Usage

```python
data = b'hello world\n'
obj_type = 'blob'
sha1_hash = hash_object(data, obj_type, write=True)
print(f"Object stored with SHA-1: {sha1_hash}")
```

---

## Function: `read_object`

### Purpose

Retrieves and decompresses a Git object given its SHA-1 hash, returning its format and raw data. This allows inspection of stored objects and supports operations like displaying file contents or commit metadata.

### Parameters

- `sha` (`str`): The SHA-1 hash of the object to read.

### Operation

1. Locates the object file in `.git/objects/` using the first two characters of `sha` as a directory and the remaining characters as the filename.
2. Reads and decompresses the object file.
3. Parses the header to extract the object type and size.
4. Returns a tuple containing the object type (`str`) and the raw data (`bytes`).

### Example Usage

```python
sha1_hash = 'e69de29bb2d1d6434b8b29ae775ad8c2e48c5391'
obj_type, data = read_object(sha1_hash)
print(f"Object type: {obj_type}")
print(f"Data length: {len(data)} bytes")
```

---

## Function: `write_object`

### Purpose

Writes raw data as a Git object of a specified type to the repository and returns its SHA-1 hash. This is typically used internally after creating or modifying an object before committing it.

### Parameters

- `data` (`bytes`): The content to store.
- `fmt` (`str`): The Git object type.

### Operation

1. Calls `hash_object` with `write=True` to compute the SHA-1 and store the object.
2. Returns the resulting SHA-1 hash.

### Example Usage

```python
blob_data = b'Example content\n'
sha1_hash = write_object(blob_data, 'blob')
print(f"Written object SHA-1: {sha1_hash}")
```

---

## Function: `find_object`

### Purpose

Locates a Git object by its abbreviated or full SHA-1 hash within the repository. Useful for resolving partial hashes entered by users or tools.

### Parameters

- `sha_prefix` (`str`): The abbreviated SHA-1 hash prefix to search for.

### Operation

1. Searches the `.git/objects` directory for objects matching the prefix.
2. If multiple matches are found, raises an error indicating ambiguity.
3. Returns the full SHA-1 hash if a unique match is found.

### Example Usage

```python
prefix = 'e69de29'
full_sha = find_object(prefix)
print(f"Full SHA-1 hash: {full_sha}")
```

---

## Function: `object_data`

### Purpose

Processes object data by decoding or encoding based on its type. This function helps interpret raw data for higher-level operations such as parsing commit or tree objects.

### Parameters

- `fmt` (`str`): The object type (e.g., `'commit'`, `'tree'`, `'blob'`).
- `data` (`bytes`): The raw object data.

### Operation

- For `'blob'`: Returns data as-is.
- For `'commit'`: Parses commit metadata into a structured format.
- For `'tree'`: Parses tree entries into a list of filenames, modes, and SHA-1 hashes.
- Raises an error if the format is unknown.

### Example Usage

```python
obj_type, raw_data = read_object(sha1_hash)
parsed = object_data(obj_type, raw_data)
print(f"Parsed object content:\n{parsed}")
```

---

## ASCII Diagram: Object Storage Structure

```
.git/
└── objects/
    ├──  e6/
    │    └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391  # Object file named by SHA-1 suffix
    ├──  4b/
    │    └── 825dc642cb6eb9a060e54bf8d69288fbee4904  # Another object
    └──  ...
```

- The first two characters of the SHA-1 hash form the directory name.
- The remaining 38 characters form the filename within that directory.
- Object files are zlib compressed.

---

This documentation should serve as a key reference for developers interacting with Git objects at the storage and retrieval level within the `pygit` project, supporting various commands and repository operations. For related information on object encoding and packfiles, see [objects_and_packs.md](./objects_and_packs.md) and for command-specific object reading, refer to [cat_file.md](./cat_file.md).