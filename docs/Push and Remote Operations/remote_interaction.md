# remote_interaction.md

## Overview

This file details the mechanisms for interacting with remote Git repositories, focusing on fetching remote references, handling authentication, and communicating over HTTP. It is part of the "Push and Remote Operations" section, which encompasses the processes to push commits, create pack files, and manage network protocols. The functions documented here enable querying the remote repository state, preparing data for pushing, and securely transmitting updates. This documentation serves as a reference for developers implementing or extending remote synchronization features in the Git implementation.

---

## Function Documentation

### `push(git_url, username=None, password=None)`

**Purpose:**  
Push the local `master` branch to the remote Git repository located at `git_url`. This function coordinates the entire push process: it authenticates, compares local and remote commits, finds missing objects, creates a pack file, and sends the data to the remote server.

**Parameters:**  
- `git_url` (str): The URL of the remote Git repository.  
- `username` (str, optional): Username for HTTP Basic authentication. Defaults to environment variable `GIT_USERNAME`.  
- `password` (str, optional): Password for HTTP Basic authentication. Defaults to environment variable `GIT_PASSWORD`.

**Operation:**  
1. If `username` or `password` are not provided, retrieve them from environment variables.  
2. Obtain the current remote `master` branch commit hash using `get_remote_master_hash`.  
3. Get the local `master` branch commit hash using `get_local_master_hash`.  
4. Determine which objects are missing at the remote by comparing the commits with `find_missing_objects`.  
5. Print status about the update from remote commit to local commit, including the count of objects to push.  
6. Prepare protocol lines indicating the refs update request.  
7. Build the data payload combining these lines and a pack file containing missing objects.  
8. Send a POST request to the remote `/git-receive-pack` endpoint with authentication.  
9. Receive the response, parse the packet lines, and verify successful unpacking and acknowledgment.  
10. Return a tuple with the remote SHA-1 commit and the set of missing objects pushed.

**Example Usage:**

```python
remote_url = "https://example.com/myrepo.git"
username = "alice"
password = "secret123"

remote_sha1, pushed_objects = push(remote_url, username, password)
print(f"Pushed {len(pushed_objects)} objects to remote commit {remote_sha1}")
```

---

### `get_remote_master_hash(git_url, username, password)`

**Purpose:**  
Retrieve the commit hash (SHA-1 hex string) of the remote repository's `master` branch. Returns `None` if there are no commits on the remote branch.

**Parameters:**  
- `git_url` (str): Remote repository URL.  
- `username` (str): Username for authentication.  
- `password` (str): Password for authentication.

**Operation:**  
1. Construct the URL to fetch refs info: `git_url + '/info/refs?service=git-receive-pack'`.  
2. Make an authenticated HTTP GET request to this URL via `http_request`.  
3. Parse the response packet lines using `extract_lines`.  
4. Confirm the first lines indicate the `git-receive-pack` service and an empty line.  
5. Check if the remote commit hash line contains all zeroes, indicating no commits; if so, return `None`.  
6. Otherwise, extract the remote master SHA-1 hash from the line.  
7. Validate the ref name is `refs/heads/master` and the SHA-1 length is 40 characters.  
8. Return the SHA-1 string.

**Example Usage:**

```python
remote_sha1 = get_remote_master_hash("https://example.com/myrepo.git", "alice", "secret123")
if remote_sha1:
    print(f"Remote master commit: {remote_sha1}")
else:
    print("Remote repository has no commits yet.")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash (SHA-1 hex string) of the local `master` branch.

**Parameters:**  
None.

**Operation:**  
1. Read the contents of `.git/refs/heads/master`.  
2. Return the trimmed commit hash string or `None` if the file does not exist.

**Example Usage:**

```python
local_sha1 = get_local_master_hash()
if local_sha1:
    print(f"Local master commit: {local_sha1}")
else:
    print("No commits on local master branch.")
```

---

### `find_missing_objects(local_sha1, remote_sha1)`

**Purpose:**  
Determine which Git objects (commits, trees, blobs) exist in the local commit but are absent in the remote commit, i.e., which objects need to be pushed.

**Parameters:**  
- `local_sha1` (str): SHA-1 hash of the local commit.  
- `remote_sha1` (str or None): SHA-1 hash of the remote commit (or None if no remote commits).

**Operation:**  
1. Retrieve all objects reachable from the local commit using `find_commit_objects`.  
2. If `remote_sha1` is None, return all local objects (since remote is empty).  
3. Otherwise, retrieve all remote commit objects similarly.  
4. Return the set difference: local objects minus remote objects.

**Example Usage:**

```python
missing = find_missing_objects(local_sha1="abc123...", remote_sha1="def456...")
print(f"Objects to push: {len(missing)}")
```

---

### `create_pack(objects)`

**Purpose:**  
Create a Git pack file containing all the objects specified by their SHA-1 hashes. This pack file is what is sent to the remote repository during a push.

**Parameters:**  
- `objects` (set of str): Set of SHA-1 hash strings representing Git objects to include.

**Operation:**  
1. Construct the pack file header with signature `PACK`, version 2, and object count.  
2. Encode each object with `encode_pack_object`, concatenate all encoded objects as the pack body.  
3. Compute SHA-1 checksum of the header + body and append it.  
4. Return the complete pack file as bytes.

**Example Usage:**

```python
pack_data = create_pack(missing_objects)
with open("objects.pack", "wb") as f:
    f.write(pack_data)
```

---

### `build_lines_data(lines)`

**Purpose:**  
Build a byte string from a list of packet lines to send to the Git server, formatted according to the Git Smart HTTP protocol.

**Parameters:**  
- `lines` (list of bytes): List of lines to encode.

**Operation:**  
1. For each line, prefix it with a 4-digit hex length (line length + 5 bytes).  
2. Append the line and a newline character.  
3. Append a flush packet (`0000`) at the end.  
4. Return concatenated bytes.

**Example Usage:**

```python
lines = [b'0000000000000000000000000000000000000000 0123456789abcdef refs/heads/master\x00 report-status']
data = build_lines_data(lines)
```

---

### `extract_lines(data)`

**Purpose:**  
Parse a response from a Git HTTP server, extracting individual packet lines from the raw data.

**Parameters:**  
- `data` (bytes): Raw data from server response.

**Operation:**  
1. Read 4 hex digits indicating packet length.  
2. Extract the line of that length minus 4 bytes.  
3. Continue until all lines are extracted or a zero-length packet is found.

**Example Usage:**

```python
lines = extract_lines(response_data)
for line in lines:
    print(line)
```

---

### `http_request(url, username, password, data=None)`

**Purpose:**  
Perform an authenticated HTTP request to the specified URL. Uses HTTP Basic authentication. Uses GET if `data` is None, otherwise POST with `data` as body.

**Parameters:**  
- `url` (str): Target URL.  
- `username` (str): Username for authentication.  
- `password` (str): Password for authentication.  
- `data` (bytes, optional): Data to POST. If None, a GET request is made.

**Operation:**  
1. Create a password manager and add credentials.  
2. Build an opener with HTTP Basic Auth handler.  
3. Open the URL with optional data.  
4. Return the response bytes.

**Example Usage:**

```python
response = http_request("https://example.com/myrepo.git/info/refs?service=git-receive-pack", "alice", "secret123")
print(response)
```

---

### ASCII Diagram: Push Operation Overview

```
+-------------------+         +---------------------+
|  Local Repository  |         |  Remote Repository   |
|                   |         |                     |
|  get_local_master_hash        |
|               |             |
|               +------------>| get_remote_master_hash
|                             |
| find_missing_objects         |
|               |             |
| create_pack                  |
|               |             | 
| push data via HTTP POST      |
|  (git-receive-pack)          |
|               |------------>|
|                             |
| <------------ unpack ok      |
| <------------ ok refs/heads/master
|                             |
+-------------------+         +---------------------+
```

---

This documentation file covers the crucial functions involved in communicating with remote Git repositories to push commits securely and efficiently via HTTP. For related details on pack file creation, commit management, and index handling, refer to the other files in the "Push and Remote Operations" and related sections of the documentation tree.