# push.md

# Documentation for the `push` Command and its Implementation

---

## Overview

The `push.md` document provides comprehensive documentation for the `push` command within the `pygit` project — a Python-based Git implementation. The `push` command is responsible for uploading local repository changes, specifically the `master` branch, to a remote Git repository. This file situates itself under the **Commands** section of the overall documentation tree, alongside other user-level Git commands and workflows.

This document explains the internal workings of the `push` functionality, detailing how it interacts with the local and remote repositories, manages the set of objects to transfer, creates packfiles for efficient transfer, and communicates with the remote server using HTTP requests. It also includes descriptions of important helper functions that support the `push` operation, such as querying local and remote commit hashes, locating missing objects, and building network data packets.

Understanding the `push` command's implementation is crucial for developers extending or maintaining `pygit`, as it encapsulates key Git concepts like object management, commit graph traversal, and network protocols.

---

## Function Documentation

---

### `push(git_url, username=None, password=None)`

#### Purpose

Pushes the local `master` branch to the specified remote Git repository URL. It ensures that the remote repository is updated with the commits present locally but missing remotely.

#### Parameters

- `git_url` (str): The URL of the remote Git repository.
- `username` (str, optional): Username for HTTP authentication. Defaults to the `GIT_USERNAME` environment variable.
- `password` (str, optional): Password for HTTP authentication. Defaults to the `GIT_PASSWORD` environment variable.

#### Operation

1. Retrieves credentials from parameters or environment variables.
2. Obtains the remote repository's current `master` branch commit SHA-1 hash.
3. Retrieves the local repository's current `master` commit SHA-1.
4. Determines which Git objects are present locally but missing remotely.
5. Prints a summary of the update operation.
6. Builds Git protocol packet lines indicating the old and new commit references.
7. Creates a Git packfile containing the missing objects.
8. Sends an authenticated HTTP POST request to the remote repository's `/git-receive-pack` endpoint with the packet lines and packfile data.
9. Parses the response, asserting success messages.
10. Returns a tuple of the remote SHA-1 before the push and the set of missing objects sent.

#### Example

```python
remote_url = 'https://example.com/myrepo.git'
push(remote_url)
```

---

### `get_local_master_hash()`

#### Purpose

Retrieves the current commit hash of the local `master` branch.

#### Returns

- SHA-1 string of the local `master` branch commit, or `None` if the branch does not exist.

#### Operation

- Reads the `.git/refs/heads/master` file to obtain the stored commit hash.
- Handles the case where the file might not exist.

#### Example

```python
local_sha1 = get_local_master_hash()
print(f"Local master commit: {local_sha1}")
```

---

### `get_remote_master_hash(git_url, username, password)`

#### Purpose

Fetches the current commit hash of the remote repository's `master` branch.

#### Parameters

- `git_url` (str): Remote repository base URL.
- `username` (str): HTTP authentication username.
- `password` (str): HTTP authentication password.

#### Returns

- SHA-1 string of the remote `master` branch commit, or `None` if no commits exist remotely.

#### Operation

- Sends an HTTP GET request to `<git_url>/info/refs?service=git-receive-pack`.
- Parses the Git protocol packet lines returned.
- Extracts the SHA-1 hash for the `refs/heads/master` reference.
- Handles the case where no commits are present (indicated by all zero SHA-1).

#### Example

```python
remote_sha1 = get_remote_master_hash('https://example.com/myrepo.git', 'user', 'pass')
print(f"Remote master commit: {remote_sha1 or 'no commits'}")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

#### Purpose

Determines which Git objects exist locally but are missing in the remote repository, based on the given commit hashes.

#### Parameters

- `local_sha1` (str): SHA-1 hash of the local commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote commit (or `None` if no commits remotely).

#### Returns

- Set of SHA-1 hashes representing missing objects.

#### Operation

- Recursively finds all objects reachable from the local commit.
- Recursively finds all objects reachable from the remote commit (if any).
- Returns the set difference: local objects minus remote objects.

#### Example

```python
missing_objects = find_missing_objects(local_sha1, remote_sha1)
print(f"Objects to push: {len(missing_objects)}")
```

---

### `build_lines_data(lines)`

#### Purpose

Constructs a byte string formatted according to the Git protocol to send packet lines to the server.

#### Parameters

- `lines` (list of bytes): List of byte strings representing individual packet lines.

#### Returns

- A single byte string representing all lines, each prefixed with their length in hex and suffixed with newline, ending with a flush packet `0000`.

#### Operation

- For each line, calculates length as line length + 5 (4 bytes length field + newline).
- Formats length as 4-digit hex.
- Concatenates length, line, and newline.
- Appends a flush packet `0000` to indicate end.

#### Example

```python
lines = [b'0123456789abcdef0123456789abcdef01234567 89abcdef0123456789abcdef0123456789abcdef refs/heads/master\x00 report-status']
data = build_lines_data(lines)
```

---

### `create_pack(objects)`

#### Purpose

Creates a Git packfile containing all specified objects for efficient transfer.

#### Parameters

- `objects` (set of str): Set of SHA-1 object hashes to include.

#### Returns

- Bytes of the complete packfile.

#### Operation

- Constructs packfile header (`PACK`, version 2, object count).
- Encodes each object using `encode_pack_object`.
- Concatenates header and encoded objects.
- Appends SHA-1 checksum of the entire packfile contents.

#### Example

```python
pack_data = create_pack(missing_objects)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
```

---

### `http_request(url, username, password, data=None)`

#### Purpose

Performs an authenticated HTTP request to the given URL, sending optional data.

#### Parameters

- `url` (str): Target URL for the request.
- `username` (str): Username for HTTP Basic Authentication.
- `password` (str): Password for HTTP Basic Authentication.
- `data` (bytes, optional): Data payload for POST request; if `None`, performs GET.

#### Returns

- Raw response bytes from the server.

#### Operation

- Sets up HTTP Basic Authentication handler with given credentials.
- Opens the URL with or without data.
- Reads and returns response.

#### Example

```python
response = http_request('https://example.com/myrepo/git-receive-pack', 'user', 'pass', data=pack_data)
```

---

### `extract_lines(data)`

#### Purpose

Extracts Git protocol packet lines from raw server response data.

#### Parameters

- `data` (bytes): Raw bytes from server response.

#### Returns

- List of byte strings representing individual packet lines.

#### Operation

- Iterates over data in chunks prefixed by 4 hex chars indicating length.
- Extracts lines accordingly until a flush packet or end of data.

#### Example

```python
lines = extract_lines(response)
for line in lines:
    print(line)
```

---

## ASCII Diagram: `push` Command Flow

```
Local Master Commit SHA-1
          │
          ▼
  get_local_master_hash()
          │
          ▼
Remote Master Commit SHA-1 <─ get_remote_master_hash() ── Remote Git Server
          │
          ▼
find_missing_objects(local_sha1, remote_sha1)
          │
          ▼
Create Packfile of missing objects
          │
          ▼
build_lines_data() + create_pack()
          │
          ▼
http_request() ────────────────► Remote Git Server (/git-receive-pack)
          │
          ▼
extract_lines(response)
          │
          ▼
Verify success and return
```

---

## Summary

The `push` command in `pygit` efficiently synchronizes the local `master` branch with a remote Git repository by:

- Comparing commit histories to identify missing objects.
- Packaging these objects into a compressed packfile.
- Sending the packfile via authenticated HTTP requests using the Git smart protocol.
- Confirming successful updates with the remote server.

This documentation outlines the key functions involved in the process, giving developers insight into the underlying mechanisms of pushing changes in `pygit`.