# push.md

# Pushing Commits to Remote Repository and Handling Missing Objects

---

## Overview

This document describes the functionality and usage of the `push` operations within the pygit project. It focuses on pushing commits from the local repository's master branch to a remote Git repository, ensuring synchronization by identifying and transferring missing objects. The pushing mechanism involves querying the remote repository's state, comparing it with the local state, preparing the necessary Git packfile for transfer, and handling the network communication with authentication. This file fits within the broader "Remote Operations" section of the documentation tree and interacts closely with object handling, commit management, and diff functionalities.

---

## Function Documentation

### `push(git_url, username=None, password=None)`

#### Purpose
Pushes the local master branch commits to the specified remote Git repository URL. It ensures that only the missing objects that the remote repository does not have are sent, optimizing the data transfer. The function performs authentication using the provided username and password or environment variables.

#### Parameters
- `git_url` (str): The URL of the remote Git repository to push to.
- `username` (str, optional): Username for HTTP authentication. Defaults to the environment variable `GIT_USERNAME` if not provided.
- `password` (str, optional): Password for HTTP authentication. Defaults to the environment variable `GIT_PASSWORD` if not provided.

#### Operation Steps
1. Retrieve authentication credentials from parameters or environment.
2. Fetch the remote master branch's latest commit SHA-1 hash.
3. Retrieve the local master branch's latest commit SHA-1 hash.
4. Identify missing objects that exist locally but not remotely.
5. Display a status message indicating the update range and number of objects.
6. Construct the Git push request lines and encode them.
7. Create a packfile containing the missing objects.
8. Send an HTTP POST request to the remote repository's `git-receive-pack` endpoint with the packed data.
9. Extract and validate the response lines to confirm successful unpack and update.
10. Return a tuple of the remote SHA-1 before push and the set of missing objects pushed.

#### Usage Example

```python
remote_url = "https://example.com/myrepo.git"
# Optionally set environment variables GIT_USERNAME and GIT_PASSWORD or provide them here:
user = "myuser"
passwd = "mypassword"

remote_sha1, pushed_objects = push(remote_url, username=user, password=passwd)
print(f"Pushed {len(pushed_objects)} objects to remote. Remote SHA1 was {remote_sha1}")
```

---

### `get_remote_master_hash(git_url, username, password)`

#### Purpose
Retrieves the latest commit SHA-1 hash of the remote repository's master branch. Returns `None` if the remote repository has no commits yet.

#### Parameters
- `git_url` (str): The URL of the remote Git repository.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.

#### Operation Steps
1. Constructs the URL to request remote references for `git-receive-pack`.
2. Performs an authenticated HTTP GET request.
3. Parses the packet-line formatted response.
4. Validates response header lines.
5. Checks if the remote master SHA-1 is a zero hash indicating no commits.
6. Extracts and returns the remote master commit SHA-1 as a hex string.

#### Usage Example

```python
remote_sha1 = get_remote_master_hash("https://example.com/myrepo.git", "user", "pass")
if remote_sha1 is None:
    print("Remote repository has no commits yet.")
else:
    print(f"Remote master commit SHA-1: {remote_sha1}")
```

---

### `get_local_master_hash()`

#### Purpose
Reads and returns the current commit SHA-1 hash of the local repository's master branch. Returns `None` if the local master branch does not exist.

#### Parameters
None

#### Operation Steps
1. Reads the `.git/refs/heads/master` file.
2. Returns the trimmed commit hash string.
3. Returns `None` if the file is not found.

#### Usage Example

```python
local_sha1 = get_local_master_hash()
if local_sha1 is None:
    print("Local master branch not found.")
else:
    print(f"Local master commit SHA-1: {local_sha1}")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

#### Purpose
Determines the set of commit objects present in the local master branch commit that are missing from the remote repository.

#### Parameters
- `local_sha1` (str): SHA-1 hash of the local master branch commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote master branch commit, or `None` if no remote commits exist.

#### Operation Steps
1. Retrieve all objects referenced by the local commit recursively.
2. If no remote commit exists, return all local objects.
3. Otherwise, retrieve the objects referenced by the remote commit.
4. Return the set difference (local_objects - remote_objects) as missing objects.

#### Usage Example

```python
missing_objects = find_missing_objects(local_sha1, remote_sha1)
print(f"Number of missing objects to push: {len(missing_objects)}")
```

---

### `build_lines_data(lines)`

#### Purpose
Encodes a list of byte strings (lines) into the Git packet-line format for sending to the Git server.

#### Parameters
- `lines` (list of bytes): Lines to encode.

#### Operation Steps
1. For each line, prepend a 4-byte hex length field (`length(line) + 5`).
2. Append the line and a newline character (`\n`).
3. Append a terminating packet-line `0000` to indicate the end.
4. Return the concatenation of all encoded lines as bytes.

#### Usage Example

```python
lines = [b'0000000000 some line\\x00 report-status']
data = build_lines_data(lines)
# data can then be sent in an HTTP request body
```

---

### `create_pack(objects)`

#### Purpose
Constructs a Git packfile containing all specified objects, returning the packed data bytes.

#### Parameters
- `objects` (set of str): Set of SHA-1 hashes of objects to include.

#### Operation Steps
1. Create a PACK header with version 2 and number of objects.
2. Encode each object into packfile format using `encode_pack_object`.
3. Concatenate header and encoded object bodies.
4. Compute and append SHA-1 checksum of the entire packfile contents.
5. Return the full packfile bytes.

#### Usage Example

```python
pack_data = create_pack(missing_objects)
# pack_data can be sent as part of the push HTTP request
```

---

### `http_request(url, username, password, data=None)`

#### Purpose
Performs an HTTP request with basic authentication. Uses GET if no data, POST if data is provided.

#### Parameters
- `url` (str): URL to request.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.
- `data` (bytes, optional): POST data to send. If `None`, makes a GET request.

#### Operation Steps
1. Set up an HTTP password manager with credentials.
2. Create an authentication handler.
3. Build an opener with the auth handler.
4. Open the URL with or without data.
5. Read and return the response bytes.

#### Usage Example

```python
response = http_request("https://example.com/git-receive-pack", "user", "pass", data=pack_data)
print("Received response of length", len(response))
```

---

### `extract_lines(data)`

#### Purpose
Parses Git packet-line encoded data into a list of lines (bytes).

#### Parameters
- `data` (bytes): Packet-line encoded data from Git server.

#### Operation Steps
1. Initialize index pointer.
2. Read 4-byte hex length prefix.
3. Extract the line of indicated length minus 4 bytes.
4. Append the line to list.
5. Repeat until end or zero length line (`0000`) encountered.
6. Return list of lines.

#### Usage Example

```python
lines = extract_lines(response_data)
for i, line in enumerate(lines):
    print(f"Line {i}: {line}")
```

---

## ASCII Diagram: Push Operation Flow

```
+--------------------+            +-----------------------+
| Local Repository    |            | Remote Repository     |
| (master branch)     |            | (master branch)       |
+---------+----------+            +-----------+-----------+
          |                                   ^
          | 1. get_local_master_hash()        |
          |---------------------------------->|
          |                                   |
          | 2. get_remote_master_hash()       |
          |<----------------------------------|
          |                                   |
          | 3. find_missing_objects()         |
          |                                   |
          | 4. create_pack(missing_objects)   |
          |                                   |
          | 5. http_request(push data)        |
          |---------------------------------->|
          |                                   |
          | 6. extract_lines(response)        |
          |<----------------------------------|
          |                                   |
          |       Push Completed              |
          +----------------------------------+
```

---

# Summary

The push functionality in pygit carefully orchestrates the synchronization between the local and remote Git repositories by comparing commit states, identifying missing objects, packaging them efficiently, and communicating via authenticated HTTP requests. This facilitates a reliable and efficient transfer of commits to remote repositories supporting Git protocol version 2.