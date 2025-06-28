# build_lines_data.md

# Constructing Git Protocol Packet Lines for Sending Commands

## Overview

The `build_lines_data.md` file documents the function responsible for constructing Git protocol packet lines used when sending commands to a Git server. This is a crucial step in the Git remote communication process, particularly during operations like pushing commits to a remote repository. 

Within the broader "Remote Operations" section of the pygit documentation, this file complements related functionalities such as making HTTP requests (`http_request`), extracting response lines (`extract_lines`), and pushing changes (`push`). The packet line construction encoded here adheres to the Git protocol's packet line format, enabling proper framing and transmission of commands and data over the network.

---

## Function: `build_lines_data`

### Purpose

Constructs a byte string formatted according to the Git protocol packet line specification from a list of individual command or data lines. This formatted byte string is then sent to the Git server as part of remote procedure calls (e.g., during `git push`).

### Parameters

- `lines` (list of bytes): A list of byte strings, each representing a single line or command to send over the Git protocol.

### Returns

- `bytes`: A single byte string representing the concatenated packet lines, each prefixed with a length header and suffixed by a newline. The sequence ends with a flush packet (`0000`).

### Description

Git's wire protocol uses length-prefixed packet lines to frame commands and data sent between client and server. Each packet line consists of:

- A 4-byte hexadecimal length prefix, indicating the total length of the packet line including the length prefix itself.
- The payload line data (command or information).
- A newline character (`\n`).

A special flush packet of length `0` (`0000`) marks the end of a series of packet lines.

This function iterates over the provided lines, calculates each packet line's length as the length of the line plus 5 bytes (4 for the length prefix and 1 for the newline), formats the length as a 4-digit hexadecimal number, and concatenates all parts. Finally, it appends the flush packet to signal the end of the transmission.

### Step-by-Step Operation

1. Initialize an empty list `result` to accumulate packet line parts.
2. For each line in the input `lines`:
   - Calculate the total packet length: length of `line` + 5 (4 bytes length prefix + 1 byte newline).
   - Format the length as a 4-character hexadecimal string and encode to bytes.
   - Append the length prefix bytes, the line bytes, and a newline byte (`b'\n'`) to `result`.
3. After processing all lines, append the flush packet represented by `b'0000'` to `result`.
4. Join all parts in `result` into a single byte string and return it.

### Example Usage

```python
lines = [
    b'0000000000000000000000000000000000000000 0123456789abcdef0123456789abcdef01234567 refs/heads/master\x00 report-status',
    b'feature-branch 0123456789abcdef0123456789abcdef01234567 refs/heads/feature'
]

packet_data = build_lines_data(lines)
print(packet_data)
```

This example constructs Git protocol packet lines from two command lines: one updating the `master` branch and one for a `feature-branch`. The output `packet_data` can be sent directly over a network socket or HTTP POST to a Git server.

---

## ASCII Diagram: Git Protocol Packet Line Structure

```
+--------------------------------------------+
| 4-byte hex length | Payload Line | '\n'    |
+--------------------------------------------+
| e.g. '0032'       | b'command...'| b'\n'   |
+--------------------------------------------+
| Repeat for each line                     ...|
+--------------------------------------------+
| Flush packet: '0000' (indicates end)       |
+--------------------------------------------+
```

- Length includes itself and the newline.
- Flush packet signals no more data will be sent.

---

## Related Functions in Remote Operations

- `http_request(url, username, password, data=None)` — Sends HTTP requests with optional data payload.
- `extract_lines(data)` — Parses response data from Git server into individual lines.
- `push(git_url, username=None, password=None)` — High-level function orchestrating pushing commits to a remote.
- `get_remote_master_hash(git_url, username, password)` — Retrieves remote branch commit hash.
- `create_pack(objects)` — Creates a Git pack file to transmit objects efficiently.

---

# Summary

The `build_lines_data` function is a foundational utility in pygit's remote communication stack, encoding commands and data into the Git protocol's packet line format. Correct use of this function ensures proper framing of data for interactions like pushing commits, enabling pygit to operate with standard Git servers reliably.