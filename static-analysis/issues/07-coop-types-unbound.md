# Issue: Unbound variables in cuda.coop kernel code generation

**GitHub issue title:** `[BUG]: UnboundLocalError in cuda.coop._types when struct_name is not Warp or Block`
**Component:** cuda.coop

## GitHub Issue Body

### Is this a duplicate?
- [x] I have searched the open bugs and this is not a duplicate

### Type of Bug
Runtime Error

### Component
cuda.coop

### Describe the bug

In `python/cuda_cccl/cuda/coop/_types.py`, the kernel code generation method sets `provide_alloc_version`, `storage`, and `sync` only inside `if self.struct_name.startswith("Warp")` / `elif self.struct_name.startswith("Block")` branches.

If `struct_name` doesn't match either prefix, these variables are never assigned. Subsequent code at line 730 reads `provide_alloc_version`, which raises `UnboundLocalError`.

```python
if self.struct_name.startswith("Warp"):
    provide_alloc_version = threads == 32
    storage = ...
    sync = "__syncwarp();"
elif self.struct_name.startswith("Block"):
    provide_alloc_version = True
    storage = ...
    sync = "__syncthreads();"
# No else — variables unbound if neither branch taken

if provide_alloc_version:  # UnboundLocalError
    ...
```

### How to Reproduce

Call the method with a struct_name that doesn't start with "Warp" or "Block".

### Expected behavior

A clear `ValueError` indicating the unsupported algorithm type, instead of a confusing `UnboundLocalError`.

### OS
N/A

### nvidia-smi
N/A

### NVCC version
N/A

## PR Title
`Handle unexpected struct_name in cuda.coop kernel codegen`

## PR Body

```markdown
## Description

closes #NNN

Add an `else` clause to raise a clear `ValueError` when `struct_name` does not start with "Warp" or "Block". Previously, three variables (`provide_alloc_version`, `storage`, `sync`) were left unbound, causing an `UnboundLocalError` at runtime instead of a meaningful error message.

Found via pylint (E0606: possibly-used-before-assignment).

## Checklist
- [x] New or existing tests cover these changes.
- [x] The documentation is up to date with these changes.
```
