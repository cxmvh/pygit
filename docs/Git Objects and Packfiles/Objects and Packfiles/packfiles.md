# packfiles.md

# Creating Pack Files with `create_pack` and Related Functions for Efficient Object Transfer

---

## Overview

This document covers the mechanisms for creating Git pack files using the `create_pack` function and its related utilities. Pack files are compressed bundles of Git objects (commits, trees, blobs) that enable efficient transfer and storage of repository data, especially during operations like pushing to a remote repository. Within the broader documentation tree, this file is part of the **Git Objects and Packfiles** section, nested under **Objects and Packfiles**. It complements other documentation on object reading, writing, and encoding by focusing on the aggregation and packaging of multiple objects into a single data stream suitable for network transmission or disk storage.

The functions herein are primarily used by the `push` command flow (`pygit.push`), which requires the creation of pack files to upload missing objects to remote repositories. Understanding pack file creation is essential for grasping how Git optimizes data synchronization and minimizes bandwidth and storage overhead.

---

## Function Documentation

### `create_pack(objects)`

**Purpose:**  
Create a Git pack file that contains all objects specified by their SHA-1 hashes. The pack file format combines a header, a sequence of encoded objects, and a SHA-1 checksum of the entire pack data. This single binary blob can then be transmitted to a remote server or stored efficiently.

**Parameters:**  
- `objects` (`set` of `str`): A set of SHA-1 hexadecimal strings representing the Git objects to include in the pack.

**Operation:**  
1. Prepare the pack file header:
   - Magic bytes `"PACK"` (4 bytes) to identify the file as a pack file.
   - Pack version number (`2`, 4 bytes, network byte order).
   - Number of objects to pack (4 bytes, network byte order).
2. For each object in sorted order:
   - Encode the object using `encode_pack_object`, which compresses the object data with a variable-length header.
3. Concatenate the header and all encoded objects into a single byte string.
4. Compute the SHA-1 hash of the entire concatenated contents.
5. Append this SHA-1 checksum to the data.
6. Return the complete byte string representing the pack file.

**Returns:**  
`bytes` — The complete binary data of the pack file.

**Example Usage:**
```python
missing_objects = {'a1b2c3d4e5f6...', 'b2c3d4e5f6a7...'}  # Set of SHA-1 hashes
pack_data = create_pack(missing_objects)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
print("Pack file created with {} objects.".format(len(missing_objects)))
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encode a single Git object for inclusion in a pack file. This involves creating a variable-length header based on the object type and size, followed by the zlib-compressed object data.

**Parameters:**  
- `obj` (`str`): SHA-1 hexadecimal hash of the object to encode.

**Operation:**  
1. Read the object data and type using `read_object`.
2. Map the object type (`commit`, `tree`, `blob`) to its numeric code.
3. Create the first byte of the header combining the object type and the low 4 bits of the object size.
4. Use a variable-length encoding scheme to encode the remaining bits of the size, setting continuation bits as needed.
5. Compress the object data using zlib compression.
6. Concatenate the header bytes and compressed data bytes.

**Returns:**  
`bytes` — The encoded object ready for inclusion in a pack file.

**Example Usage:**
```python
sha1 = 'a1b2c3d4e5f6...'
encoded_obj = encode_pack_object(sha1)
# encoded_obj can be concatenated with other encoded objects in a pack
```

---

### Related Concepts and Flow in Pack Creation

When pushing to a remote repository, the local Git client:

- Determines the latest commit hash on the remote (`get_remote_master_hash`).
- Determines the latest commit hash locally (`get_local_master_hash`).
- Finds missing objects that are present locally but not remotely (`find_missing_objects`).
- Creates a pack file containing all these missing objects (`create_pack`).
- Sends the pack file to the remote server.

This sequence optimizes network usage by transferring only the necessary objects in a compressed and packed format.

---

## ASCII Diagram: Pack File Structure

Below is a simplified ASCII diagram illustrating the structure of a Git pack file created by `create_pack`:

```
+----------------------+-------------------------+------------------+
| Pack Header (12 bytes)| Pack Objects (variable) | SHA-1 Checksum (20 bytes) |
+----------------------+-------------------------+------------------+

Pack Header:
+------+----------+----------------+
| 'PACK' (4 bytes) | Version (4 bytes) | Number of objects (4 bytes) |
+------+----------+----------------+

Pack Objects:
+------------------------+------------------------+ ... +
| Encoded Object 1       | Encoded Object 2       | ... |
+------------------------+------------------------+ ... +

Encoded Object:
+-------------------+-----------------------+
| Variable-length   | Compressed Object Data |
| header (size+type) | (zlib compressed)      |
+-------------------+-----------------------+
```

---

## Summary

- `create_pack` bundles multiple Git objects into a single pack file with a structured header and checksum.
- `encode_pack_object` prepares individual objects for this packaging by compressing and prefixing them with a size/type header.
- Together, these functions enable efficient transfer of Git repository data, especially during push operations.
- Understanding these pack file internals provides insight into Git’s data optimization strategies.

---

## Additional References

- See `pygit.push` for how `create_pack` integrates into the push workflow.
- Refer to `encode_pack_object` and `read_object` for object encoding and reading details.
- For reading and writing individual objects, consult `object_reading.md` and `object_encoding.md`.
- For packfile usage and Git object management, see the broader **Git Objects and Packfiles** documentation section.