# Pack File Creation and Object Encoding

This document describes functions related to creating Git pack files, encoding Git objects for efficient transmission during push operations, and identifying which objects are missing on the remote. These functions are part of the **Push and Remote Operations** section and support the `pygit.push` flow to enable pushing commits to remote repositories.

Pack files aggregate multiple Git objects into a single compressed file for efficient network transfer and storage. This document provides technical details on how pack files are created, how individual objects are encoded within them, and how missing objects are determined during push operations.

---

## Functions

### `create_pack(objects)`

Create a Git pack file containing all objects specified by their SHA-1 hashes.

**Signature:**

```python
def create_pack(objects)
```

**Parameters:**

- `objects` (`set` of `str`): A set of SHA-1 hex string hashes representing the Git objects to be packed.

**Returns:**

- `bytes`: The complete pack file data including header, encoded objects, and SHA-1 checksum.

**Description:**

This function constructs a Git pack file in the following steps:

1. Creates a pack file header containing:
   - The ASCII string `'PACK'`
   - The pack format version number (currently 2)
   - The number of objects included in the pack

2. Encodes each object using `encode_pack_object` and concatenates their encoded bytes.

3. Concatenates header and body, then computes a SHA-1 checksum of this concatenated data.

4. Appends the checksum to the pack data.

5. Returns the full bytes of the pack file ready for transmission or storage.

**Example Usage:**

```python
missing_objects = {'a1b2c3d4...', 'f5e6d7c8...'}
pack_data = create_pack(missing_objects)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
```

---

### `encode_pack_object(obj)`

Encode a single Git object for inclusion in a pack file.

**Signature:**

```python
def encode_pack_object(obj)
```

**Parameters:**

- `obj` (`str`): SHA-1 hash string identifying the Git object to encode.

**Returns:**

- `bytes`: The encoded byte sequence representing the object in pack file format.

**Description:**

This function performs the following steps:

1. Reads the object type and raw data using `read_object`.

2. Encodes the object header using a variable-length format:
   - The low 4 bits of the first header byte encode the low bits of the object's size.
   - The next 3 bits encode the object type (commit, tree, blob, etc.).
   - The most significant bit signals if more header bytes follow.
   - The size is shifted right by 4 bits and the process continues for additional bytes until size is fully encoded.

3. Compresses the object data using zlib compression.

4. Returns the concatenation of the header bytes and compressed data.

**Example Usage:**

```python
encoded_obj = encode_pack_object('a1b2c3d4e5f6...')
# encoded_obj can be included in a pack file body
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

Determine which objects exist locally but are missing on the remote repository.

**Signature:**

```python
def find_missing_objects(local_sha1, remote_sha1)
```

**Parameters:**

- `local_sha1` (`str` or `None`): SHA-1 hash string of the local master branch commit.
- `remote_sha1` (`str` or `None`): SHA-1 hash string of the remote master branch commit, or `None` if remote has no commits.

**Returns:**

- `set` of `str`: A set of SHA-1 hash strings representing objects missing on the remote.

**Description:**

This function works by:

1. Recursively finding all objects reachable from the local commit using `find_commit_objects`.

2. If the remote commit exists, finding all objects reachable from the remote commit.

3. Computing the set difference to identify objects present locally but missing remotely.

If the remote has no commits (`remote_sha1` is `None`), all local objects are considered missing.

**Example Usage:**

```python
local_commit = 'abc123...'
remote_commit = 'def456...'
missing = find_missing_objects(local_commit, remote_commit)
print(f'Missing {len(missing)} objects on remote.')
```

---

### Supporting ASCII Diagram: Pack File Structure

```
+-------------------+
| Pack Header       |
| ----------------- |
| 'PACK' (4 bytes)  |
| Version (4 bytes) |
| Object count (4b) |
+-------------------+
| Encoded Object 1  |--> Variable-length header + compressed data
+-------------------+
| Encoded Object 2  |
+-------------------+
|       ...         |
+-------------------+
| SHA-1 Checksum    | 20 bytes, covers all previous bytes
+-------------------+
```

The pack file begins with a fixed header, followed by encoded Git objects, each compressed individually. The file ends with a SHA-1 checksum verifying integrity.

---

## Related Functions and Flow Context

- `create_pack` is called in the `pygit.push` flow to build the pack file for transmission.
- `encode_pack_object` is used internally by `create_pack`.
- `find_missing_objects` helps determine which objects need to be included in the pack.
- Other functions in the `pygit.push` flow handle fetching remote hashes, HTTP communication, and verifying push success.

For more details on `pygit.push` and related remote interaction, see [push.md](push.md) and [remote_interaction.md](remote_interaction.md).

---

## Summary

This documentation covered key functions for pack file creation, object encoding, and detecting missing objects in the push process. Understanding these functions is essential for implementing efficient Git push operations and ensuring correct object synchronization between local and remote repositories.