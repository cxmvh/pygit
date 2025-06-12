# push.md

# Push Command and Related Functions

---

## Overview

The `push.md` document details the implementation and usage of the `push` command within the `pygit` project, a minimal Git-like version control system written in Python. This command is responsible for uploading local commits and objects on the `master` branch to a remote repository. The document situates the `push` command within the broader context of Git command operations and their supporting internal mechanisms, including HTTP authentication, object packing, and communication protocols.

The `push` command is a critical operation enabling synchronization between a local repository and its remote counterpart, ensuring that changes committed locally are transmitted securely and efficiently. This document covers the main `push` function and its helper functions that handle remote hash retrieval, packfile creation, communication with the remote server, and verification of the push operation's success.

---

## Function Documentation

---

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch commits and their associated Git objects to a remote repository specified by the URL `git_url`. 

If `username` or `password` are not provided, the function attempts to use environment variables `GIT_USERNAME` and `GIT_PASSWORD`.

**Parameters:**  
- `git_url` (`str`): URL of the remote Git repository to push to.  
- `username` (`str`, optional): Username for HTTP authentication.  
- `password` (`str`, optional): Password for HTTP authentication.

**Operation:**  
1. Retrieve the remote repository's current `master` commit hash (`remote_sha1`).  
2. Retrieve the local repository's current `master` commit hash (`local_sha1`).  
3. Determine which objects exist locally but are missing remotely (`missing`).  
4. Print a summary of the update operation.  
5. Construct the Git protocol lines to initiate the push with `report-status` capability.  
6. Build the data payload, including the lines and a packfile containing the missing objects.  
7. Perform an authenticated HTTP POST request to the remote's `/git-receive-pack` endpoint.  
8. Extract and validate the server response lines to confirm a successful push.  
9. Return a tuple containing the remote's previous commit hash and the set of pushed objects.

**Example Usage:**  
```python
remote_url = "https://example.com/myrepo.git"
user = "alice"
passwd = "s3cret"

remote_sha1, pushed_objects = push(remote_url, username=user, password=passwd)
print(f"Pushed {len(pushed_objects)} objects to remote commit {remote_sha1}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash (`SHA-1` string) of the local `master` branch.

**Parameters:**  
None.

**Operation:**  
- Read the contents of `.git/refs/heads/master`.  
- Return the commit hash as a string or `None` if the file does not exist (no commits yet).

**Example Usage:**  
```python
local_master = get_local_master_hash()
if local_master:
    print(f"Local master commit: {local_master}")
else:
    print("No commits on local master branch.")
```

---

### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Fetch the commit hash of the remote repository's `master` branch via the Git smart HTTP protocol.

**Parameters:**  
- `git_url` (`str`): URL of the remote Git repository.  
- `username` (`str`): Username for authentication.  
- `password` (`str`): Password for authentication.

**Operation:**  
- Send an authenticated HTTP GET request to `git_url + "/info/refs?service=git-receive-pack"`.  
- Parse the service response lines using `extract_lines`.  
- Verify the response format and extract the SHA-1 of the remote `master` branch.  
- Return the SHA-1 as a string or `None` if the remote has no commits.

**Example Usage:**  
```python
remote_sha1 = get_remote_master_hash("https://example.com/myrepo.git", "alice", "s3cret")
print(f"Remote master commit hash: {remote_sha1}")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Identify which commit objects exist locally but are missing on the remote repository.

**Parameters:**  
- `local_sha1` (`str`): SHA-1 of the local commit to push.  
- `remote_sha1` (`str` or `None`): SHA-1 of the remote commit.

**Operation:**  
- Recursively find all objects reachable from the local commit.  
- If remote commit is `None`, return all local objects (no commits remotely).  
- Otherwise, recursively find all objects from the remote commit and compute the set difference.

**Example Usage:**  
```python
missing_objects = find_missing_objects(local_sha1="abc123...", remote_sha1="def456...")
print(f"Objects to push: {len(missing_objects)}")
```

---

### `build_lines_data(lines)`

**Purpose:**  
Convert a list of lines into a byte string formatted according to the Git pkt-line protocol for transmission to the remote server.

**Parameters:**  
- `lines` (`list` of `bytes`): Lines to encode.

**Operation:**  
- For each line, prepend a 4-byte hex length prefix (length of line + 5).  
- Append a newline character after each line.  
- Append a final flush packet `b'0000'` to indicate the end of the input.

**Example Usage:**  
```python
lines = [b"0000000000000000000000000000000000000000 0123456789abcdef refs/heads/master\x00 report-status"]
data = build_lines_data(lines)
```

---

### `create_pack(objects)`

**Purpose:**  
Create a Git packfile containing all the objects with the given SHA-1 hashes.

**Parameters:**  
- `objects` (`set` of `str`): SHA-1 hashes of the objects to pack.

**Operation:**  
- Build the packfile header with signature, version (2), and object count.  
- Encode each object into the packfile body using `encode_pack_object`.  
- Concatenate header, body, and SHA-1 checksum of the packfile contents.  
- Return the full packfile data as bytes.

**Example Usage:**  
```python
packfile_data = create_pack(missing_objects)
with open("objects.pack", "wb") as f:
    f.write(packfile_data)
```

---

### `http_request(url, username, password, data=None)`

**Purpose:**  
Make an authenticated HTTP request to the specified URL.

**Parameters:**  
- `url` (`str`): Target URL.  
- `username` (`str`): Username for HTTP Basic Authentication.  
- `password` (`str`): Password for HTTP Basic Authentication.  
- `data` (`bytes`, optional): Data to POST; if `None`, a GET request is made.

**Operation:**  
- Create a password manager and authentication handler.  
- Build an opener with authentication.  
- Open the URL with optional data.  
- Return the response content as bytes.

**Example Usage:**  
```python
response = http_request("https://example.com/repo/git-receive-pack", "alice", "s3cret", data=packfile_data)
```

---

### `extract_lines(data)`

**Purpose:**  
Parse server response data following the Git pkt-line format into a list of byte lines.

**Parameters:**  
- `data` (`bytes`): Raw response data.

**Operation:**  
- Read 4-byte hex length prefixes to determine line sizes.  
- Extract each line according to the length prefix.  
- Stop when a flush packet (`0000`) or end of data is reached.

**Example Usage:**  
```python
lines = extract_lines(response_data)
for line in lines:
    print(line.decode())
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encode a single Git object into the packfile format, including a variable-length header and compressed data.

**Parameters:**  
- `obj` (`str`): SHA-1 hash of the object.

**Operation:**  
- Read the object type and data.  
- Compute the variable-length header encoding type and size.  
- Compress the object data with zlib.  
- Concatenate header and compressed data and return as bytes.

**Example Usage:**  
```python
pack_object_bytes = encode_pack_object("abc123...")
```

---

## ASCII Diagram: Push Operation Flow

```
+------------------+
| Local Repository  |
|  master commit    |
|  and objects      |
+--------+---------+
         |
         | get_local_master_hash()
         |
         v
+------------------+         +---------------------+
|   push()         |         | Remote Repository    |
|                  |         |                     |
|  get_remote_master_hash()  |                     |
|  find_missing_objects()    |                     |
|  build_lines_data()        |                     |
|  create_pack()             |                     |
|  http_request() ---------->|                     |
+------------------+         +---------------------+
         |                             |
         |<----------------------------|
         |   server response lines     |
         v
+------------------+
| Verify response   |
| Return pushed objs|
+------------------+
```

---

## Summary

This document covers the implementation of the `push` command and its supporting functions within the `pygit` project. The `push` command handles authentication, object discovery, packfile creation, and communication with the remote Git server using the Git smart HTTP protocol. Understanding the interplay of these functions provides insight into how a minimal Git client synchronizes local changes with a remote repository.