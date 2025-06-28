# get_remote_master_hash.md

## Overview

This document describes the function `get_remote_master_hash`, which is responsible for fetching the current commit SHA-1 hash of the remote repository's `master` branch. This function plays a critical role in the push operation workflow (`pygit.push`), where it is necessary to compare the local master branch state with the remote state to identify new commits that need to be pushed.

Located within the **Remote Operations** section of the pygit documentation, this file complements other remote communication utilities such as `http_request`, `extract_lines`, and `build_lines_data`. Together, these functions facilitate interaction with the remote Git server to synchronize repository states securely and efficiently.

---

## Function: get_remote_master_hash

```python
def get_remote_master_hash(git_url, username, password):
    """Get commit hash of remote master branch, return SHA-1 hex string or
    None if no remote commits.
    """
    url = git_url + '/info/refs?service=git-receive-pack'
    response = http_request(url, username, password)
    lines = extract_lines(response)
    assert lines[0] == b'# service=git-receive-pack\n'
    assert lines[1] == b''
    if lines[2][:40] == b'0' * 40:
        return None
    master_sha1, master_ref = lines[2].split(b'\x00')[0].split()
    assert master_ref == b'refs/heads/master'
    assert len(master_sha1) == 40
    return master_sha1.decode()
```

### Purpose

`get_remote_master_hash` retrieves the SHA-1 hash string representing the latest commit on the remote repository's `master` branch. If no commits exist remotely (e.g., an empty repository), it returns `None`.

This function is essential for determining the synchronization state between the local and remote repositories, particularly before pushing new commits.

### Parameters

- `git_url` (str): The base URL of the remote Git repository (e.g., `https://github.com/user/repo.git`).
- `username` (str): Username for HTTP Basic Authentication.
- `password` (str): Password or token for HTTP Basic Authentication.

### Returns

- `str`: The 40-character SHA-1 hexadecimal string of the remote master commit.
- `None`: If the remote master branch has no commits.

### Preconditions

- The remote repository must support the Git smart HTTP protocol.
- Credentials must be valid to access the repository.
- The remote URL must be correctly formatted.

### How It Works (Step-by-Step)

1. **Construct URL**: The function appends the Git service endpoint `'/info/refs?service=git-receive-pack'` to the base `git_url`. This endpoint provides references info for pushing.

2. **Make HTTP Request**: Using `http_request`, it sends an authenticated GET request to the constructed URL and receives raw response data.

3. **Extract Lines**: The response data is parsed into packet lines using `extract_lines`. These packet lines follow the Git smart protocol format.

4. **Validate Headers**: The first two lines are checked:
   - The first line must indicate the service: `# service=git-receive-pack\n`.
   - The second line must be empty (`b''`).

5. **Check for Empty Remote**: The third line contains the SHA-1 and ref info for `master`. If the SHA-1 is all zeros (`'0' * 40`), it means no commits exist on the remote master branch, so return `None`.

6. **Parse SHA-1 and Reference**: The third line is split by the NUL (`\x00`) separator, then by spaces, to extract the SHA-1 hash string and the reference name.

7. **Validate Reference**: Assert that the reference corresponds to `refs/heads/master`.

8. **Return SHA-1**: Decode the SHA-1 bytes to a string and return it.

---

### Usage Example

```python
from pygit import get_remote_master_hash

git_url = "https://github.com/exampleuser/sample-repo.git"
username = "exampleuser"
password = "exampletoken"

remote_master_sha1 = get_remote_master_hash(git_url, username, password)

if remote_master_sha1 is None:
    print("Remote master branch has no commits.")
else:
    print(f"Remote master commit SHA-1: {remote_master_sha1}")
```

---

## Related Functions Overview

For a complete understanding of how `get_remote_master_hash` fits into the push process, see the following related functions:

- **`http_request(url, username, password, data=None)`**: Makes authenticated HTTP GET or POST requests to Git servers.

- **`extract_lines(data)`**: Parses Git protocol packet lines from server response data.

- **`push(git_url, username=None, password=None)`**: Orchestrates the full push operation, calling `get_remote_master_hash` to fetch the remote state before uploading missing objects.

---

## ASCII Diagram: Interaction Flow for Fetching Remote Master Hash

```
+----------------+          HTTP GET          +----------------------+
|    pygit       |  ----------------------->  | Remote Git Server     |
| (get_remote_master_hash)                       | (/info/refs)          |
+----------------+                              +----------------------+
         |                                                 |
         | <----------------------- Response -------------|
         |   (Git ref info in packet lines format)         |
         |                                                 |
         v                                                 v
+------------------------------+             +------------------------+
| Parse response lines          |             | Provide refs with SHA-1 |
| Validate service header       |             |                        |
| Extract master SHA-1          |             |                        |
+------------------------------+             +------------------------+
         |
         v
+------------------------------+
| Return SHA-1 hex string or    |
| None (if no remote commits)   |
+------------------------------+
```

---

This documentation provides the necessary details to understand and use the `get_remote_master_hash` function within the pygit project, particularly for remote synchronization operations such as pushing commits.