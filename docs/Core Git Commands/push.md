# push.md

# Push Operation Documentation

---

## Overview

The `push.md` document details the implementation and usage of the **push operation** in the pygit system. This operation is responsible for updating a remote Git repository by transferring local commits and associated objects to the remote server. Located under the **Core Git Commands** and **Remote Operations** sections in the documentation tree, this file plays a critical role in enabling collaboration and synchronization between local and remote repositories. It builds on foundational concepts such as repository state, object handling, and index management to ensure a smooth and secure update of the remote repository's `master` branch.

---

## Function Documentation

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch updates to a remote Git repository located at `git_url`. If `username` or `password` are not provided, they are fetched from environment variables `GIT_USERNAME` and `GIT_PASSWORD` respectively.

**Parameters:**  
- `git_url` (str): The URL of the remote Git repository to push to.  
- `username` (str, optional): Username for HTTP authentication. Defaults to environment variable if not provided.  
- `password` (str, optional): Password for HTTP authentication. Defaults to environment variable if not provided.

**Returns:**  
Tuple `(remote_sha1, missing)` where:  
- `remote_sha1` is the SHA-1 string representing the remote master commit before push (or `None` if no commits remotely).  
- `missing` is a set of SHA-1 hashes of objects that were sent to the remote because they were missing.

**Operation Flow:**  
1. Retrieve credentials if not provided.  
2. Get the remote master commit hash via `get_remote_master_hash`.  
3. Get the local master commit hash via `get_local_master_hash`.  
4. Determine which objects exist locally but are missing remotely using `find_missing_objects`.  
5. Prepare Git protocol lines announcing the update from remote to local master commit, including a "report-status" capability.  
6. Build the payload combining protocol lines and a packfile containing missing objects (`create_pack`).  
7. Send an authenticated HTTP POST request to the remote `/git-receive-pack` endpoint using `http_request`.  
8. Parse the response lines via `extract_lines` and validate expected success messages (`unpack ok` and `ok refs/heads/master`).  
9. Return the remote SHA-1 and the set of missing objects sent.

**Example Usage:**

```python
# Push the local master branch to a remote repository
remote_url = "https://github.com/example/repo.git"
remote_commit, sent_objects = push(remote_url, username="gituser", password="gitpass")
print(f"Pushed {len(sent_objects)} objects to remote commit {remote_commit}")
```

---

### Supporting Functions Used in `push`

---

#### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Fetch the SHA-1 hash of the remote repository's `master` branch commit. Returns `None` if the remote has no commits.

**Parameters:**  
- `git_url` (str): Remote Git repository URL.  
- `username` (str): Username for authentication.  
- `password` (str): Password for authentication.

**Returns:**  
- SHA-1 hex string of the remote `master` commit or `None`.

**Operation Details:**  
- Makes an HTTP request to `git_url + '/info/refs?service=git-receive-pack'`.  
- Parses the response lines according to the Git smart protocol.  
- Validates service header and empty line.  
- Extracts commit hash from the first reference line for `refs/heads/master`.  
- Returns `None` if the hash is all zeros (indicating no commits).

**Example:**

```python
remote_hash = get_remote_master_hash("https://github.com/example/repo.git", "gituser", "gitpass")
if remote_hash is None:
    print("Remote repository has no commits.")
else:
    print(f"Remote master commit: {remote_hash}")
```

---

#### `get_local_master_hash()`

**Purpose:**  
Retrieve the SHA-1 commit hash of the local `master` branch.

**Returns:**  
- SHA-1 hex string of the local `master` commit or `None` if the ref does not exist.

**Operation Details:**  
- Reads `.git/refs/heads/master` file contents.  
- Returns the commit hash string or `None` if the file is missing.

---

#### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Determine which Git objects need to be pushed by comparing local and remote commits.

**Parameters:**  
- `local_sha1` (str): SHA-1 of local commit.  
- `remote_sha1` (str or None): SHA-1 of remote commit or None.

**Returns:**  
- Set of SHA-1 strings representing objects present locally but missing remotely.

**Operation Details:**  
- Uses `find_commit_objects` to recursively find all objects reachable from each commit.  
- Returns the set difference: local objects minus remote objects.

---

#### `build_lines_data(lines)`

**Purpose:**  
Convert a list of Git protocol lines into a byte string formatted for transmission.

**Parameters:**  
- `lines` (list of bytes): Lines to encode.

**Returns:**  
- Byte string formatted with length prefixes and trailing flush packet.

**Operation Details:**  
- Each line prepended with a 4-hex-digit length (line length + 5).  
- Each line appended with a newline byte.  
- Ends with a "0000" flush packet.

---

#### `create_pack(objects)`

**Purpose:**  
Create a Git packfile containing all objects with given SHA-1 hashes.

**Parameters:**  
- `objects` (set of str): SHA-1 hashes to include.

**Returns:**  
- Bytes representing the full packfile data.

**Operation Details:**  
- Constructs packfile header (signature, version=2, number of objects).  
- Encodes each object via `encode_pack_object`.  
- Appends SHA-1 checksum of the packfile contents.

---

#### `http_request(url, username, password, data=None)`

**Purpose:**  
Perform HTTP GET or POST request with basic authentication.

**Parameters:**  
- `url` (str): Target URL.  
- `username` (str): Username for auth.  
- `password` (str): Password for auth.  
- `data` (bytes, optional): POST data if provided, otherwise GET.

**Returns:**  
- Response content as bytes.

---

#### `extract_lines(data)`

**Purpose:**  
Parse Git protocol response data into a list of lines.

**Parameters:**  
- `data` (bytes): Raw response bytes.

**Returns:**  
- List of lines (bytes), excluding length prefixes.

**Operation Details:**  
- Iteratively reads each packet line length from the first 4 hex digits.  
- Extracts line content until flush packet or end of data.

---

## ASCII Diagram: Push Operation Flow

```
+------------------+          +------------------------+
| Local Repository  |          | Remote Repository      |
|  (master branch)  |          |  (master branch)       |
+------------------+          +------------------------+
          |                              ^
          | push()                      |
          |                              |
          v                              |
+----------------------------------------------+
| Step 1: Get remote master commit hash         |
| Step 2: Get local master commit hash          |
| Step 3: Determine missing objects             |
| Step 4: Prepare Git protocol lines & packfile |
| Step 5: Send data to remote /git-receive-pack |
| Step 6: Receive and validate response         |
+----------------------------------------------+
          |                              |
          | Update remote objects         |
          v                              v
+----------------------------------------------+
| Remote repository updated with new commits   |
+----------------------------------------------+
```

---

## Summary

This document thoroughly covers the `push` operation, providing a step-by-step guide to how local commits and objects are transferred to update a remote repository's `master` branch. The operation leverages Git's smart HTTP protocol, packfile creation, and object hashing mechanisms to ensure efficient and consistent synchronization. The included example and ASCII diagram provide practical insights to users and developers for utilizing and understanding the push process within pygit.