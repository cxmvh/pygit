# Git Push Operation

This document details the **push** operation in the Git client implementation, describing how commits and associated objects are transmitted from a local repository to a remote repository. It covers key aspects such as authentication, retrieval of local and remote commit hashes, detection of missing objects that need to be sent, creation of Git packfiles containing those objects, and the HTTP interaction used to transmit data to the remote server.

The push operation is a critical step in synchronizing local changes with a remote repository and is closely related to other areas of the project such as packfile creation ([packfile_creation.md](./packfile_creation.md)) and object storage management ([object_handling.md](./object_handling.md)).

---

## Overview of the Push Process

The push operation consists of several stages:

1. **Authentication:** Establish credentials and authorization to communicate with the remote.
2. **Local and Remote Commit Hash Retrieval:** Determine the commit(s) currently present locally and remotely to identify what needs to be pushed.
3. **Missing Object Detection:** Identify which Git objects (commits, trees, blobs) are missing on the remote.
4. **Packfile Creation:** Package the missing objects efficiently into a single packfile.
5. **HTTP Interaction:** Send the packfile and update requests to the remote repository using Git's HTTP protocol.

The following sections describe the main function and supporting operations involved in this flow.

---

## `push(repository, remote_url, refspec, auth)`

### Purpose
The main function orchestrating the push operation by comparing local and remote repository states, packaging missing objects, and transmitting them securely.

### Parameters
- `repository`: The local Git repository object.
- `remote_url`: URL of the remote repository to push to.
- `refspec`: The reference specification indicating which refs (branches/tags) to update.
- `auth`: Authentication credentials (e.g., username/password or token).

### Preconditions
- The local repository must have a valid ref to push.
- Network connectivity to the remote must be available.
- Authentication credentials must be valid.

### Operation Steps

1. **Authenticate with the Remote:**
   - Use provided credentials to establish an authorized connection.
   
2. **Fetch Remote Refs:**
   - Query the remote repository for its current refs (commit hashes).
   
3. **Determine Local Refs:**
   - Read the current commit hashes for the specified refs locally.
   
4. **Calculate Missing Objects:**
   - Compare local and remote commit histories.
   - Traverse commits, trees, and blobs reachable from local commits but absent remotely.
   - Collect these objects for transfer.
   
5. **Create Packfile:**
   - Encode missing objects into a Git packfile.
   - This packfile efficiently bundles objects to minimize transfer size.
   
6. **Transmit Packfile and Update Refs:**
   - Send the packfile via an HTTP POST request.
   - Request remote to update refs to the new commit hashes after receiving objects.
   
7. **Verify Success:**
   - Confirm that the remote updated refs successfully.
   - Handle errors such as rejected pushes or authentication failures.

### Example Usage

```python
from pygit import Repository, push

repo = Repository("/path/to/local/repo")
remote_url = "https://github.com/user/repo.git"
refspec = "refs/heads/main"
auth = {"username": "user", "password": "token"}

push(repo, remote_url, refspec, auth)
```

---

## Supporting Functions

### `authenticate(remote_url, auth)`

**Purpose:**  
Establish a connection with the remote repository using the given credentials.

**Parameters:**
- `remote_url`: The remote repository URL.
- `auth`: Authentication information.

**Returns:**  
An authenticated HTTP session or raises an error if authentication fails.

**Example:**

```python
session = authenticate("https://github.com/user/repo.git", {"username": "user", "password": "token"})
```

---

### `get_remote_refs(session, remote_url)`

**Purpose:**  
Retrieve the current references and their commit hashes from the remote repository.

**Parameters:**
- `session`: Authenticated HTTP session.
- `remote_url`: Remote repository URL.

**Returns:**  
A dictionary mapping ref names to commit hashes, e.g.,

```python
{
  "refs/heads/main": "a1b2c3d4...",
  "refs/heads/dev": "d4c3b2a1..."
}
```

---

### `get_local_ref_commit(repository, ref)`

**Purpose:**  
Get the commit hash for a given local ref.

**Parameters:**
- `repository`: Local repository object.
- `ref`: Reference name, e.g., `refs/heads/main`.

**Returns:**  
Commit hash string.

---

### `find_missing_objects(local_commit, remote_commit, repository)`

**Purpose:**  
Identify objects present locally but missing remotely.

**Parameters:**
- `local_commit`: Local commit hash.
- `remote_commit`: Remote commit hash.
- `repository`: Local repository.

**Returns:**  
A set of object hashes that need to be pushed.

**Operation:**

- Traverse the commit graph from `local_commit` back to `remote_commit`.
- Collect all reachable objects (commits, trees, blobs) not present remotely.

---

### `create_packfile(objects, repository)`

**Purpose:**  
Generate a Git packfile containing all specified objects.

**Parameters:**
- `objects`: Iterable of object hashes.
- `repository`: Local repository to read object data.

**Returns:**  
Binary data representing the packfile.

---

### `send_packfile(session, remote_url, packfile_data, refspec)`

**Purpose:**  
Transmit the packfile and update the remote refs.

**Parameters:**
- `session`: Authenticated HTTP session.
- `remote_url`: Remote repository URL.
- `packfile_data`: Binary packfile contents.
- `refspec`: Reference update specification.

**Returns:**  
Response status confirming push success or failure.

---

## ASCII Diagram of Push Flow

```
+-----------------+                      +---------------------+
| Local Repository |                      |  Remote Repository   |
|                 |                      |                     |
| 1. Read local ref|                      |                     |
| 2. Authenticate |  <-- HTTP auth -->   |                     |
| 3. Get remote refs| <-- HTTP GET refs--|                     |
| 4. Calculate missing objects             |                     |
| 5. Create packfile                       |                     |
| 6. POST packfile + ref updates -------->|  Receive packfile   |
|                                         |  Update refs        |
|                                         |                     |
+-----------------+                      +---------------------+
```

---

This document complements the related files in the documentation tree such as:

- [packfile_creation.md](./packfile_creation.md) — for details on how packfiles are created.
- [object_handling.md](./object_handling.md) — for understanding Git object storage and retrieval.
- [commit.md](./commit.md) — for commit object creation before pushing.

For further details on interacting with Git objects and packfiles, consult those references.