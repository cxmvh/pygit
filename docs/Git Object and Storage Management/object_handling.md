# Object Handling

This document provides detailed reference information on Git object management within the repository. It covers core operations such as reading Git objects from the database, locating objects by SHA-1 prefix, hashing and writing objects, reading tree and commit objects, and encoding objects for pack file creation. Situated within the **Git Object and Storage Management** section, this file supports many Git commands including `cat-file`, `status`, `ls-files`, `diff`, and `commit`, by enabling efficient and correct object handling in the underlying storage layer.

---

## read_object(sha)

### Purpose

Reads a Git object from the repository given its SHA-1 hash. It retrieves the raw object data, decompresses it, and parses the header to return the object type and content.

### Parameters

- `sha` (str): The 40-character SHA-1 hash identifying the object.

### Description

1. Locate the object file in the `.git/objects/` directory structure, using the first two characters as the directory and the remaining 38 as the filename.
2. Open and decompress the object file using zlib.
3. Parse the header, which includes the object type (e.g., `blob`, `tree`, `commit`, `tag`) and size.
4. Return a tuple `(object_type, data)` where `data` is the object's content bytes.

### Example

```python
obj_type, content = read_object('a1b2c3d4e5f678901234567890abcdef12345678')
print(f"Type: {obj_type}")
print(content.decode())
```

---

## find_object(sha_prefix)

### Purpose

Finds a Git object by a SHA-1 prefix, resolving a partial hash to a full 40-character SHA if uniquely identified.

### Parameters

- `sha_prefix` (str): The initial characters of a SHA-1 hash (minimum length typically 4).

### Description

1. Validate the prefix length and characters.
2. Search the `.git/objects` directory for objects matching the prefix.
3. If exactly one object matches, return its full SHA.
4. Raise an error if no or multiple objects match.

### Example

```python
full_sha = find_object('a1b2c')
print(f"Full SHA: {full_sha}")
```

---

## hash_object(data, obj_type, write=False)

### Purpose

Computes the SHA-1 hash of an object given its content and type, optionally writes the object to the Git object database.

### Parameters

- `data` (bytes): The raw content of the object.
- `obj_type` (str): The type of the object (`blob`, `tree`, `commit`, `tag`).
- `write` (bool): If True, writes the object to disk.

### Description

1. Construct the object header as: `<obj_type> <len(data)>\0`.
2. Concatenate the header and data.
3. Compute the SHA-1 hash of this concatenation.
4. If `write` is True:
   - Compress the concatenated data using zlib.
   - Write it to the `.git/objects/` directory under the path derived from the SHA.
5. Return the SHA-1 hash string.

### Example

```python
blob_content = b'Hello, Git!'
sha = hash_object(blob_content, 'blob', write=True)
print(f"Object stored with SHA: {sha}")
```

---

## read_tree(sha)

### Purpose

Reads a Git tree object and parses its entries into a structured list.

### Parameters

- `sha` (str): The SHA-1 hash of the tree object.

### Description

1. Use `read_object` to retrieve the tree object's raw data.
2. Parse the tree entries, each containing:
   - File mode (e.g., `100644` for regular file)
   - Filename
   - SHA-1 hash of the object the tree entry points to.
3. Return a list of entries, each typically represented as a tuple `(mode, filename, sha)`.

### Example

```python
entries = read_tree('d670460b4b4aece5915caf5c68d12f560a9fe3e4')
for mode, filename, sha in entries:
    print(f"{mode} {filename} {sha}")
```

---

## read_commit(sha)

### Purpose

Reads a commit object and parses its metadata and content.

### Parameters

- `sha` (str): The SHA-1 hash of the commit object.

### Description

1. Use `read_object` to get the commit object's raw data.
2. Parse the commit data, extracting fields such as:
   - Tree SHA
   - Parent commit SHAs (if any)
   - Author and committer information
   - Commit message
3. Return a dictionary or structured object representing the commit.

### Example

```python
commit_info = read_commit('f5f3693a4b0f9d1a1f8d75e4ab21f5a3e4d9a5c0')
print(commit_info['author'])
print(commit_info['message'])
```

---

## encode_object_for_pack(sha)

### Purpose

Encodes a Git object for inclusion in a pack file, preparing it for efficient storage and transfer.

### Parameters

- `sha` (str): The SHA-1 hash of the object to encode.

### Description

1. Read the object data and type using `read_object`.
2. Encode the object header for packfile format:
   - Include type and size encoded in a variable-length format.
3. Append the compressed object data.
4. Return the encoded bytes suitable for packfile inclusion.

### ASCII Diagram: Pack Object Encoding

```
+-------------------+----------------------+-----------------------+
| Header (type+size)| Compressed Object Data| (Optional delta info) |
+-------------------+----------------------+-----------------------+

Header encoding uses variable-length integer encoding:
 - Lower 4 bits: part of size
 - Next 3 bits: object type
 - MSB: indicates continuation
```

### Example

```python
pack_data = encode_object_for_pack('a1b2c3d4e5f678901234567890abcdef12345678')
with open('object.pack', 'ab') as f:
    f.write(pack_data)
```

---

This document provides foundational knowledge and practical examples to work with Git objects at a low level, enabling the implementation of higher-level Git commands and storage optimizations. For related topics on packfile creation, refer to [packfile_creation.md](./packfile_creation.md).