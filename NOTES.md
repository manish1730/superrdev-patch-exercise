# Engineering Patch Notes

## Summary of Changes
Fixed two high-value production defects across backend SQL and frontend React layers:
1. **BUG-1 (SQL Precedence)**: Grouped `(LOWER(title) LIKE :term OR LOWER(description) LIKE :term)` in `TaskRepository.java`, `search_tasks.sql`, and `task_search_package.sql`. This prevents SQL `AND` operator precedence from causing status filters to be ignored and archived items to leak.
2. **BUG-2 (React Search Race Condition)**: Added `AbortController` lifecycle cleanup in `useTasks.js` and passed `signal` through `api.js` to `fetch()`. Ignores `AbortError` so rapid typing cancels stale in-flight requests and prevents out-of-order API responses from overwriting latest results.

## Deferred Issues
- **BUG-3 (Pagination Reset)**: Search/filter state updates do not reset `page` to 1. Deferred to minimize scope; users on high page numbers may see empty results until resetting manually.
- **BUG-4 (Error State Cleanup)**: `setLoading(false)` is omitted in `useTasks` catch block for non-abort errors. Deferred to avoid expanding UI state architecture.

## Biggest Remaining Risks
- **In-Memory H2 Persistence**: All task state resides in volatile `jdbc:h2:mem:taskdb`, resetting on app restart.
- **Unindexed LIKE Queries**: Full-table wildcard searches (`%term%`) will degrade performance under large datasets without full-text indexing.

## AI & Tool Usage
Developed with Antigravity AI assistant. Antigravity was used to map application flows, diagnose SQL operator precedence, execute API verification tests via `curl`, and implement minimal `AbortController` cleanup in React hooks. All fixes were manually validated against runtime endpoints and production builds.
