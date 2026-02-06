---
name: fix-pyright
description: Fix pyright type checking errors in Python files. Use when pyright reports type errors or when asked to fix type issues.
argument-hint: "[optional: file or pattern to check]"
allowed-tools: Bash(uv run pyright:*), Read, Edit, Write, Glob, Grep
---

# Fix Pyright Errors

Fix all pyright type checking errors in the codebase (or in specific files if provided as arguments).

## Running pyright

In uv projects, always run pyright via uv:

```bash
uv run pyright
uv run pyright path/to/file.py
```

## Workflow

1. Run `uv run pyright $ARGUMENTS` to get the current errors.
2. For each error, read the relevant code and fix it following the rules below.
3. Re-run `uv run pyright` to verify all errors are resolved.
4. Repeat until clean.

## Rules

### Never suppress errors with comments

Do NOT add `# type: ignore`, `# pyright: ignore`, or similar suppression comments.

Instead, fix errors using these strategies in order of preference:

1. **Add an assertion** — e.g. `assert x is not None` or `assert isinstance(x, Foo)` to narrow the type.
2. **Add or improve a stub** — create or update `.pyi` files in the `typings/` directory.
3. **Use `cast()`** — as a last resort, use `from typing import cast` and `cast(TargetType, value)`.

### Diffusers imports

For imports from Hugging Face's `diffusers` library, use the correct deep import path that matches the actual module structure. For example:

- `from diffusers.models.autoencoders.autoencoder_kl import AutoencoderKL` (correct)
- `from diffusers import AutoencoderKL` (avoid — may not resolve for pyright)

When adding stubs in `typings/`, mirror the actual package structure (e.g. `typings/diffusers/models/autoencoders/autoencoder_kl.pyi`).

### Avoid `Any`

Do not introduce `Any` as a type annotation. When a type is unclear, prefer in order:

1. **The concrete type** if it can be determined from context.
2. **A `Protocol`** — define a structural protocol in the same file or a shared types module.
3. **A type union** — e.g. `int | float | str` if a small set of types is expected.

When fixing existing stubs in `typings/` that use `Any`, replace with concrete types where feasible (e.g. `torch.Tensor` for tensor arguments).

### Validating Literal values with `get_args`

When validating that a runtime value matches a `Literal` type, use `get_args()` to extract the allowed values from the type alias rather than duplicating them. Define a tuple constant from the Literal, then validate against it:

```python
from typing import Literal, get_args

Mode = Literal["fast", "balanced", "accurate"]
MODES: tuple[Mode, ...] = get_args(Mode)

# Use for validation:
assert mode in MODES, f"Invalid mode: {mode}"
```

This keeps the set of valid values in a single `Literal` definition and avoids drift between the type and runtime checks.
