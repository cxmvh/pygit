# Push Operation Documentation

This document provides a detailed reference for the main push operation and its helper functions used to push local commits from a Git repository to a remote repository. It is part of the **Push and Remote Operations** section in the overall Git implementation documentation. This file explains how local commit objects are identified, packed, and sent to a remote Git server, covering authentication, network communication, and error checking to ensure synchronization of branches.

---

## Overview

The push operation is a critical mechanism in Git that transfers commits from a local repository to a remote repository. This process involves identifying which commits and related objects are missing on the remote end, packaging these objects efficiently into a pack file, and sending them over an authenticated connection.

Within the broader documentation tree, this file complements other documents such as `remote_interaction.md` (covering remote communication protocols) and `pack.md` (covering pack file creation and encoding). Together, they enable a full understanding of how local changes propagate to remote repositories in Git.

---

## Function Documentation

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch commits to a remote Git repository at the given URL.

**Parameters:**  
- `git_url` (str): The URL of the remote Git repository to push to.  
- `username` (str, optional): Username for HTTP authentication. Defaults to the environment variable `GIT_USERNAME`.  
- `password` (str, optional): Password for HTTP authentication. Defaults to the environment variable `GIT_PASSWORD`.

**Operation Steps:**  
1. Retrieve authentication credentials, either from parameters or environment variables.  
2. Fetch the current commit hash of the remote `master` branch using `get_remote_master_hash`.  
3. Get the local `master` branch commit hash via `get_local_master_hash`.  
4. Determine the set of commit objects missing on the remote by comparing local and remote commit trees using `find_missing_objects`.  
5. Print a status message showing the update from the remote commit to the local commit and the number of objects to push.  
6. Build the protocol lines to update the remote ref with `build_lines_data`.  
7. Create a pack file containing the missing objects with `create_pack`.  
8. Send the push request to the remote's `/git-receive-pack` endpoint using `http_request`.  
9. Extract and verify the response lines for successful unpacking and ref update confirmation.  
10. Return a tuple `(remote_sha1, missing)` indicating the previous remote commit hash and the set of pushed objects.

**Example Usage:**
```python
git_url = "https://example.com/myrepo.git"
username = "alice"
password = "secret"
remote_commit, pushed_objects = push(git_url, username, password)
print(f"Pushed {len(pushed_objects)} objects to remote commit {remote_commit}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash string of the local `master` branch.

**Returns:**  
- A 40-character SHA-1 hex string of the local `master` branch commit, or `None` if no commit exists.

**Operation:**  
- Reads the contents of the file `.git/refs/heads/master`.  
- Returns the commit hash stored there, or `None` if the file does not exist.

**Example Usage:**
```python
local_commit = get_local_master_hash()
if local_commit:
    print(f"Local master commit: {local_commit}")
else:
    print("No commits on local master branch yet.")
```

---

### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Fetch the current commit hash of the remote `master` branch.

**Parameters:**  
- `git_url` (str): Remote repository URL.  
- `username` (str): Username for authentication.  
- `password` (str): Password for authentication.

**Returns:**  
- A 40-character SHA-1 hex string of the remote `master` branch commit, or `None` if no commits exist remotely.

**Operation:**  
- Sends an authenticated HTTP GET request to `<git_url>/info/refs?service=git-receive-pack`.  
- Parses the response to extract the hash of the remote `master` branch.  
- Returns `None` if the remote branch is empty (all zeros hash).

**Example Usage:**
```python
remote_commit = get_remote_master_hash("https://example.com/myrepo.git", "alice", "secret")
print(f"Remote commit hash: {remote_commit or 'no commits'}")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Identify the set of commit objects present locally but missing on the remote.

**Parameters:**  
- `local_sha1` (str): SHA-1 hash of the local commit.  
- `remote_sha1` (str or None): SHA-1 hash of the remote commit, or `None` if remote is empty.

**Returns:**  
- A set of SHA-1 hex strings representing objects missing on the remote.

**Operation:**  
- Recursively finds all object hashes reachable from the local commit using `find_commit_objects`.  
- Does the same for the remote commit if it exists.  
- Returns the set difference: objects in local but not in remote.

**Example Usage:**
```python
missing = find_missing_objects(local_commit, remote_commit)
print(f"Objects missing on remote: {len(missing)}")
```

---

### `create_pack(objects)`

**Purpose:**  
Create a Git pack file containing all specified objects, suitable for network transfer.

**Parameters:**  
- `objects` (set): Set of SHA-1 hex strings of objects to include.

**Returns:**  
- Byte string representing the complete pack file.

**Operation:**  
- Begins with a pack file header (`PACK` signature, version 2, count of objects).  
- Encodes each object using `encode_pack_object` and concatenates the results.  
- Appends a SHA-1 checksum of the entire pack content.

**Example Usage:**
```python
pack_data = create_pack(missing_objects)
with open("packfile.pack", "wb") as f:
    f.write(pack_data)
```

---

### `http_request(url, username, password, data=None)`

**Purpose:**  
Perform an authenticated HTTP request to a remote server with optional POST data.

**Parameters:**  
- `url` (str): Target URL.  
- `username` (str): Username for HTTP Basic Authentication.  
- `password` (str): Password for HTTP Basic Authentication.  
- `data` (bytes, optional): POST data to send. If `None`, a GET request is made.

**Returns:**  
- Response body as bytes.

**Operation:**  
- Sets up HTTP Basic Authentication handlers.  
- Opens the URL with the given data.  
- Returns the response content.

**Example Usage:**
```python
response = http_request("https://example.com/git-receive-pack", "alice", "secret", data=pack_data)
print(f"Server response length: {len(response)} bytes")
```

---

### `extract_lines(data)`

**Purpose:**  
Parse packet-line encoded protocol data from the server into individual lines.

**Parameters:**  
- `data` (bytes): Raw response data.

**Returns:**  
- List of byte strings, each representing a single line in the protocol.

**Operation:**  
- Reads length-prefixed lines (4 hex digits length + payload).  
- Stops parsing on zero-length line (`0000`).

**Example Usage:**
```python
lines = extract_lines(response_data)
for line in lines:
    print(line.decode())
```

---

### `build_lines_data(lines)`

**Purpose:**  
Construct packet-line encoded data from a list of lines to send to the server.

**Parameters:**  
- `lines` (list of bytes): Lines to encode.

**Returns:**  
- Byte string of packet-line encoded data including flush packet.

**Operation:**  
- For each line, prepends a 4-byte hex length field.  
- Appends a flush packet (`0000`) at the end.

**Example Usage:**
```python
lines = [b'0000000000000000000000000000000000000000 1234567890abcdef refs/heads/master\x00 report-status']
data = build_lines_data(lines)
```

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively collect all Git objects (commits, trees, blobs) reachable from a commit.

**Parameters:**  
- `commit_sha1` (str): SHA-1 hash of the commit object.

**Returns:**  
- Set of SHA-1 hashes of all objects referenced by the commit.

**Operation:**  
- Reads the commit object.  
- Extracts the tree hash and parent commits.  
- Recursively collects objects from the tree and parents.

**Example Usage:**
```python
objects = find_commit_objects("abcd1234...")
print(f"Found {len(objects)} objects in commit tree")
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encode a single Git object into the pack file format.

**Parameters:**  
- `obj` (str): SHA-1 hash of the object.

**Returns:**  
- Bytes containing the variable-length header and zlib-compressed object data.

**Operation:**  
- Reads the object type and data.  
- Constructs a variable-length header encoding the type and size.  
- Follows with compressed data.

**Example Usage:**
```python
encoded = encode_pack_object("abcd1234...")
print(f"Encoded object size: {len(encoded)} bytes")
```

---

## ASCII Diagram: Push Operation Flow

```
+----------------+          +----------------+          +------------------+
| Local Git Repo |          | Remote Git Repo|          | Remote Git Server |
+----------------+          +----------------+          +------------------+
       |                           |                             |
       | 1. get_local_master_hash()|                             |
       |-------------------------->|                             |
       |                           |                             |
       | 2. get_remote_master_hash()                            |
       |------------------------------------------------------->|
       |                           |                             |
       | 3. find_missing_objects()  |                             |
       | (local - remote commits)   |                             |
       |                           |                             |
       | 4. create_pack(missing)    |                             |
       |                           |                             |
       | 5. build_lines_data()      |                             |
       |                           |                             |
       | 6. http_request(push)      |                             |
       |------------------------------------------------------->|
       |                           |                             |
       | 7. receive response        |                             |
       |<-------------------------------------------------------|
       |                           |                             |
       | 8. verify success          |                             |
       |                           |                             |
```

---

# See Also

- [`remote_interaction.md`](remote_interaction.md): Details on remote repository communication and authentication.  
- [`pack.md`](pack.md): Functions related to pack file creation and encoding.  
- [`commit.md`](../Commit%20Management/commit.md): How commits are created and managed locally.

---

This completes the documentation for the push operation and its helper functions that enable pushing local commits to a remote Git repository.