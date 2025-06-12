# HTTP.md - HTTP Communication Functions Documentation

---

## Overview

The `http.md` file documents the functions responsible for handling HTTP communication related to Git remote operations within the `pygit` repository. These functions are primarily used during `push` operations to send data to remote Git repositories and parse responses. This file focuses on `http_request` for performing authenticated HTTP requests and `extract_lines` for parsing HTTP response data encoded in Git’s pkt-line format. These utilities are essential for enabling communication between the local Git client and remote Git servers over HTTP(S).

Within the broader documentation tree, `http.md` resides in the **HTTP Communication** section, which details the network interaction layer of `pygit`, supporting the core Git commands documented elsewhere, such as `push.md`. The HTTP functions enable secure and correct transmission of Git objects and refs, ensuring the integrity and success of remote operations.

---

## Function Documentation

### 1. `http_request(url, username, password, data=None)`

#### Purpose

Performs an authenticated HTTP request to a specified URL. If `data` is provided, the request is a POST with the given data as the request body; otherwise, it defaults to a GET request. Authentication uses HTTP Basic Auth with the provided username and password.

#### Parameters

- `url` (`str`): The target HTTP(S) URL for the request.
- `username` (`str`): Username for HTTP Basic Authentication.
- `password` (`str`): Password for HTTP Basic Authentication.
- `data` (`bytes` or `None`): Optional data to send as the POST request body. If `None`, a GET request is made.

#### Operation

1. Create a password manager and add the username/password credentials for the given URL.
2. Set up an HTTP Basic Auth handler with the password manager.
3. Build an opener with the auth handler.
4. Open the URL with or without data (POST if data present).
5. Read and return the raw response bytes from the server.

#### Example Usage

```python
response_data = http_request(
    url="https://example.com/git-receive-pack",
    username="gituser",
    password="secretpass",
    data=b"PACK data bytes here..."
)
print(f"Received {len(response_data)} bytes from server")
```

---

### 2. `extract_lines(data)`

#### Purpose

Extracts and returns a list of lines from the raw HTTP response data returned by Git servers, which use the pkt-line protocol format. Each line is prefixed with a 4-byte hexadecimal length header. This function decodes these length-prefixed lines into a list of raw byte strings.

#### Parameters

- `data` (`bytes`): Raw byte string response content from a Git server.

#### Operation

1. Initialize a read index `i`.
2. Loop up to 1000 times (arbitrary limit to avoid infinite loops).
3. Read the next 4 bytes starting at `i` and interpret as hex length (`line_length`).
4. If `line_length` is 0, this marks the end of the stream; increment `i` by 4.
5. Otherwise, extract bytes from `i+4` to `i+line_length` as the line.
6. Append the extracted line to the `lines` list.
7. Advance `i` by `line_length`.
8. Stop if the index exceeds data length.
9. Return the list of extracted lines.

#### Example Usage

```python
response = b"001e# service=git-receive-pack\n0000"
lines = extract_lines(response)
for line in lines:
    print(line.decode())
```

Output:

```
# service=git-receive-pack

```

---

## ASCII Diagram: Pkt-Line Format Parsing by `extract_lines`

```
+----------------------------+
| Raw Data (bytes)            |
|                            |
|  4-byte length header       |  =>  hex length (e.g. "001e" -> 30 bytes)
|  N-byte content line        |  =>  Extract N-4 bytes as content
|  4-byte length header       |
|  N-byte content line        |
|  ...                       |
+----------------------------+

Parsing steps:

[ i ] -> read 4 bytes -> length_header = data[i:i+4]
          |
          v
length = int(length_header, 16)
if length == 0: end of lines
else:
    read line = data[i+4 : i+length]
    append line to list
    i += length
repeat
```

---

## Integration in `pygit.push`

The `http_request` and `extract_lines` functions are utilized in the `push` command to communicate with the remote Git server:

- `http_request` sends the packfile and reference update info to the remote.
- The raw response is passed to `extract_lines` to parse Git server responses such as `unpack ok` and `ok refs/heads/master`.

Example snippet from `push` flow:

```python
response = http_request(url, username, password, data=pack_data)
lines = extract_lines(response)
assert lines[0] == b'unpack ok\n'
assert lines[1] == b'ok refs/heads/master\n'
```

This confirms successful receipt and application of the pushed objects.

---

# Summary

| Function         | Description                                  | Typical Use Case                     |
|------------------|----------------------------------------------|------------------------------------|
| `http_request`   | Perform authenticated GET/POST HTTP requests | Send Git commands/data to remotes  |
| `extract_lines`  | Parse pkt-line encoded response data          | Decode Git server response packets |

Together, these functions form the communication backbone for remote Git operations over HTTP in the `pygit` project.

---

# End of `http.md` Documentation File Content