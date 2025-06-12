# push.md

# Pushing Commits and Objects to Remote Repositories

---

## Overview

This document details the mechanisms and functions involved in pushing commits and associated git objects from a local repository to a remote git repository. It is part of the broader "Remote Operations" section of the documentation tree, which focuses on synchronizing local repository state with remote servers.

The push operations are critical to sharing changes with collaborators, updating remote branches, and ensuring that the remote repository reflects the latest state of the local repository. This file explains the process of determining which commits and objects are missing on the remote, packaging these objects efficiently, communicating via HTTP with authentication, and updating remote references.

The functions documented here build upon core git concepts such as commits, trees, blobs, and refs, and utilize object storage, indexing, and packfile creation covered in earlier sections of the documentation.

---

## Functions

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch to a remote git repository specified by `git_url`. It uploads missing commit objects and updates the remote reference accordingly.

**Parameters:**  
- `git_url` (str): The URL of the remote git repository (e.g., `https://github.com/user/repo.git`).  
- `username` (str, optional): Username for HTTP authentication. Defaults to the environment variable `GIT_USERNAME` if not provided.  
- `password` (str, optional): Password for HTTP authentication. Defaults to the environment variable `GIT_PASSWORD` if not provided.

**Operation Steps:**  
1. Retrieve credentials either from parameters or environment variables.  
2. Obtain the SHA-1 hash of the remote `master` branch commit via `get_remote_master_hash()`.  
3. Read the local `master` branch commit hash via `get_local_master_hash()`.  
4. Determine which objects are missing on the remote by calling `find_missing_objects(local_sha1, remote_sha1)`.  
5. Notify user about updating the remote from the current commit to the new commit and how many objects will be transferred.  
6. Prepare the reference update line to send, including the old and new commit hashes and the ref name `refs/heads/master`.  
7. Create a packfile containing all missing objects using `create_pack(missing)`.  
8. Send an authenticated HTTP POST request to the remote's `/git-receive-pack` endpoint with the reference update and packfile data using `http_request()`.  
9. Parse the server's response lines via `extract_lines()` and assert successful unpacking and ref update.  
10. Return a tuple containing the remote commit hash before the push and the set of missing objects sent.

**Example Usage:**

```python
git_url = "https://github.com/example/repo.git"
username = "myusername"          # Optional if GIT_USERNAME is set
password = "mypassword"          # Optional if GIT_PASSWORD is set

remote_before, pushed_objects = push(git_url, username, password)
print(f"Pushed {len(pushed_objects)} objects. Remote was at {remote_before}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the SHA-1 commit hash string of the local `master` branch. Returns `None` if no local commits exist.

**Returns:**  
- `str` or `None`: SHA-1 hex string of the local `master` commit or `None` if unavailable.

**Operation:**  
- Reads the file `.git/refs/heads/master` and returns its contents stripped of trailing whitespace.  
- If the file does not exist, returns `None`.

**Example:**

```python
commit_hash = get_local_master_hash()
if commit_hash:
    print(f"Local master commit: {commit_hash}")
else:
    print("No commits on local master branch.")
```

---

### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Get the SHA-1 commit hash of the remote `master` branch from the remote repository.

**Parameters:**  
- `git_url`: The remote repository URL.  
- `username`: HTTP basic authentication username.  
- `password`: HTTP basic authentication password.

**Returns:**  
- `str` or `None`: SHA-1 hex string of remote `master` commit, or `None` if no commits exist remotely.

**Operation:**  
- Performs an HTTP GET request to `${git_url}/info/refs?service=git-receive-pack`.  
- Parses the response lines extracted by `extract_lines()`.  
- Validates response headers and extracts the commit hash associated with `refs/heads/master`.  
- Returns `None` if the remote has no commits (hash all zeroes).

**Example:**

```python
remote_hash = get_remote_master_hash("https://github.com/example/repo.git", "user", "pass")
print(f"Remote master commit hash: {remote_hash or 'no commits'}")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Identify the set of git objects present in the local commit graph that are missing in the remote commit graph.

**Parameters:**  
- `local_sha1` (str): SHA-1 hash of the local commit.  
- `remote_sha1` (str or None): SHA-1 hash of the remote commit or None if no remote commit.

**Returns:**  
- `set` of SHA-1 strings: Objects that need to be pushed to the remote.

**Operation:**  
- Recursively find all objects reachable from `local_sha1` using `find_commit_objects()`.  
- If `remote_sha1` is `None`, all local objects are missing.  
- Otherwise, recursively find all objects reachable from `remote_sha1`.  
- Return the set difference (local objects minus remote objects).

**Example:**

```python
missing = find_missing_objects(local_sha1, remote_sha1)
print(f"Objects missing on remote: {len(missing)}")
```

---

### `create_pack(objects)`

**Purpose:**  
Create a git packfile containing all specified git objects for efficient transfer.

**Parameters:**  
- `objects` (set of str): SHA-1 hashes of objects to include in the packfile.

**Returns:**  
- `bytes`: Raw bytes of the complete packfile.

**Operation:**  
- Constructs the packfile header with signature `PACK`, version 2, and count of objects.  
- Encodes each object using `encode_pack_object()` and concatenates them for the pack body.  
- Appends SHA-1 checksum of the entire packfile for integrity.  
- Returns the full packfile data bytes.

**Example:**

```python
pack_data = create_pack(missing_objects)
with open("objects.pack", "wb") as f:
    f.write(pack_data)
```

---

### `http_request(url, username, password, data=None)`

**Purpose:**  
Perform an HTTP request with basic authentication.

**Parameters:**  
- `url` (str): The URL to send the request to.  
- `username` (str): Username for HTTP basic auth.  
- `password` (str): Password for HTTP basic auth.  
- `data` (bytes or None): If provided, sends a POST request with this data; otherwise, a GET request.

**Returns:**  
- `bytes`: Raw response body from the server.

**Operation:**  
- Uses `urllib.request` to build an opener with HTTPBasicAuthHandler.  
- Opens the URL with the optional data payload.  
- Reads and returns the response content.

**Example:**

```python
response = http_request("https://example.com/api", "user", "pass", data=b"payload")
print(response.decode())
```

---

### `extract_lines(data)`

**Purpose:**  
Parse a Git protocol packet-line formatted byte string into a list of payload lines.

**Parameters:**  
- `data` (bytes): Raw response data from Git server containing packet lines.

**Returns:**  
- `list` of `bytes`: Extracted payload lines without length prefixes.

**Operation:**  
- Reads each packet-line by interpreting the first 4 hex chars as length.  
- Extracts the payload line accordingly, stopping at zero-length packet (`0000`).  
- Returns list of lines.

**Example:**

```python
lines = extract_lines(response_data)
for line in lines:
    print(line.decode())
```

---

### ASCII Diagram: Push Data Flow Overview

```
+------------------------+          +---------------------------+
| Local Repository       |          | Remote Repository         |
|                        |          |                           |
|  +-----------------+   |          |   +-------------------+   |
|  | Local Master    |   |          |   | Remote Master      |   |
|  | Commit SHA      |   |          |   | Commit SHA        |   |
|  +-----------------+   |          |   +-------------------+   |
|           |            |          |            ^              |
|           v            |          |            |              |
|  find_missing_objects  |          |            |              |
|           |            |          |            |              |
|           v            |          |            |              |
|  create_pack(missing)  |          |            |              |
|           |            |          |            |              |
|           v            |          |            |              |
|  HTTP POST /git-receive-pack ---->+            |              |
|           |                     unpack ok      |              |
|           |                     ok refs/heads/master          |
|           v                                            Update ref|
|  Print Update Summary                                  +---------+
+------------------------+                               |
                                                         v
                                            +--------------------------+
                                            | Remote Repository Updated |
                                            +--------------------------+
```

---

## Summary

The push functionality leverages several supporting functions to determine the state of the local and remote repositories, identify missing git objects, encode them into a packfile, and communicate with the remote repository over HTTP with authentication. The process ensures that the remote branch `master` is updated with the latest commits and associated objects from the local repository.

By understanding and utilizing these functions, users and developers can implement and debug git push operations within the `pygit` project or extend its capabilities for remote repository management.