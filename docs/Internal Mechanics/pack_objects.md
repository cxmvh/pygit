# pack_objects.md

# Creating and Encoding Packfiles for Pushing Objects

---

## Overview

The `pack_objects.md` file documents the processes and functions involved in creating and encoding Git packfiles, which are essential for efficiently transferring multiple Git objects during a push operation. Packfiles bundle multiple Git objects into a single compressed file, reducing redundancy and network transfer size. This file is part of the *Internal Mechanics* section of the documentation tree and supports the `pygit.push` command flow by handling the packaging of missing objects for a remote repository update. Understanding packfile creation is crucial for developers working on the internals of Git operations, especially push and fetch commands.

---

## Function Documentation

---

### `create_pack(objects)`

**Purpose:**  
Create a Git packfile containing all objects specified by their SHA-1 hashes. This packfile is a compact representation of multiple Git objects, used to efficiently transfer data during push operations.

**Parameters:**  
- `objects` (set of str): A set of SHA-1 hex string hashes representing Git objects to be packed.

**Returns:**  
- `bytes`: The complete byte content of the packfile including the header, encoded objects, and SHA-1 checksum.

**Operation Steps:**  
1. Prepare a packfile header with:
   - The ASCII string `PACK`
   - Version number `2`
   - Number of objects packed

2. Sort the objects by their SHA-1 hash for consistency.

3. Encode each object using `encode_pack_object()` which compresses and formats the object data.

4. Concatenate the header and all encoded objects.

5. Append the SHA-1 checksum calculated over the entire packfile content (header + objects).

6. Return the assembled packfile data.

**Example Usage:**

```python
missing_objects = {'e83c5163316f89bfbde7d9ab23ca2e25604af29e', 'fbc1a68f9b1d2a5f5e9e9a9a1c3e2f650c9b8d1a'}
packfile_data = create_pack(missing_objects)

# packfile_data can now be sent to the remote server during push
with open('objects.pack', 'wb') as f:
    f.write(packfile_data)
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encode a single Git object for inclusion in a packfile. The encoding includes a variable-length header describing the object type and size, followed by the compressed object data.

**Parameters:**  
- `obj` (str): SHA-1 hash (hex string) of the Git object to encode.

**Returns:**  
- `bytes`: The encoded byte string representing the packfile entry for the object.

**Operation Steps:**  
1. Read the object type and raw data using `read_object(obj)`.

2. Determine the numeric type code from an internal `ObjectType` enumeration.

3. Calculate the size of the object data.

4. Construct the first header byte:
   - High 3 bits: object type
   - Low 4 bits: lower 4 bits of object size

5. Encode the remaining size bits into continuation bytes if necessary, setting the MSB of each byte except the last.

6. Compress the object data using zlib compression.

7. Concatenate the header bytes and compressed data.

**Example Usage:**

```python
encoded_data = encode_pack_object('e83c5163316f89bfbde7d9ab23ca2e25604af29e')

# encoded_data contains the variable-length header + compressed object data
with open('object.packdata', 'wb') as f:
    f.write(encoded_data)
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Read a Git object identified by a SHA-1 prefix from the object store, returning its type and raw data.

**Parameters:**  
- `sha1_prefix` (str): A prefix string of the SHA-1 hash identifying the object.

**Returns:**  
- `(str, bytes)`: Tuple containing the object type (e.g., 'commit', 'tree', 'blob') and the raw decompressed data bytes.

**Preconditions:**  
- The SHA-1 prefix must uniquely identify one object in the `.git/objects` directory.

**Operation Steps:**  
1. Locate the full object file path using `find_object(sha1_prefix)`.

2. Read the compressed data from the file.

3. Decompress the data using zlib.

4. Parse the header to extract the object type and size.

5. Extract and return the object data bytes.

**Example Usage:**

```python
obj_type, data = read_object('e83c5163')
print(f"Object type: {obj_type}")
print(f"Data length: {len(data)} bytes")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Identify which objects exist in the local commit history but are missing on the remote repository, enabling incremental push of only missing objects.

**Parameters:**  
- `local_sha1` (str): SHA-1 hash of the local commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote commit or `None` if no remote commits exist.

**Returns:**  
- `set` of SHA-1 strings representing missing objects on the remote.

**Operation Steps:**  
1. Retrieve all objects reachable from the local commit using `find_commit_objects(local_sha1)`.

2. If the remote SHA-1 is `None`, return all local objects (remote is empty).

3. Otherwise, retrieve all objects reachable from the remote commit.

4. Calculate the set difference: local objects minus remote objects.

**Example Usage:**

```python
missing = find_missing_objects('localcommitsha1', 'remotecommitsha1')
print(f"Objects to push: {len(missing)}")
```

---

### ASCII Diagram: Packfile Structure

```
+-------------------+
| PACK (4 bytes)    |  <- Packfile header signature
+-------------------+
| Version (4 bytes) |  <- Packfile version (e.g., 2)
+-------------------+
| Object Count (4)  |  <- Number of objects in packfile
+-------------------+
| Object 1          |  <- Encoded object (header + compressed data)
+-------------------+
| Object 2          |
+-------------------+
| ...               |
+-------------------+
| Object N          |
+-------------------+
| SHA-1 checksum    |  <- SHA-1 of all previous bytes in packfile
+-------------------+
```

---

### Integration in Push Workflow

The packfile creation is part of the `pygit.push` command execution:

1. Determine the remote master commit hash with `get_remote_master_hash`.

2. Determine the local master commit hash with `get_local_master_hash`.

3. Use `find_missing_objects` to identify objects to send.

4. Build the packfile data with `create_pack(missing_objects)`.

5. Send the packfile data in the push request to the remote repository.

---

# End of pack_objects.md documentation content.