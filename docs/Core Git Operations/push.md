# push.md

# Push Command and Related Helper Functions

---

## Overview

This document provides detailed technical documentation for the `push` command and its associated helper functions within the `pygit` project. The `push` command is a core Git operation responsible for transferring local commits from the master branch to a remote Git repository. This file fits within the broader "Core Git Operations" section of the project documentation, complementing commands like `add`, `commit`, `status`, and `diff`. Understanding the `push` command and its helpers is critical for users and developers looking to manage remote synchronization in this Git implementation effectively.

---

## Function Documentation

---

### `push(git_url, username=None, password=None)`

#### Purpose

Pushes the local `master` branch commits to the remote Git repository specified by `git_url`. It handles authentication, identifies missing objects on the remote, creates a pack file containing those objects, and sends the data to update the remote repository's master branch.

#### Parameters

- `git_url` (str): The URL of the remote Git repository.
- `username` (str, optional): Username for authentication. Defaults to environment variable `GIT_USERNAME` if not provided.
- `password` (str, optional): Password for authentication. Defaults to environment variable `GIT_PASSWORD` if not provided.

#### Operation Steps

1. Retrieve authentication credentials from parameters or environment variables.
2. Fetch the remote master branch commit hash using `get_remote_master_hash`.
3. Get the local master branch commit hash using `get_local_master_hash`.
4. Determine the set of objects missing from the remote repository via `find_missing_objects`.
5. Inform the user about the update details (old and new commit hashes, number of objects).
6. Construct the Git protocol lines indicating the old and new references.
7. Build the payload by concatenating the protocol lines and a pack file created from missing objects (`create_pack`).
8. Send an HTTP POST request to the remote's `git-receive-pack` endpoint using `http_request`.
9. Parse the response lines using `extract_lines`.
10. Verify that the remote unpacked the objects successfully and updated the reference.
11. Return the previous remote commit hash and the set of missing objects pushed.

#### Example Usage

```python
remote_url = "https://example.com/myrepo.git"
username = "alice"
password = "secretpass"

prev_remote_sha1, pushed_objects = push(remote_url, username, password)
print(f"Pushed {len(pushed_objects)} objects to remote, previous remote commit was {prev_remote_sha1}")
```

#### ASCII Diagram: Push Operation Flow

```
Local Master Commit SHA1   Remote Master Commit SHA1
          |                           |
          |                           |
          |---- find_missing_objects --|
          |                           |
          |---- create_pack(missing) ----> Pack Data
          |                           |
          |--- http_request(push) ---> Remote Git Server
          |                           |
          |<-- unpack ok, ok refs ---|
          |                           |
       Local Repo                Remote Repo
```

---

### `get_local_master_hash()`

#### Purpose

Reads and returns the current commit hash (SHA-1 hex string) of the local `master` branch. Returns `None` if the local master reference does not exist.

#### Parameters

None.

#### Operation Steps

1. Construct the path to `.git/refs/heads/master`.
2. Attempt to read the file content at this path.
3. Decode the content and strip whitespace to get the SHA-1 hash.
4. Return the hash or `None` if the file is missing.

#### Example Usage

```python
local_sha1 = get_local_master_hash()
if local_sha1:
    print(f"Local master commit hash: {local_sha1}")
else:
    print("No commits found on local master branch.")
```

---

### `get_remote_master_hash(git_url, username, password)`

#### Purpose

Fetches and returns the commit hash of the remote repository’s master branch. Returns `None` if the remote repository has no commits.

#### Parameters

- `git_url` (str): URL of the remote Git repository.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.

#### Operation Steps

1. Form the URL to query remote refs: `{git_url}/info/refs?service=git-receive-pack`.
2. Make an authenticated HTTP request to fetch the refs.
3. Extract protocol lines from the response using `extract_lines`.
4. Validate the service announcement and empty line.
5. Inspect the third line that contains SHA-1 and ref information.
6. Return `None` if the SHA-1 is all zeros (no commits).
7. Otherwise, parse and return the SHA-1 hash of the remote master.

#### Example Usage

```python
remote_sha1 = get_remote_master_hash(remote_url, username, password)
print(f"Remote master commit hash: {remote_sha1 or 'no commits'}")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

#### Purpose

Computes the set of object hashes present in the local commit but missing on the remote. This set represents the objects that need to be pushed.

#### Parameters

- `local_sha1` (str): SHA-1 hash of the local commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote commit, or `None` if remote has no commits.

#### Operation Steps

1. Find all objects reachable from the local commit using `find_commit_objects`.
2. If the remote has no commits, all local objects are missing.
3. Otherwise, find all objects reachable from the remote commit.
4. Return the difference: local objects minus remote objects.

#### Example Usage

```python
missing = find_missing_objects(local_sha1, remote_sha1)
print(f"Objects to push: {len(missing)}")
```

---

### `create_pack(objects)`

#### Purpose

Creates a Git pack file containing all specified objects. The pack file is a compact binary representation used to efficiently transfer objects.

#### Parameters

- `objects` (set of str): Set of SHA-1 hex string object hashes to include in the pack.

#### Operation Steps

1. Build pack file header: magic "PACK", version 2, number of objects.
2. Encode each object using `encode_pack_object` and concatenate.
3. Append SHA-1 checksum of the entire pack contents (header + body).
4. Return the complete pack file bytes.

#### Example Usage

```python
pack_data = create_pack(missing)
print(f"Created pack file of size {len(pack_data)} bytes")
```

---

### `http_request(url, username, password, data=None)`

#### Purpose

Performs an authenticated HTTP request to the specified URL. Uses GET if `data` is `None`, otherwise POST with the given data payload.

#### Parameters

- `url` (str): Target URL.
- `username` (str): Username for HTTP Basic Authentication.
- `password` (str): Password for HTTP Basic Authentication.
- `data` (bytes, optional): Data to send in a POST request.

#### Operation Steps

1. Create an HTTP password manager and add credentials.
2. Create an auth handler using the password manager.
3. Build an opener with the auth handler.
4. Open the URL with or without data.
5. Read and return the response bytes.

#### Example Usage

```python
response = http_request(push_url, username, password, data=pack_data)
print(f"Received {len(response)} bytes in response")
```

---

### `extract_lines(data)`

#### Purpose

Parses Git protocol packet lines from raw byte data returned by the server. Returns a list of lines without packet length prefixes.

#### Parameters

- `data` (bytes): Raw byte data from the server.

#### Operation Steps

1. Iterate through the data:
    - Read 4-byte ASCII hex length prefix.
    - Extract the line bytes of given length minus 4.
    - Append the line to the list.
2. Stop when a flush packet (`0000`) or end of data is reached.

#### Example Usage

```python
lines = extract_lines(response_data)
print(f"Extracted {len(lines)} protocol lines")
```

---

### Additional Helper Functions

The `push` operation depends on several other core helper functions, including but not limited to:

- `get_local_master_hash()`: Retrieves local master commit hash.
- `get_remote_master_hash()`: Retrieves remote master commit hash.
- `find_missing_objects()`: Determines what objects to push.
- `create_pack()`: Creates Git pack files.
- `http_request()`: Handles HTTP communication with authentication.
- `extract_lines()`: Parses Git protocol data lines.

For detailed documentation on these helpers and other related Git operations (`commit`, `add`, `status`, `diff`), please refer to their respective documentation files as outlined in the overall documentation tree.

---

## Summary Diagram: Push Command Interaction

```
+---------------------+      +-------------------------+      +----------------------+
|   Local Repository   |      |    Push Command Logic   |      |  Remote Git Server    |
| - Local master SHA1  |----->| - get_local_master_hash |      |                      |
| - Objects in .git    |      | - get_remote_master_hash|<-----|                      |
|                     |      | - find_missing_objects  |      |                      |
|                     |      | - create_pack           |      |                      |
|                     |      | - http_request          |----->|                      |
+---------------------+      +-------------------------+      +----------------------+
                                   |                                     ^
                                   |                                     |
                                   +-------------------------------------+
                                        Push pack file and refs update
```

---

This document should serve as a comprehensive guide to the `push` command and the related helper functions within the `pygit` project, enabling users and developers to understand, utilize, and extend the push functionality effectively.