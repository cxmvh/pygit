# push.md

## Overview

This document provides detailed technical reference information about the `push` function in the `pygit` module. The `push` function is responsible for sending local commits and objects to a remote Git repository, enabling synchronization between local and remote repositories. This involves packing commit objects, authenticating with the remote server, and managing communication protocols such as HTTP or SSH. 

In the broader documentation hierarchy, this file is part of the **Git Push Command** section, which covers the mechanisms behind pushing changes to remotes. It complements related documentation such as `remote_communication.md`, which focuses on the underlying communication protocols and HTTP interactions used during push operations.

---

## Function: `push`

### Purpose

The `push` function transmits commits, trees, blobs, and tags from the local repository to a remote repository. It ensures that all necessary objects are packed and sent in an efficient manner and that authentication and network communication with the remote server are handled correctly. This function is essential for collaborative workflows, allowing developers to share their changes with others.

### Parameters

- `remote_url` (str): The URL of the remote Git repository to push to.
- `refspecs` (list of str): A list of reference specifications indicating which local branches or tags to push and their corresponding remote destinations. E.g., `["refs/heads/main:refs/heads/main"]`
- `auth` (optional, dict or callable): Authentication credentials or a callback to handle authentication challenges (e.g., username/password or SSH keys).
- `progress_callback` (optional, callable): A function to receive progress updates during the push.
- `pack_objects` (bool, default=True): Whether to pack objects before sending.
- `timeout` (int or float, optional): Network operation timeout in seconds.

### Preconditions

- The local repository must have commits and objects corresponding to the references specified in `refspecs`.
- Network connectivity to the remote repository must be available.
- Proper authentication credentials must be provided or accessible.

### Operation Steps

1. **Resolve Refs and Objects to Push:**
   - Identify commits, trees, blobs, and tags reachable from the local references specified in `refspecs`.
   - Determine which objects the remote repository already has to avoid redundant transfers.

2. **Pack Objects:**
   - Create a packfile that bundles all required objects efficiently.
   - The packfile format is optimized for transport and storage.

3. **Authenticate:**
   - Establish a connection to the remote repository using the provided URL.
   - Perform authentication using the provided credentials or callback mechanism.
   
4. **Send Packfile:**
   - Transmit the packfile over the network to the remote server.
   - Handle responses and errors according to the Git protocol.
   
5. **Update Remote References:**
   - Request the remote repository to update its references to point to the newly pushed commits.
   - Confirm success or handle rejection due to conflicts or permission issues.

6. **Report Progress:**
   - Continuously update the caller via `progress_callback` about the push status (e.g., objects packed, sent, refs updated).

---

### Example Usage

```python
from pygit import push

remote = "https://github.com/exampleuser/myrepo.git"
refs = ["refs/heads/main:refs/heads/main"]

def progress_handler(message):
    print(f"Push progress: {message}")

auth_credentials = {
    'username': 'exampleuser',
    'password': 'examplepassword'
}

# Push the main branch to the remote repository with authentication and progress reporting
push(
    remote_url=remote,
    refspecs=refs,
    auth=auth_credentials,
    progress_callback=progress_handler,
    pack_objects=True,
    timeout=30
)
```

---

### ASCII Diagram: Push Operation Flow

```
+-----------------+        +----------------+        +--------------------+
| Local Repository | ---->  | Pack Objects   | ---->  | Network Connection  |
| (commits, refs)  |        | (packfile)     |        | (HTTP/SSH)          |
+-----------------+        +----------------+        +--------------------+
                                                            |
                                                            v
                                                  +--------------------+
                                                  | Remote Repository   |
                                                  | (receive packfile,  |
                                                  | update refs)        |
                                                  +--------------------+
```

---

### Notes

- The `push` function leverages Git's packfile format for efficient data transfer.
- Authentication methods can vary based on remote URL schemes (HTTP Basic Auth, SSH keys, OAuth tokens).
- Handling errors such as rejected pushes due to non-fast-forward updates is part of the function's responsibility.
- For low-level communication details, refer to [`remote_communication.md`](remote_communication.md).

---

This concludes the technical reference for the `push` function in the `pygit` module. For additional details on remote protocols and authentication mechanisms, see the related documentation files in the **Git Push Command** section.