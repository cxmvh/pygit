# push.md

# Pygit Push Command Documentation

---

## Overview

The `push.md` file documents the `push` function used in Pygit, a simplified Git implementation in Python. This function is responsible for pushing local commits from the `master` branch to a remote Git repository URL. It handles authentication, determines which objects (commits, trees, blobs) are missing on the remote repository, packages these objects into Git's packfile format, and communicates with the remote via Git's smart HTTP transport protocol.

Within the broader documentation tree, `push.md` resides under the "Remote Operations" section, emphasizing its role in synchronizing local Git history with remote repositories. It builds upon foundational operations such as repository initialization (`init`), commit creation (`commit`), object storage and retrieval, and working copy status checks (`diff`, `status`). This file serves as a crucial bridge between local repository state and remote server synchronization.

---

## Important Functions

### Function: `push`

**Signature:**

```python
def push(git_url, username=None, password=None):
```

**Description:**

Pushes the local `master` branch commits to a remote Git repository at the specified `git_url`.

- If `username` or `password` is omitted, it attempts to read them from environment variables `GIT_USERNAME` and `GIT_PASSWORD`.
- Retrieves the current commit hash (SHA-1) of the remote master branch.
- Retrieves the current commit hash of the local master branch.
- Computes which Git objects are present locally but missing remotely.
- Packs these missing objects into a Git packfile.
- Sends the packfile and update commands to the remote repository using HTTP POST to the `/git-receive-pack` endpoint.
- Validates the remote response to confirm successful unpack and reference update.

**Parameters:**

- `git_url` (str): The URL of the remote Git repository (e.g., `https://example.com/repo.git`).
- `username` (str, optional): Username for HTTP Basic Authentication.
- `password` (str, optional): Password for HTTP Basic Authentication.

**Returns:**

- Tuple `(remote_sha1, missing)` where:
  - `remote_sha1` is the SHA-1 hash string of the remote master branch before the push.
  - `missing` is a set of SHA-1 hashes of the objects sent to the remote.

**Operation Steps:**

1. Determine credentials from parameters or environment.
2. Obtain remote master commit hash using `get_remote_master_hash`.
3. Obtain local master commit hash using `get_local_master_hash`.
4. Identify missing objects with `find_missing_objects` by comparing local and remote commit object sets.
5. Print status message indicating the update.
6. Build Git protocol command lines with old and new commit hashes and reference.
7. Create a packfile containing the missing objects (`create_pack`).
8. Send the combined command lines and packfile via HTTP POST to the remote's `/git-receive-pack` endpoint (`http_request`).
9. Extract response lines (`extract_lines`) to verify success (`unpack ok` and `ok refs/heads/master`).
10. Return results.

**Example Usage:**

```python
git_url = "https://github.com/username/myrepo.git"
username = "myusername"
password = "mypassword"

remote_sha1, pushed_objects = push(git_url, username, password)
print(f"Pushed {len(pushed_objects)} objects to remote {git_url}, updated from {remote_sha1}")
```

---

### Function: `get_remote_master_hash`

**Signature:**

```python
def get_remote_master_hash(git_url, username, password):
```

**Description:**

Fetches the SHA-1 hash of the remote repository's `master` branch.

- Sends a GET request to `git_url + '/info/refs?service=git-receive-pack'`.
- Parses the Git smart HTTP protocol response.
- Returns `None` if no commits exist on the remote.

**Parameters:**

- `git_url` (str): The remote repository URL.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.

**Returns:**

- SHA-1 hex string of the remote `master` commit or `None`.

**Example Usage:**

```python
remote_sha1 = get_remote_master_hash("https://github.com/username/myrepo.git", "myusername", "mypassword")
if remote_sha1:
    print(f"Remote master commit: {remote_sha1}")
else:
    print("Remote repository has no commits.")
```

---

### Function: `get_local_master_hash`

**Signature:**

```python
def get_local_master_hash():
```

**Description:**

Reads the SHA-1 hash of the local `master` branch commit from `.git/refs/heads/master`.

Returns `None` if the local `master` branch does not exist.

**Returns:**

- SHA-1 hex string of local `master` branch commit or `None`.

**Example Usage:**

```python
local_sha1 = get_local_master_hash()
print(f"Local master commit hash: {local_sha1}")
```

---

### Function: `find_missing_objects`

**Signature:**

```python
def find_missing_objects(local_sha1, remote_sha1):
```

**Description:**

Determines which Git objects exist in the local commit but are missing on the remote commit.

- Recursively finds all objects referenced by the commits, their trees, and blobs.
- Returns the set difference (local objects minus remote objects).

**Parameters:**

- `local_sha1` (str): SHA-1 hash of local commit.
- `remote_sha1` (str or None): SHA-1 hash of remote commit or `None` if none.

**Returns:**

- Set of SHA-1 hex strings of missing objects.

**Example Usage:**

```python
missing_objs = find_missing_objects(local_sha1, remote_sha1)
print(f"Missing objects count: {len(missing_objs)}")
```

---

### Function: `create_pack`

**Signature:**

```python
def create_pack(objects):
```

**Description:**

Creates a Git packfile containing all given objects.

- Constructs a packfile header.
- Encodes each object with a variable-length header and compressed data.
- Appends SHA-1 checksum of the packfile contents.
- Returns the full packfile as bytes.

**Parameters:**

- `objects` (set): Set of SHA-1 strings of objects to include.

**Returns:**

- Bytes representing the complete packfile.

**Example Usage:**

```python
pack_data = create_pack(missing)
with open("objects.pack", "wb") as f:
    f.write(pack_data)
```

---

### Function: `build_lines_data`

**Signature:**

```python
def build_lines_data(lines):
```

**Description:**

Constructs Git protocol packet lines to send commands to the server.

- Each line is prefixed with a 4-digit hex length (line length + 5).
- Lines end with a newline character.
- Ends with a flush packet (`0000`).

**Parameters:**

- `lines` (list of bytes): List of command lines.

**Returns:**

- Bytes of all concatenated packet lines ready for transmission.

**Example Usage:**

```python
lines = [b"0000000000000000000000000000000000000000 1234567890abcdef1234567890abcdef12345678 refs/heads/master\x00 report-status"]
data = build_lines_data(lines)
```

---

### Function: `http_request`

**Signature:**

```python
def http_request(url, username, password, data=None):
```

**Description:**

Performs an authenticated HTTP request.

- Uses Basic HTTP Authentication.
- Sends a GET request if `data` is `None`.
- Sends a POST request if `data` is provided.

**Parameters:**

- `url` (str): URL to request.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.
- `data` (bytes, optional): POST data.

**Returns:**

- Response body as bytes.

**Example Usage:**

```python
response = http_request("https://github.com/username/myrepo/git-receive-pack", "user", "pass", data=pack_data)
```

---

### Function: `extract_lines`

**Signature:**

```python
def extract_lines(data):
```

**Description:**

Parses the raw Git protocol response data into a list of lines.

- Each line is prefixed with a 4-digit hex length.
- A length of zero indicates a flush packet and ends parsing.

**Parameters:**

- `data` (bytes): Raw response bytes.

**Returns:**

- List of lines as bytes.

**Example Usage:**

```python
lines = extract_lines(response_data)
for line in lines:
    print(line)
```

---

## ASCII Diagram: Push Flow Overview

```
+----------------------+          +-----------------------+          +-----------------------+
|  Local Repository    |          |  Network Transfer      |          |  Remote Repository     |
|                      |          |                       |          |                       |
|  +---------------+   |          |  +-----------------+  |          |  +-----------------+  |
|  | Local master  |   |  Push    |  | HTTP POST to     |  | Receive  |  | Remote master    |  |
|  | commit SHA-1  |--------->|  | /git-receive-pack|--------->|  | commit SHA-1    |  |
|  +---------------+   |          |  +-----------------+  |          |  +-----------------+  |
|                      |          |                       |          |                       |
|  +---------------+   |          |                       |          |                       |
|  | Missing objs   |--|          |                       |          |                       |
|  | packfile data  |            |                       |          |                       |
|  +---------------+            |                       |          |                       |
+----------------------+          +-----------------------+          +-----------------------+
```

---

# Summary

This document explains the mechanism by which Pygit pushes local commits and associated Git objects to a remote repository over HTTP. The `push` function orchestrates authentication, object discovery, packaging, and network communication to update the remote `master` branch efficiently. Understanding these functions is essential for extending or troubleshooting remote synchronization in Pygit.