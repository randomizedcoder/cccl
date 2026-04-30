# Issue: Uninitialized members in BuildInformation default constructor

**GitHub issue title:** `[BUG]: Uninitialized members in BuildInformation default constructor (c/parallel test)`
**Component:** General CCCL

## GitHub Issue Body

### Is this a duplicate?
- [x] I have searched the open bugs and this is not a duplicate

### Type of Bug
Silent Failure

### Component
General CCCL

### Describe the bug

In `c/parallel/test/algorithm_execution.h`, the `BuildInformation` class template has a defaulted constructor (`BuildInformation() = default`) that leaves all 6 members uninitialized, including 4 raw pointers:

```cpp
int cc_major;               // indeterminate
int cc_minor;               // indeterminate
const char* cub_path;       // wild pointer
const char* thrust_path;    // wild pointer
const char* libcudacxx_path; // wild pointer
const char* ctk_path;       // wild pointer

BuildInformation() = default;
```

Reading any of these members after default construction is undefined behavior.

### How to Reproduce

Run cppcheck against `c/parallel/test/algorithm_execution.h`.

### Expected behavior

Members should have default initializers: `int cc_major{};`, `const char* cub_path{};`, etc. This zero-initializes ints and null-initializes pointers while preserving the parameterized constructor.

### OS
N/A

### nvidia-smi
N/A

### NVCC version
N/A

## PR Title
`Add default member initializers to BuildInformation`

## PR Body

```markdown
## Description

closes #NNN

Add brace-or-equal-initializers to `BuildInformation` members so the defaulted constructor produces zero-initialized ints and null pointers instead of indeterminate values. The parameterized constructor is unchanged — its member-init-list overrides the default initializers.

```cpp
// Before:
int cc_major;
const char* cub_path;

// After:
int cc_major{};
const char* cub_path{};
```

Found via cppcheck 2.18.3 ([uninitMemberVarPrivate](https://cppcheck.sourceforge.io/manual.pdf)).

## Checklist
- [x] New or existing tests cover these changes.
- [x] The documentation is up to date with these changes.
```
