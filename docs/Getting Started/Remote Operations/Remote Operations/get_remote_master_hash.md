# get_remote_master_hash.md

## Overview

The `get_remote_master_hash` module provides functionality for fetching the SHA-1 commit hash of the remote Git repository's `master` branch. This is a critical step in synchronizing local and remote repositories, particularly used during push operations to determine which objects are missing remotely and need to be sent. Within the overall documentation tree, this file resides under **Remote Operations**, highlighting its role in communication and synchronization with remote Git servers.

This document details the `get_remote_master_hash` function, explaining how it queries the remote repository, processes the Git protocol response, and extracts the commit hash of the remote `master` branch. Usage examples are included to demonstrate its integration with the `pygit.push` workflow.

---

## Function: `get_remote_master_hash(git_url, username, password)`

### Purpose

Fetch the SHA-1 hash string of the latest commit on the remote repository's `master` branch. If the remote repository has no commits on the `master` branch, this function returns `None`.

This function is essential for determining the current state of the remote `master` branch before pushing new commits, enabling the client to compute the difference between local and remote commits.

### Parameters

- **git_url** (`str`): The URL of the remote Git repository (e.g., `'https://github.com/user/repo.git'`).
- **username** (`str`): Username for HTTP basic authentication.
- **password** (`str`): Password for HTTP basic authentication.

### Returns

- `str` or `None`: Returns the 40-character hexadecimal SHA-1 commit hash of the remote `master` branch if it exists; otherwise, returns `None`.

### Preconditions

- The remote repository supports the Git "smart" HTTP protocol.
- Proper credentials are provided for HTTP authentication if required.
- Network connectivity to the remote Git server is available.

### How It Works (Step-by-Step)

1. **Construct the URL**  
   The function appends the Git service query string `/info/refs?service=git-receive-pack` to the `git_url` to request a service advertisement of refs available for pushing.

2. **Make an HTTP Request**  
   It calls `http_request()` (another function in the pygit module) with the constructed URL and authentication credentials, which performs an HTTP GET request and returns the raw server response bytes.

3. **Extract Packet Lines**  
   The raw response data uses the Git pkt-line protocol. The function calls `extract_lines()` to parse this data into a list of byte strings representing individual lines sent by the server.

4. **Validate Protocol Response**  
   It asserts that the first two lines match the Git protocol service announcement and a blank line:
   - Line 0 must be `b'# service=git-receive-pack\n'`
   - Line 1 must be an empty line `b''`

5. **Parse the Reference Line**  
   Line 2 contains information about the remote refs. It is split on the null byte (`\x00`), then on spaces to separate the commit hash and the ref name.  
   If the commit hash is 40 zeroes (`'0' * 40`), it indicates no commits on the remote master branch.

6. **Return the Remote SHA-1**  
   The function ensures the reference is `refs/heads/master`, verifies the SHA-1 length, and returns the decoded SHA-1 string.

### Example Usage

```python
from pygit import get_remote_master_hash

git_url = 'https://github.com/exampleuser/example-repo.git'
username = 'myusername'
password = 'mypassword'

remote_master_sha1 = get_remote_master_hash(git_url, username, password)

if remote_master_sha1 is None:
    print("Remote master branch has no commits yet.")
else:
    print(f"Remote master branch commit hash: {remote_master_sha1}")
```

---

## Related Functions in Remote Operations

- **http_request(url, username, password, data=None)**  
  Makes authenticated HTTP GET or POST requests to the Git server.

- **extract_lines(data)**  
  Parses the Git pkt-line formatted response data into individual lines.

- **push(git_url, username=None, password=None)**  
  Uses `get_remote_master_hash` to determine remote state before pushing commits.

---

## ASCII Diagram: Git Protocol Interaction for Fetching Remote Master Hash

```
Client (get_remote_master_hash)
    |
    | HTTP GET /info/refs?service=git-receive-pack
    |-------------------------------------------->
    |
    | <--------------------------------------------
    | # service=git-receive-pack\n
    | \n
    | <40-byte SHA-1> refs/heads/master\x00<capabilities>
    |
Client parses response, extracts SHA-1 of remote master branch.
```

---

## Full Function Source Code

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

---

This completes the documentation for `get_remote_master_hash.md`. It serves as a foundational building block in remote repository synchronization workflows, particularly when pushing commits using the `pygit.push` command.