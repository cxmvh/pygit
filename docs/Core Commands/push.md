# push.md — Pushing Local Changes to Remote Git Repository

---

## Overview

The `push.md` document details the process of pushing local commits from the master branch of a local Git repository to a remote repository URL. This functionality is a core command in the `pygit` suite, forming a crucial part of repository synchronization by transferring new commits and their associated objects to the remote server.

Within the broader documentation tree, this file resides under the **Core Commands** section, emphasizing its role as a fundamental Git operation alongside commits, status checks, diffs, and index management. The `push` command builds upon lower-level repository structures like commits, trees, and objects, coordinating them to update a remote repository securely and efficiently.

---

## Function Documentation

### `push(git_url, username=None, password=None)`

Push the local `master` branch commits and objects to the specified remote Git repository.

#### Purpose

- Synchronizes local changes by sending missing commit objects to the remote repository.
- Authenticates with the remote using provided credentials or environment variables.
- Ensures that the remote `master` branch advances from its current commit to the local `master` commit.

#### Parameters

- `git_url` (str): The URL of the remote Git repository to push to.
- `username` (str, optional): Username for authentication; defaults to environment variable `GIT_USERNAME` if not provided.
- `password` (str, optional): Password for authentication; defaults to environment variable `GIT_PASSWORD` if not provided.

#### Operation Steps

1. Determine the username and password for authentication; fallback to environment variables if not explicitly given.
2. Retrieve the remote's current `master` branch commit SHA-1 via `get_remote_master_hash`.
3. Retrieve the local `master` branch commit SHA-1 via `get_local_master_hash`.
4. Identify the set of missing objects that exist locally but not on the remote using `find_missing_objects`.
5. Display a summary of updating from the remote SHA to the local SHA and the number of objects being pushed.
6. Construct the push request lines, including old and new commit hashes and the reference update command, encoded properly for Git protocol.
7. Create a packfile containing all missing objects (`create_pack`) and concatenate it with the request lines.
8. Send the assembled data to the remote's `/git-receive-pack` endpoint using an authenticated HTTP POST (`http_request`).
9. Parse the response lines (`extract_lines`) and verify the push succeeded by checking for `unpack ok` and `ok refs/heads/master` lines.
10. Return the remote SHA before the push and the set of pushed object hashes.

#### Example Usage

```python
remote_url = "https://example.com/myrepo.git"
# Optionally set environment variables GIT_USERNAME and GIT_PASSWORD for authentication
previous_sha, pushed_objects = push(remote_url)
print(f"Pushed {len(pushed_objects)} object(s) to remote. Remote was at {previous_sha}")
```

#### ASCII Diagram: Push Operation Flow

```
Local master SHA:  A1B2C3... (commit hash)
Remote master SHA: D4E5F6... (commit hash or None)

+----------------------------+
| Identify missing objects    |
| (local_objects - remote_objects) |
+--------------+-------------+
               |
               V
+----------------------------+
| Build packfile of missing  |
| objects                    |
+--------------+-------------+
               |
               V
+----------------------------+
| Send push request with pack|
| to remote /git-receive-pack|
+--------------+-------------+
               |
               V
+----------------------------+
| Remote unpacks objects and |
| updates refs/heads/master  |
+----------------------------+
```

---

### `get_local_master_hash()`

Retrieve the current commit hash (SHA-1) of the local `master` branch.

#### Purpose

- Reads the commit hash stored in `.git/refs/heads/master`.
- Returns `None` if the reference does not exist (e.g., no commits yet).

#### Usage Example

```python
local_sha = get_local_master_hash()
if local_sha:
    print(f"Local master commit: {local_sha}")
else:
    print("No commits found on local master branch.")
```

---

### `get_remote_master_hash(git_url, username, password)`

Get the SHA-1 commit hash of the remote `master` branch.

#### Purpose

- Accesses the remote's `/info/refs?service=git-receive-pack` endpoint.
- Parses the response to find the current commit hash of `refs/heads/master`.
- Returns `None` if the remote has no commits.

#### Parameters

- `git_url` (str): Remote git repository URL.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.

#### Example Usage

```python
remote_sha = get_remote_master_hash(remote_url, username="user", password="pass")
if remote_sha:
    print(f"Remote master commit: {remote_sha}")
else:
    print("Remote repository has no commits on master branch.")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

Determine which Git objects exist locally but are missing remotely.

#### Purpose

- Recursively identifies all objects reachable from the local commit.
- Compares to objects reachable from the remote commit.
- Returns a set of SHA-1 hashes representing objects to push.

#### Parameters

- `local_sha1` (str): Local commit SHA-1 hash.
- `remote_sha1` (str or None): Remote commit SHA-1 hash, or None if no remote commits.

#### Example Usage

```python
missing = find_missing_objects(local_sha, remote_sha)
print(f"{len(missing)} objects need to be pushed.")
```

---

### `create_pack(objects)`

Create a Git packfile from a set of object SHA-1 hashes.

#### Purpose

- Builds a packfile binary containing all specified objects.
- Includes a packfile header, concatenated encoded objects, and a trailing SHA-1 checksum.
- Enables efficient network transfer of multiple Git objects in one payload.

#### Parameters

- `objects` (set of str): Set of SHA-1 hashes to include in the pack.

#### Returns

- `bytes`: The complete packfile data ready for transmission.

#### Example Usage

```python
pack_data = create_pack(missing_objects)
# 'pack_data' can then be sent as part of a push request
```

---

### `build_lines_data(lines)`

Encode protocol lines for the Git push request.

#### Purpose

- Converts a list of byte strings (lines) into the Git pkt-line format.
- Each line is prefixed with a 4-byte hexadecimal length, followed by the line and a newline.
- Terminates with a `0000` flush packet.

#### Parameters

- `lines` (list of bytes): Lines to encode.

#### Returns

- `bytes`: Encoded packet line data.

#### Example Usage

```python
lines = [b'00000000 HEAD\x00 report-status\n']
data = build_lines_data(lines)
```

---

### `http_request(url, username, password, data=None)`

Perform an HTTP request with optional POST data and basic authentication.

#### Purpose

- Opens a connection to the specified URL.
- Uses HTTP Basic Auth with provided credentials.
- Sends data using POST if `data` is not None; otherwise, GET.
- Returns the response body.

#### Parameters

- `url` (str): URL to access.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.
- `data` (bytes, optional): Data to POST (if any).

#### Returns

- `bytes`: Response content.

#### Example Usage

```python
response = http_request('https://example.com/git-receive-pack', 'user', 'pass', data=pack_data)
```

---

### `extract_lines(data)`

Parse raw server response into individual protocol lines.

#### Purpose

- Processes Git pkt-line formatted data.
- Extracts each line by parsing the 4-byte length prefix.
- Stops on flush packet (`0000`).

#### Parameters

- `data` (bytes): Response data from server.

#### Returns

- `list` of `bytes`: List of extracted lines.

#### Example Usage

```python
lines = extract_lines(response_data)
for line in lines:
    print(line)
```

---

## Summary Diagram of the Push Process

```
+------------------+
| Determine local   |----+
| master SHA       |    |
+------------------+    |
                        |         +------------------+
+------------------+    |         | Determine remote  |
| Determine remote  |----+-------->| master SHA        |
| master SHA       |              +------------------+
+------------------+                       |
                                          |
                                          v
                                  +------------------+
                                  | Find missing     |
                                  | objects to push  |
                                  +------------------+
                                          |
                                          v
                                  +------------------+
                                  | Build packfile   |
                                  +------------------+
                                          |
                                          v
                                  +------------------+
                                  | Send push HTTP   |
                                  | request          |
                                  +------------------+
                                          |
                                          v
                                  +------------------+
                                  | Receive response |
                                  | and verify push  |
                                  +------------------+
```

---

## Additional Notes

- This push implementation currently supports pushing only the `master` branch.
- Authentication relies on HTTP Basic Auth; environment variables `GIT_USERNAME` and `GIT_PASSWORD` can be used.
- The push protocol uses Git's native pkt-line format and packfile structure to efficiently transfer data.
- The implementation verifies remote responses to ensure the push succeeded before returning.

---

# End of `push.md` Documentation