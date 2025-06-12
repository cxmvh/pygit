# push.md

# Documentation for Pushing Commits to Remote Repositories

---

## Overview

This document describes the functionality and usage of the **push** operation within the `pygit` project, specifically the process of pushing commits and git objects from a local repository to a remote repository. It fits within the **Remote Operations** section of the overall documentation tree, complementing other repository management features such as commit creation, object storage, and index management.

The `push` command enables synchronization of the local master branch with a remote git repository by transferring missing commit objects and updating the remote reference. It handles authentication, packfile creation, and communication using Git's smart HTTP protocol.

---

## Functions Related to Push Operation

### 1. `push(git_url, username=None, password=None)`

#### Purpose

Pushes the local `master` branch to the specified remote git repository URL. It authenticates with the remote server, determines which commits/objects are missing remotely, creates a packfile of these missing objects, and sends them to update the remote branch.

#### Parameters

- `git_url` (str): URL of the remote git repository.
- `username` (str, optional): Username for HTTP authentication. Defaults to environment variable `GIT_USERNAME`.
- `password` (str, optional): Password for HTTP authentication. Defaults to environment variable `GIT_PASSWORD`.

#### Operation Steps

1. Retrieve credentials from parameters or environment variables.
2. Obtain the current commit hash of the remote master branch using `get_remote_master_hash`.
3. Obtain the current commit hash of the local master branch using `get_local_master_hash`.
4. Identify the set of objects present locally but missing remotely via `find_missing_objects`.
5. Inform the user about the update, including the number of objects to push.
6. Construct protocol lines indicating the refs update along with the `report-status` option.
7. Build the line data string and create a packfile containing the missing objects using `build_lines_data` and `create_pack`.
8. Send an authenticated HTTP POST request to the remote `/git-receive-pack` endpoint with the data.
9. Parse the server response using `extract_lines` and verify success by checking for expected responses (`unpack ok`, `ok refs/heads/master`).
10. Return a tuple of the remote SHA-1 prior to the push and the set of missing objects sent.

#### Example Usage

```python
remote_url = "https://example.com/myrepo.git"
# Optionally set environment variables GIT_USERNAME and GIT_PASSWORD or pass directly
remote_sha1, pushed_objects = push(remote_url)
print(f"Pushed {len(pushed_objects)} objects to remote {remote_url}")
```

---

### 2. `get_remote_master_hash(git_url, username, password)`

#### Purpose

Fetches the current commit hash of the remote repository's master branch. If there are no commits on the remote, returns `None`.

#### Parameters

- `git_url` (str): URL of the remote git repository.
- `username` (str): HTTP authentication username.
- `password` (str): HTTP authentication password.

#### Operation Steps

1. Compose URL for `info/refs` service endpoint with `git-receive-pack`.
2. Perform authenticated HTTP GET request using `http_request`.
3. Extract lines from the response using `extract_lines`.
4. Verify response starts with expected service announcement.
5. Parse the line containing the remote master SHA-1 and reference.
6. If the SHA-1 consists of all zeros, return `None`.
7. Otherwise, return the SHA-1 string.

#### Example Usage

```python
remote_sha1 = get_remote_master_hash("https://example.com/myrepo.git", "user", "pass")
print(f"Remote master commit: {remote_sha1}")
```

---

### 3. `get_local_master_hash()`

#### Purpose

Reads the current commit hash of the local repository's master branch. Returns `None` if no commit exists (e.g., newly initialized repo).

#### Operation Steps

1. Reads the contents of `.git/refs/heads/master`.
2. Decodes and strips whitespace.
3. Returns the SHA-1 string or `None` if the file is missing.

#### Example Usage

```python
local_sha1 = get_local_master_hash()
print(f"Local master commit: {local_sha1}")
```

---

### 4. `find_missing_objects(local_sha1, remote_sha1)`

#### Purpose

Determines which git objects are present locally but missing on the remote repository by comparing commits.

#### Parameters

- `local_sha1` (str): SHA-1 hash of the local commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote commit, or None if no remote commits exist.

#### Operation Steps

1. Recursively find all objects referenced by the local commit using `find_commit_objects`.
2. If remote commit is `None`, return all local objects as missing.
3. Otherwise, find all objects referenced by the remote commit.
4. Return the set difference (local objects minus remote objects).

#### Example Usage

```python
missing = find_missing_objects(local_sha1, remote_sha1)
print(f"Objects to push: {len(missing)}")
```

---

### 5. `build_lines_data(lines)`

#### Purpose

Encodes a list of byte strings (protocol lines) into the Git pkt-line format, preparing the data to send to the remote server.

#### Parameters

- `lines` (list of bytes): Lines to encode.

#### Operation Steps

1. For each line, calculate length including 4-byte length header plus newline.
2. Format length as 4 hex characters.
3. Append line and a newline.
4. Append a flush packet (`0000`) to indicate the end.

#### Example Usage

```python
lines = [b'0000000000000000000000000000000000000000 1234567890abcdef1234567890abcdef12345678 refs/heads/master\x00 report-status']
data = build_lines_data(lines)
```

---

### 6. `create_pack(objects)`

#### Purpose

Creates a Git packfile containing all specified objects, ready for upload to a remote repository.

#### Parameters

- `objects` (set of str): Set of SHA-1 hex strings of objects to include.

#### Operation Steps

1. Packfile header: magic "PACK", version 2, number of objects.
2. Encode each object using `encode_pack_object` (variable-length header + compressed data).
3. Concatenate header and encoded objects.
4. Compute SHA-1 checksum of the concatenated data.
5. Append checksum to form the full packfile data.

#### Example Usage

```python
pack_data = create_pack(missing_objects)
# pack_data can be sent as HTTP POST payload
```

---

### 7. `http_request(url, username, password, data=None)`

#### Purpose

Performs an HTTP request to the specified URL with optional POST data and basic authentication.

#### Parameters

- `url` (str): Target URL.
- `username` (str): HTTP auth username.
- `password` (str): HTTP auth password.
- `data` (bytes, optional): POST data if provided; otherwise, GET request.

#### Operation Steps

1. Setup password manager and authentication handler.
2. Build an opener with the auth handler.
3. Open the URL with data if given (POST), else GET.
4. Return the raw response bytes.

#### Example Usage

```python
response = http_request("https://example.com/myrepo.git/git-receive-pack", "user", "pass", data=pack_data)
```

---

### 8. `extract_lines(data)`

#### Purpose

Parses Git pkt-line formatted data and returns a list of byte lines.

#### Parameters

- `data` (bytes): Raw data received from the server.

#### Operation Steps

1. Parse 4-byte length header (hex).
2. Extract line of specified length minus 4.
3. Repeat until flush packet (`0000`) or data end.

#### Example Usage

```python
lines = extract_lines(response)
for line in lines:
    print(line)
```

---

## ASCII Diagram: Push Operation Flow

```
+----------------+         +-------------------+        +------------------+
| Local Repository|         | Remote Git Server  |        | Authentication   |
| (Local master)  |  -----> |                   |        | (Username/Pass)  |
+----------------+         +-------------------+        +------------------+
        |                          ^                           ^
        | push()                   |                           |
        |                          |                           |
        v                          |                           |
+---------------------+            |                           |
| Get local commit SHA |            |                           |
+---------------------+            |                           |
        |                          |                           |
        v                          |                           |
+----------------------+           |                           |
| Get remote commit SHA | ---------|---------------------------|
+----------------------+           |
        |                          |
        v                          |
+-----------------------------+   |
| Identify missing objects     |   |
+-----------------------------+   |
        |                          |
        v                          |
+-----------------------------+   |
| Build protocol lines & packfile|  |
+-----------------------------+   |
        |                          |
        v                          |
+-----------------------------+   |
| Send POST to /git-receive-pack| |
+-----------------------------+   |
        |                        Response
        v                          |
+-----------------------------+   |
| Parse response, confirm push |   |
+-----------------------------+   |
```

---

# Summary

The `push.md` documentation covers the essential functions involved in pushing commits from a local git repository to a remote repository using HTTP and the Git smart protocol. It explains how the local and remote commit states are compared, how missing objects are packed and sent, and how the server response is handled to confirm success. This document is critical for understanding the remote synchronization capabilities of the `pygit` implementation.