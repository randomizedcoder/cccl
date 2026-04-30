# Issue: Uninitialized iterator state in c/parallel test utility

**GitHub issue title:** `[BUG]: Uninitialized iterator state member in make_iterator (c/parallel test)`
**Component:** General CCCL

## GitHub Issue Body

### Is this a duplicate?
- [x] I have searched the open bugs and this is not a duplicate

### Type of Bug
Silent Failure

### Component
General CCCL

### Describe the bug

In `c/parallel/test/test_util.h`, the function `make_iterator` (line 917) constructs an `iterator_t<ValueT, StateT>` without initializing the `state` member:

```cpp
iterator_t<ValueT, StateT> it;      // state is indeterminate
it.state_name  = state.name;         // state_name assigned
it.advance     = make_operation(...); // advance assigned
it.dereference = make_operation(...); // dereference assigned
return it;                            // state never assigned
```

When the returned `iterator_t` is later converted via `operator cccl_iterator_t()`, the `state` member's address is taken (`it.state = &state`), pointing to uninitialized memory.

### How to Reproduce

Run cppcheck against `c/parallel/test/test_util.h:924`.

### Expected behavior

The `state` member should be value-initialized. Fix: change `iterator_t<ValueT, StateT> it;` to `iterator_t<ValueT, StateT> it{};`.

### OS
N/A

### nvidia-smi
N/A

### NVCC version
N/A

## PR Title
`Fix uninitialized iterator state in c/parallel test utility`

## PR Body

```markdown
## Description

closes #NNN

Value-initialize `iterator_t` in `make_iterator()` to prevent the `state` member from containing indeterminate values. The `state` field was never assigned after construction, but its address is taken when converting to `cccl_iterator_t` via `operator cccl_iterator_t()`.

The fix changes `iterator_t<ValueT, StateT> it;` to `iterator_t<ValueT, StateT> it{};`, which zero-initializes POD types and default-constructs class types.

Found via cppcheck 2.18.3 ([uninitvar](https://cppcheck.sourceforge.io/manual.pdf)).

## Checklist
- [x] New or existing tests cover these changes.
- [x] The documentation is up to date with these changes.
```
