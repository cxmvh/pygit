# push.md - Pushing Changes to a Remote Repository

---

## Overview

This document provides detailed technical guidance on the process of pushing local commits to a remote Git repository within the pygit project. It explains the functionality and operation of the push command implementation (`pygit.push`) which updates the remote repository’s master branch by sending the necessary Git objects. This file is part of the broader *Remote Operations* section in the documentation tree, complementing other core Git commands such as commit creation and index management. Understanding this push process is critical for synchronizing local development progress with a shared remote repository.

---

## Function Documentation

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the current local `master` branch commits and related objects to the remote Git repository specified by `git_url`. This function handles authentication, determines the missing objects on the remote, packages them, and performs the network update to the remote's `master` branch.

**Parameters:**  
- `git_url` (str): The URL of the remote Git repository to push to.  
- `username` (str, optional): Username for HTTP authentication. Defaults to environment variable `GIT_USERNAME` if not provided.  
- `password` (str, optional): Password for HTTP authentication. Defaults to environment variable `GIT_PASSWORD` if not provided.

**Operation Steps:**

1. Retrieve credentials from environment variables if not explicitly provided.  
2. Obtain the SHA-1 hash of the remote repository’s current `master` commit using `get_remote_master_hash()`.  
3. Retrieve the local repository’s current `master` commit SHA-1 using `get_local_master_hash()`.  
4. Determine which Git objects exist in the local commit but are missing on the remote with `find_missing_objects()`.  
5. Print a summary of the update operation including the number of objects to push.  
6. Construct the push request lines specifying the old and new commit hashes and the reference to update (`refs/heads/master`), formatted per Git protocol.  
7. Build the binary payload by encoding the protocol lines and creating a pack file of missing objects using `build_lines_data()` and `create_pack()`.  
8. Send the push request to the remote at the endpoint `/git-receive-pack` with HTTP POST via `http_request()`.  
9. Parse and validate the remote server’s response to ensure successful unpacking and reference update.  
10. Return a tuple containing the remote's previous SHA-1 and the set of missing objects pushed.

**Example Usage:**

```python
git_url = "https://example.com/user/repo.git"
username = "alice"
password = "s3cr3t"

remote_sha1, pushed_objects = push(git_url, username, password)
print(f"Pushed {len(pushed_objects)} objects to remote. Previous remote commit was {remote_sha1}.")
```

---

### Supporting Functions Used by `push`

#### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Fetch the current commit hash of the remote repository’s `master` branch.

**Example:**

```python
remote_hash = get_remote_master_hash("https://example.com/user/repo.git", "alice", "s3cr3t")
print(f"Remote master hash: {remote_hash}")
```

#### `get_local_master_hash()`

**Purpose:**  
Return the SHA-1 hash of the local `master` branch commit, or `None` if none exists.

**Example:**

```python
local_hash = get_local_master_hash()
print(f"Local master hash: {local_hash}")
```

#### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Compute the set of Git objects present in the local commit but absent in the remote commit.

**Example:**

```python
missing = find_missing_objects(local_hash, remote_hash)
print(f"Objects missing on remote: {len(missing)}")
```

#### `build_lines_data(lines)`

**Purpose:**  
Encode protocol lines into the Git packet line format for transmission.

**Example:**

```python
lines = [b'0000', b'...']
data = build_lines_data(lines)
```

#### `create_pack(objects)`

**Purpose:**  
Create a packed Git object archive containing all specified objects.

**Example:**

```python
pack_data = create_pack(missing)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
```

#### `http_request(url, username, password, data=None)`

**Purpose:**  
Perform an authenticated HTTP request (GET or POST) to the specified URL.

**Example:**

```python
response = http_request("https://example.com/git-receive-pack", "alice", "s3cr3t", data=pack_data)
print(response)
```

#### `extract_lines(data)`

**Purpose:**  
Parse raw server response data into individual protocol lines.

---

## ASCII Diagram: Push Operation Flow

```
+----------------+       +-------------------+       +---------------------+
| Local Repository|       | Remote Repository  |       | pygit push function  |
| (master commit) |       | (master commit)    |       |                     |
+-------+--------+       +---------+---------+       +----------+----------+
        |                          |                            |
        | Get local master SHA-1   |                            |
        +------------------------->|                            |
        |                          |                            |
        |          Get remote master SHA-1                     |
        |<-----------------------------------------------------+
        |                          |                            |
        |          Compare commits and find missing objects    |
        |----------------------------------------------------->|
        |                          |                            |
        |               Create pack of missing objects         |
        |----------------------------------------------------->|
        |                          |                            |
        |         HTTP POST /git-receive-pack with pack data   |
        |----------------------------------------------------->|
        |                          |                            |
        |               Remote validates and updates refs      |
        |<-----------------------------------------------------+
        |                          |                            |
        |               Confirm success and report pushed objs |
        +----------------------------------------------------->
```

---

## Summary

The `push.md` documentation details the implementation of the Git push command within pygit. The `push()` function orchestrates communication with the remote repository, ensuring that only necessary objects are transmitted and that the remote `master` branch reference is updated accordingly. This operation relies on several supporting functions for handling Git objects, network communication, and protocol formatting. Mastery of this flow is essential for understanding how pygit synchronizes local development progress with remote repositories.