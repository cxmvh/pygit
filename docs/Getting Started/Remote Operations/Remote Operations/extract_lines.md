# extract_lines.md

# Extracting Lines from Git Protocol Server Responses

## Overview

The `extract_lines.md` file documents the `extract_lines` function, which is a utility used during remote operations in the pygit library, specifically when handling responses from Git protocol servers. This function is critical in parsing the packet line data received from Git servers during push operations or other interactions involving the Git protocol over HTTP.

Within the broader documentation tree, this file resides under the **Remote Operations** section, alongside other files that manage communication with remote repositories (e.g., `http_request.md`, `get_remote_master_hash.md`, `build_lines_data.md`). It plays a foundational role in enabling the `pygit.push` command to correctly interpret server responses, ensuring reliable synchronization of local and remote repositories.

---

## Function: `extract_lines(data)`

### Purpose

`extract_lines` takes a binary data stream typically returned by a Git server following a push or fetch request and extracts the individual packet lines encoded within. Git protocol communication uses a packet line format, where each line or message is prefixed by its length in a 4-byte hexadecimal number, followed by the payload.

This function parses that format, returning a list of individual lines (as byte strings) that can be further processed by pygit functions to understand the server's response.

### Parameters

- `data` (`bytes`): The raw byte stream received from the Git server containing packet lines formatted per the Git protocol.

### Returns

- `List[bytes]`: A list of byte strings, each representing a decoded packet line from the server response.

### Preconditions

- The `data` must conform to the Git protocol packet line format:
  - Each packet line starts with a 4-byte ASCII hexadecimal length prefix.
  - A length of `0000` indicates a flush packet or end of data.
  - The length includes the 4 bytes for the length itself.

### How It Works

1. Initialize an empty list `lines` to hold the extracted lines.
2. Start at the beginning index `i = 0`.
3. Loop with an upper bound (e.g., 1000 iterations) to avoid infinite loops:
   - Read 4 bytes from `data[i:i+4]` and decode as a hex number to get the `line_length`.
   - Extract the line content from `data[i+4:i+line_length]`.
   - Append the extracted line to `lines`.
   - If `line_length` is zero (flush packet), increment `i` by 4.
   - Otherwise, increment `i` by `line_length`.
   - Break the loop if `i` reaches or exceeds the length of `data`.
4. Return the collected list of lines.

### Usage Example

```python
# Assume `response` is bytes received from a Git server after a push request
response = b'003f# service=git-receive-pack\n0000...'

# Extract lines from the response
lines = extract_lines(response)

# Print each extracted line (as bytes)
for line in lines:
    print(line)
```

### Example Output

```
b'# service=git-receive-pack\n'
b''
b'...'
```

---

## ASCII Diagram of Packet Line Parsing

```
+---------------------+----------------------------+
| 4-byte length prefix | Line content (length-4) bytes |
+---------------------+----------------------------+
|  0  |  0  |  3  | f |  #  |  s  |  e  |  r  | ... |
+-----+-----+-----+---+-----+-----+-----+-----+-----+
|<-------- length = 63 bytes in hex ----------->|
|<-------------- total length including prefix ------------------>|
```

The `extract_lines` function reads these packet lines sequentially until all lines are extracted or a flush packet (`0000`) is encountered.

---

## Integration with pygit.push Flow

In the `pygit.push` function, after sending a pack file to the remote Git server, the raw response is read and passed to `extract_lines`:

```python
response = http_request(url, username, password, data=data)
lines = extract_lines(response)
```

The extracted lines are then validated to ensure the push was successful:

```python
assert lines[0] == b'unpack ok\n'
assert lines[1] == b'ok refs/heads/master\n'
```

This demonstrates how `extract_lines` is essential for interpreting the server's status messages in the push workflow.

---

# Summary

- `extract_lines` is a key utility for parsing Git protocol packet lines from server responses.
- It supports reliable remote communication by decoding raw byte streams into discrete message lines.
- Used primarily in remote operations, such as during pushes (`pygit.push`), to interpret server feedback.
- Understanding this function is critical for anyone extending or debugging pygit's remote synchronization features.