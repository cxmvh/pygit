# push.md

# Push Command Documentation

## Overview

The `push` command in pygit is responsible for updating the remote repository's master branch with commits from the local repository. It ensures that the remote branch reflects the latest local commits by transferring any missing Git objects via the Git packfile protocol over HTTP(S). This command is a critical part of the remote operations section within the pygit project, enabling synchronization between local and remote repositories.

Within the overall documentation tree, `push.md` fits under "Git Operations" → "Remote Operations" and is closely related to functions handling object storage, commit creation, and HTTP communication. It relies on multiple supporting functions such as obtaining local and remote commit hashes, finding missing objects, creating packfiles, and performing authenticated HTTP requests.

---

## Important Functions

### 1. `push(git_url, username=None, password=None)`

Push the local master branch commits to the specified remote Git repository URL.

#### Purpose

- Updates the remote `refs/heads/master` branch to match the local master branch.
- Transfers all missing commit objects from the local to the remote repository.
- Performs authentication using provided username and password or environment variables.

#### Parameters

- `git_url` (str): The URL of the remote Git repository.
- `username` (str, optional): Username for HTTP authentication. Defaults to environment variable `GIT_USERNAME`.
- `password` (str, optional): Password for HTTP authentication. Defaults to environment variable `GIT_PASSWORD`.

#### Operation Details

1. Retrieve remote master branch commit hash using `get_remote_master_hash`.
2. Retrieve local master commit hash using `get_local_master_hash`.
3. Identify missing commit objects that exist locally but not remotely via `find_missing_objects`.
4. Prepare Git protocol "push" lines and create a packfile containing missing objects.
5. Send an authenticated HTTP POST request with the packfile to `/git-receive-pack` endpoint.
6. Parse the server response to confirm success.
7. Return a tuple of `(remote_sha1, missing_objects)` indicating the previous remote hash and the objects pushed.

#### Usage Example

```python
from pygit import push

remote_url = "https://example.com/myrepo.git"
# Optional: set environment variables or pass username/password explicitly
result = push(remote_url)
remote_hash, pushed_objects = result
print(f"Pushed {len(pushed_objects)} objects to remote commit {remote_hash or 'none'}.")
```

#### ASCII Diagram: Push Process Flow

```
Local Master Commit Hash ----+
                             |---> Determine missing objects ---> Create packfile
Remote Master Commit Hash ----+
                             |
                             +---> Send push request (packfile) to remote git URL
                                         |
                                         v
                                Remote server updates master branch
                                         |
                                         v
                                Receive confirmation response
```

---

### 2. `get_remote_master_hash(git_url, username, password)`

Fetches the SHA-1 hash of the remote repository's master branch.

#### Purpose

- Queries the remote Git server for the current commit hash of `refs/heads/master`.
- Returns `None` if the remote has no commits.

#### Parameters

- `git_url` (str): Remote repository URL.
- `username` (str): Username for HTTP authentication.
- `password` (str): Password for HTTP authentication.

#### How It Works

- Sends an HTTP GET to `git_url + '/info/refs?service=git-receive-pack'`.
- Extracts lines from the response using `extract_lines`.
- Validates protocol headers.
- Parses the hash corresponding to `refs/heads/master`.
- Returns the hash as a hex string or `None` if the remote is empty.

#### Usage Example

```python
remote_url = "https://example.com/myrepo.git"
username = "user"
password = "pass"

remote_hash = get_remote_master_hash(remote_url, username, password)
if remote_hash:
    print(f"Remote master hash: {remote_hash}")
else:
    print("Remote repository has no commits.")
```

---

### 3. `get_local_master_hash()`

Retrieves the current commit hash of the local master branch.

#### Purpose

- Reads the SHA-1 commit hash stored in `.git/refs/heads/master`.
- Returns `None` if the local master branch does not exist.

#### Parameters

None

#### How It Works

- Reads the commit hash from `.git/refs/heads/master`.
- Decodes and strips whitespace.
- Handles `FileNotFoundError` gracefully.

#### Usage Example

```python
local_hash = get_local_master_hash()
if local_hash:
    print(f"Local master commit: {local_hash}")
else:
    print("No local master commit found.")
```

---

### 4. `find_missing_objects(local_sha1, remote_sha1)`

Finds commit objects present in the local commit but missing on the remote.

#### Purpose

- Computes the set difference between local commit's objects and remote commit's objects.
- Returns all local objects if remote has no commits.

#### Parameters

- `local_sha1` (str): SHA-1 hash of the local commit.
- `remote_sha1` (str or None): SHA-1 hash of the remote commit or None if no remote commits.

#### How It Works

- Uses `find_commit_objects` to recursively gather all objects reachable from the commit.
- Returns the set difference: local objects minus remote objects.

#### Usage Example

```python
missing = find_missing_objects(local_sha1, remote_sha1)
print(f"Missing objects to push: {len(missing)}")
```

---

### 5. `build_lines_data(lines)`

Constructs a Git protocol packet lines byte sequence for sending commands.

#### Purpose

- Converts a list of byte strings representing Git protocol lines into the packet line format.

#### Parameters

- `lines` (List[bytes]): List of lines to encode.

#### How It Works

- Prepends each line with its length in 4 hex digits.
- Appends a terminating flush packet: `0000`.

#### Usage Example

```python
lines = [b'0000 HEAD\x00 report-status']
data = build_lines_data(lines)
```

---

### 6. `create_pack(objects)`

Creates a Git packfile containing the specified objects.

#### Purpose

- Packages multiple Git objects into a single compressed packfile for efficient transfer.

#### Parameters

- `objects` (Set[str]): Set of SHA-1 hashes of objects to include.

#### How It Works

- Packs header with signature, version, and object count.
- Encodes each object with `encode_pack_object`.
- Appends SHA-1 checksum of the packfile contents.

#### Usage Example

```python
pack_data = create_pack(missing_objects)
with open('packfile.pack', 'wb') as f:
    f.write(pack_data)
```

---

### 7. `http_request(url, username, password, data=None)`

Performs an authenticated HTTP request to the Git server.

#### Purpose

- Sends either GET (if `data` is None) or POST (if `data` is given) requests with basic HTTP authentication.

#### Parameters

- `url` (str): URL to request.
- `username` (str): Username for authentication.
- `password` (str): Password for authentication.
- `data` (bytes or None): Data to POST; if None, a GET is performed.

#### How It Works

- Uses `urllib.request` with HTTPBasicAuthHandler.
- Returns raw response bytes.

#### Usage Example

```python
response = http_request('https://example.com/git-receive-pack', 'user', 'pass', data=pack_data)
```

---

### 8. `extract_lines(data)`

Extracts packet lines from Git protocol server response data.

#### Purpose

- Parses a byte stream of Git packet lines into a list of byte strings.

#### Parameters

- `data` (bytes): Raw response data from Git server.

#### How It Works

- Reads length prefix (4 hex digits).
- Extracts the indicated number of bytes for each line.
- Stops on flush packet (length zero).

#### Usage Example

```python
lines = extract_lines(response)
for line in lines:
    print(line.decode())
```

---

## Summary

The `push` command and its associated functions encapsulate the full process of pushing local commits to a remote repository in pygit. It manages authentication, object discovery, packfile creation, and communication with the Git server, ensuring that the remote master branch accurately reflects the local state.

By understanding these functions and their interplay, developers can extend, debug, or integrate pygit's push functionality effectively.

---

# Additional Notes

- The push implementation currently supports only the `master` branch.
- Authentication credentials are handled via environment variables or explicit parameters.
- The packfile format and Git protocol packet lines follow Git's official specifications.
- Error handling includes assertions to verify server responses and protocol correctness.