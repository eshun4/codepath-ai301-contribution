# codepath-ai301-contribution# CodePath AI301 Open Source Contribution

## Student

Kofi Eshun

## Selected Issue

https://github.com/pwndbg/pwndbg/issues/1551

## Project

pwndbg

## Issue Summary

This issue focuses on improving `lib/net.py` by adding more functionality and unit tests, especially around Unix Domain Socket parsing. This matters because the code helps pwndbg correctly understand process networking and socket information while debugging.

## Why I Chose This Issue

I chose this issue because a previous related PR specifically mentioned that a test should be added for this parsing behavior, giving me a clear and focused starting point. My first step will be to understand the existing parsing behavior and add a small unit test for the Unix socket case.

## Current Phase

Phase I: Issue Selection

Phase II: Reproduction Process
1. I forked the `pwndbg` repository and cloned my fork locally.
2. Added the original `pwndbg` repository as upstream.
3. Checked out the dev branch and created a working branch named `test-lib-net-unix-parsing`.
4. Ran `./setup.sh` to setup the local development environment,
5. Activated the virtual environment with `source .venv/bin/activate`.
6. Locted the Unix socket parsing function `pwndbg/lib/net.py`: `def unix(data: str) => list[UnixSocket]`.
7. Confirmed that the parse intentionally uses `data.split("\n") instead of `.splitlines()` because Unix socket paths may contain `\r`.
8. Ran a small localPython reproduction script using a fake `/proc/net/unix` -style input containing a socket path with`\r`.
9. Lastly, then I verified that:
    a. The parser returned exactly 1 socket
    b. The inode was parsed correctly
    c. The socket path still contained `\r`.

Evidence:
https://github.com/eshun4/pwndbg/tree/test-lib-net-unix-parsing

Reproduction output:
1. `number of sockets: 1`.
2. `inode: 23302`
3. `contains carriage return: True`


Implementation Plan
My plan is to add a unit test for `pwndbg.lib.net.unix()` covering the Unit socket parsing case where the socket path contains a carriage return (`\r`). The test should help me verify that the parser does not incorrectly split the socket entry into multiple lines, and that also it correctly preserves the path and parses the inode. First off, I will look for the preferred unit test location and existing test patterns in `tests/unit_tests/`, then add the smallest possible test for this behavior. 
