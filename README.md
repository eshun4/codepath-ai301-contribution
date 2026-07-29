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

Phase III: Build

I added a focused regression test for `pwndbg.lib.net.unix()` in `tests\unit_tests\test_net.py` . The test covers the Unix socket parsing case where the socket path contains a carriage return (\r). This verifies that the parser preserves the full socket path and does not incorrectly split the entry into multiple socket records.

Code Changes

Working branch:
`https://github.com/eshun4/pwndbg/tree/test-lib-net-unix-parsing

Commit:
`cd56d3b1e` - Add unix socket parsing regression test

Files changed:

tests/unit_tests/test_net.py
Testing Strategy

I ran the project unit test script:

`./unit-tests.sh tests/unit_tests/test_net.py -k unix`



This confirmed that the new Unix socket parsing regression test passes and that the existing unit tests still pass.
My plan is to add a unit test for `pwndbg.lib.net.unix()` covering the Unit socket parsing case where the socket path contains a carriage return (`\r`). The test should help me verify that the parser does not incorrectly split the socket entry into multiple lines, and that also it correctly preserves the path and parses the inode. 

I will start off by looking for the preferred unit test location and existing test patterns in `tests/unit_tests/`, then add the smallest possible test for this behavior. 

## Phase IV: Submit & Iterate

### Pull Request

PR Link: https://github.com/pwndbg/pwndbg/pull/3993

### PR Summary

I submitted a pull request to `pwndbg/pwndbg` adding a focused regression test for `pwndbg.lib.net.unix()`. The test verifies that Unix socket paths containing a carriage return (`\r`) are preserved correctly and are not split into multiple socket entries.

### Testing

I ran the project unit test script:

```bash
./unit-tests.sh tests/unit_tests/test_net.py -k unix
```

Result:

```text
68 passed, 4 skipped
```

### Maintainer Feedback / Next Steps

The PR has been submitted and is currently awaiting maintainer review. If maintainers request changes, I will update the branch, push a follow-up commit, and document the feedback here.

### Status

Maintainer Feedback / Next Steps

Maintainer @disconnect3d reviewed the PR and requested changes. He pointed out that while my test covered \r preservation, socket paths can also contain \n characters — and because the parser splits on \n, such paths break the line-based parsing entirely. He asked me to extend the test to cover a path containing both \r and \n, and to fix the underlying bug. He suggested an approach: when the parser encounters an unparsable line, treat it as a continuation of the previous entry's path.
How I Responded

First, I wrote a test feeding a multi-line path into the parser and confirmed it crashed with an IndexError — reproducing the bug the maintainer described.
I initially reverted to a valid-input-only test, but realized this sidestepped the review: the maintainer had asked for a fix, not just an acknowledgment. I went back and implemented the fix properly.

I added a `_is_unix_entry() helper to pwndbg/lib/net.py that validates whether a line matches the expected /proc/net/unix entry format (hex slot number ending with :, five hex fields, decimal inode). Lines that fail validation are appended to the previous socket's path with the \n restored.
I extended the tests to cover: a path containing both `\r` and `\n` spanning multiple lines, a pathless (anonymous) entry following it, and empty input.

Debugging Notes

Getting to green took several iterations, mostly due to transcription errors I introduced while editing (fields[1:16] instead of fields[1:6], a missing `prev_had_path` assignment, typos in test assertions). Each failure's traceback pointed at the cause. Key lessons: read git diff line-by-line before running tests, and test failures that share a pattern (e.g., "only entries with paths fail") point at a single root cause. I also learned that UnixSocket.path defaults to "(anonymous)" rather than None, which required using a flag (prev_had_path) instead of checking the attribute — you can't distinguish a pathless entry from the attribute alone.

Known Limitation
Entry detection is a heuristic: the kernel writes socket paths verbatim with no escaping, so a path fragment could theoretically mimic a valid entry line. I noted this in my reply to the maintainer.

Testing
`./unit-tests.sh tests/unit_tests/test_net.py`

Result: `71 passed, 4 skipped`

Status
Fix and tests pushed; rebased on upstream dev; re-requested review from @disconnect3d.

First PR Approved and Merged.
The maintainer reviewed the code, approved my Pull request and merged my changes.


## Selected Issue #2

https://github.com/vineethwilson15/codemind/issues/31

## Project

CodeMind

## Issue Summary

This issue focuses on adding integration tests for the `RepoIndexer` in `core/indexer/`. The tests will run against real Neo4j and Qdrant containers rather than mocked database services.

The integration test should index a small synthetic repository, verify that a corresponding `File` node is created in Neo4j, confirm that a vector is stored in Qdrant, and remove all test data during teardown.

## Why I Chose This Issue

I chose this issue because it has a clear and manageable scope while allowing me to gain practical experience with integration testing, Docker containers, Neo4j, Qdrant, and pytest.

The project already has unit tests for the indexer, so this contribution provides a focused opportunity to test how the complete indexing workflow behaves when connected to real database services.

My first step will be to study the existing `RepoIndexer` implementation, unit tests, Docker configuration, and contribution guidelines. I will then reproduce the current indexing workflow locally and develop an integration test that verifies data is written correctly to both Neo4j and Qdrant.

## Current Phase

Phase I: Issue Selection

