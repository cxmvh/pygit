---
sidebar_position: 6
---

# Push Feature Documentation

## Overview

This document provides a detailed reference for the push feature implemented in `pygit.push`. The push operation is essential for synchronizing local repository changes with a remote Git server. This includes handling authentication, discovering which objects need to be pushed, creating packfiles for efficient data transfer, and managing communication protocols with remote repositories. This file fits into the broader "Core Git Features" section of the documentation, which covers fundamental repository operations and remote interactions.

## Key Concepts

The push process involves several critical steps:

1. **Authentication**: Verifying user credentials with the remote server.
2. **Object Discovery**: Identifying commits, trees, and blobs that are missing on the remote side.
3. **Pack Creation**: Bundling objects into a compressed packfile for efficient transfer.
4. **Communication**: Exchanging references and objects with the remote Git server via the Git protocol.

---

## Functions and Their Descriptions

### `push(repository, remote_name, refspecs, credentials)`

#### Purpose

The primary function to perform a push operation from a local repository to a remote Git server.

- **Parameters:**
  - `repository`: The local repository object representing the current Git repository.
  - `remote_name`: The name of the remote (e.g., `'origin'`) as configured in the repository.
  - `refspecs`: A list of refspec strings specifying which references to push (e.g., `['refs/heads/main:refs/heads/main']`).
  - `credentials`: Authentication credentials (username/password, SSH keys, or tokens) required for access to the remote.

#### Description

1. **Initialization**: Load remote configuration based on `remote_name`.
2. **Authentication**: Use provided `credentials` to authenticate with the remote server.
3. **Reference Advertisement**: Query the remote server to obtain the current state of references.
4. **Object Discovery**: Compare local objects against remote references to find missing objects that need to be pushed.
5. **Packfile Generation**: Create a Git packfile containing all missing objects to optimize data transfer.
6. **Data Transmission**: Send the packfile and update references on the remote.
7. **Result Handling**: Interpret the remote server response to confirm successful push or handle errors.

#### Example Usage

```python
from pygit import Repository, push

repo = Repository('/path/to/repo')
remote = 'origin'
refs = ['refs/heads/main:refs/heads/main']
creds = {
    'username': 'gituser',
    'password': 'secretpassword'
}

push(repo, remote, refs, creds)
```

---

### `authenticate(remote_url, credentials)`

#### Purpose

Handles the authentication process when connecting to a remote Git server.

- **Parameters:**
  - `remote_url`: The URL of the remote Git repository.
  - `credentials`: A dictionary or object containing authentication details.

#### Description

- Supports various authentication methods (HTTP Basic Auth, SSH keys, OAuth tokens).
- Establishes a secure connection to the remote.
- Returns an authenticated session or connection object for further communication.

#### Example Usage

```python
session = authenticate('https://github.com/user/repo.git', {
    'username': 'user',
    'password': 'pass'
})
```

---

### `discover_objects(local_repo, remote_refs, refspecs)`

#### Purpose

Determines which Git objects (commits, trees, blobs) need to be pushed by comparing local references with remote references.

- **Parameters:**
  - `local_repo`: The local repository object.
  - `remote_refs`: Dictionary mapping remote reference names to commit SHAs.
  - `refspecs`: List of refspec strings specifying references to push.

#### Description

- For each refspec, identify the local commit SHA.
- Traverse commit history from the local commit backward until commits present in remote are found.
- Collect all objects that are not present in the remote.
- Return a set or list of object SHAs to be included in the packfile.

#### Example Usage

```python
missing_objects = discover_objects(repo, remote_refs, ['refs/heads/main:refs/heads/main'])
```

---

### `create_packfile(repository, object_shas)`

#### Purpose

Generates a Git packfile from the specified objects to optimize transfer size.

- **Parameters:**
  - `repository`: The local repository object.
  - `object_shas`: Iterable of object SHA-1 hashes to include in the packfile.

#### Description

- Reads objects from the local object store.
- Compresses and delta-encodes objects for efficient storage.
- Writes a packfile with the appropriate header, object data, and trailer.
- Returns the packfile data stream or file path.

#### Example Usage

```python
packfile_data = create_packfile(repo, missing_objects)
```

---

### `send_packfile(session, packfile_data, ref_updates)`

#### Purpose

Sends the generated packfile and reference updates to the remote Git server through the authenticated session.

- **Parameters:**
  - `session`: Authenticated connection/session to the remote.
  - `packfile_data`: The binary data of the packfile.
  - `ref_updates`: Dictionary mapping reference names to new commit SHAs for updating remote refs.

#### Description

- Transmits the packfile using the Git pack protocol.
- Sends commands to update remote references after successful pack transfer.
- Receives and parses remote server response to confirm success or report errors.

#### Example Usage

```python
send_packfile(session, packfile_data, {'refs/heads/main': 'abc123...'})
```

---

## ASCII Diagram: Push Process Overview

```
Local Repository                     Remote Git Server
+----------------+                  +--------------------+
|                |                  |                    |
| 1. Authenticate+----------------->| 2. Verify credentials|
|                |                  |                    |
+--------+-------+                  +---------+----------+
         |                                    |
         | 3. Request Remote Refs             |
         +---------------------------------->|
         |                                    |
         |<----------------------------------+
         |                                    |
         | 4. Discover Missing Objects       |
         +---------------------------------->|
         |                                    |
         | 5. Create Packfile                 |
         +----------------------------------+
         |                                    |
         | 6. Send Packfile and Ref Updates  |
         +---------------------------------->|
         |                                    |
         |<----------------------------------+
         | 7. Confirm Push Success or Error  |
         +----------------------------------+
```

---

This documentation file offers a comprehensive guide to the `pygit.push` feature, enabling developers to understand and utilize the push functionality effectively within the pygit codebase. For related features, please refer to other files in the "Core Git Features" section, such as repository initialization and object handling.