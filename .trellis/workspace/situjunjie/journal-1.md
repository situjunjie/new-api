# Journal - situjunjie (Part 1)

> AI development session journal
> Started: 2026-05-22

---



## Session 1: Bootstrap project specs

**Date**: 2026-05-22
**Task**: Bootstrap project specs
**Branch**: `main-gigi`

### Summary

Initialized Trellis backend and frontend specs for the Go/React new-api gateway, including directory, database, error, logging, UI, routing, data-flow, and quality guidelines.

### Main Changes

- Removed `.agents/`, `.codex/`, and `web/default/src/features/usage-logs/data/schema.ts` from Git tracking with `git rm --cached`.
- Preserved the local files while committing their removal from the repository index.
- Pushed the cleanup commit to `origin/main-gigi`.

### Git Commits

| Hash | Message |
|------|---------|
| `5f68699b` | (see git log) |

### Testing

- [OK] Verified the working tree was clean after push.
- [OK] Verified ignored tracked files list was empty with `git ls-files -ci --exclude-standard`.
- [OK] Verified local ignored files were still present.

### Status

[OK] **Completed**

### Next Steps

- None - task complete


## Session 2: 闲聊：查看最近提交

**Date**: 2026-05-22
**Task**: 闲聊：查看最近提交
**Branch**: `main-gigi`

### Summary

无任务会话。用户问候后查看 git log，确认 main-gigi 分支近期主要是 Trellis 接入与 main 同步合并，未做代码变更。

### Main Changes

(Add details)

### Git Commits

(No commits - planning session)

### Testing

- [OK] (Add test results)

### Status

[OK] **Completed**

### Next Steps

- None - task complete


## Session 3: Stop tracking ignored files

**Date**: 2026-06-01
**Task**: Stop tracking ignored files
**Branch**: `main-gigi`

### Summary

Removed ignored local agent/config artifacts from Git tracking while preserving local files, then pushed the cleanup commit to origin/main-gigi.

### Main Changes

(Add details)

### Git Commits

| Hash | Message |
|------|---------|
| `f07acbce` | (see git log) |

### Testing

- [OK] (Add test results)

### Status

[OK] **Completed**

### Next Steps

- None - task complete
