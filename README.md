# PES-VCS Lab Report

## Student and Repository Details

- **Name:** Ronit  
- **SRN:** PES1UG24CS549  
- **Repository:** https://github.com/Ronit77x/PES1UG24CS549-pes-vcs  
- **Platform Used:** Ubuntu 25.10 (compatible with Ubuntu 22.04 requirement)

---

## Implemented Source Files

The following files were completed as part of the lab:

- `object.c`
- `tree.c`
- `index.c`
- `commit.c`

---

# Screenshot Evidence

## Phase 1 — Object Storage

### 1A - Output of `./test_objects`

![Screenshot 1A](screenshots/1A_test_objects.png)

### 1B - Object Store Sharding Layout

![Screenshot 1B](screenshots/1B_objects_structure.png)

---

## Phase 2 — Tree Objects

### 2A - Output of `./test_tree`

![Screenshot 2A](screenshots/2A_test_tree.png)

### 2B - Raw Tree Object using `xxd`

![Screenshot 2B](screenshots/2B_raw_tree_object.png)

---

## Phase 3 — Index / Staging Area

### 3A - Repository Initialization, Add, Status

![Screenshot 3A](screenshots/3A_init_add_status.png)

### 3B - Contents of `.pes/index`

![Screenshot 3B](screenshots/3B_index_contents.png)

---

## Phase 4 — Commits and History

### 4A - Commit Log with Multiple Commits

![Screenshot 4A](screenshots/4A_commit_log.png)

### 4B - `.pes` Internal File Growth

![Screenshot 4B](screenshots/4B_pes_files.png)

### 4C - HEAD and Branch Reference Chain

![Screenshot 4C](screenshots/4C_head_refs.png)

---

## Final Integration Test

### Output of `make test-integration`

![Integration Test](screenshots/final_integration_test.png)

---

# Analysis Questions

## Q5.1 - How `checkout` Could Be Implemented

To implement `pes checkout <branch>`, the following metadata files would need updates:

- `.pes/HEAD` should point to the selected branch (or direct commit in detached mode)
- `.pes/refs/heads/<branch>` provides the latest commit hash
- `.pes/index` should be refreshed to match the checked-out snapshot

The working directory must then be synchronized with the target tree:

- Create files that exist in target commit
- Update modified tracked files
- Remove tracked files no longer present
- Restore file modes (normal/executable)

The difficult part is safely handling uncommitted user changes.

---

## Q5.2 - Detecting Dirty Working Directory Conflicts

Using only the index and object database:

1. Read target branch tip commit and reconstruct its tree.
2. Read current HEAD snapshot and current index entries.
3. Compare current working files with indexed versions.
4. Any locally modified tracked file is considered dirty.
5. If the same path also differs in the target branch, checkout must be refused.

This prevents overwriting local changes.

---

## Q5.3 - Detached HEAD State

Detached HEAD means `.pes/HEAD` stores a commit hash directly rather than a branch name.

If commits are created in this state:

- New commits are valid
- No branch automatically points to them
- They may later become unreachable

Recovery is simple:

- Create a new branch reference pointing to that commit chain
- Or manually move an existing branch

---

## Q6.1 - Garbage Collection Strategy

A safe garbage collector can use mark-and-sweep:

1. Collect all roots from `.pes/refs/heads/*`
2. Traverse commit history from each root
3. Mark reachable commit, tree, and blob objects
4. Scan `.pes/objects`
5. Delete any unmarked object

Efficient structure: hash set of object IDs for fast lookup.

For 100,000 commits and 50 branches:

- Around 100,000 unique commits may be visited
- Additional tree/blob objects depending on project size
- Total reachable objects may be several hundred thousand+

---

## Q6.2 - Why Concurrent GC Is Risky

Example race condition:

1. Commit process writes new blobs and trees
2. Branch reference not updated yet
3. GC scans refs and sees new objects as unreachable
4. GC deletes them
5. Commit later updates ref to missing objects

This can corrupt repository state.

Git avoids this using:

- Locking mechanisms
- Delayed pruning
- Temporary retention windows
- Conservative reachability checks

---

# Conclusion

This lab recreated the core internal design of Git using C and Linux filesystem APIs.

Key concepts learned:

- Content-addressable storage
- Hash-based integrity verification
- Atomic file operations
- Tree snapshots
- Linked commit history
- Reachability-based cleanup

PES-VCS demonstrates how real version control systems manage data efficiently.

---
