---
name: python
description: Python code quality standards. Always use this when writing or editing any Python code.
---

# Python Code Quality Spec

- Follow these rules for all new or changed Python code.
- Pre-existing errors in untouched code are out of scope; every line you write or modify must pass linting and strict type checking.
- Always use `uv` when running Python commands (ex: `uv run python` not bare `python`)

## 0. Validate Changed Files

Run, in order, on affected files only:

1. `uv run ruff check --fix <file>`
2. `uv run mypy --strict <file>`
3. `uv run ruff format <file>` after all code changes are complete

Rules:
- Fix mypy errors in the code; do not silence them.
- Bare `# type: ignore` is banned. If unavoidable, include an error code and explanation: `# type: ignore[override]  # covariant return in test stub`.
- Avoid `cast()` unless the type system cannot express the constraint; try `Protocol`, `@overload`, or `TypeVar` first.
- Never format files you did not change.

## 1. Type Everything

- Annotate every parameter and return type, including `-> None` for no return and explicit `bool` returns.
- Use modern syntax: `X | None`, `X | Y`, `list[X]`, `dict[K, V]`, `tuple[X, Y]`; never `Optional`, `Union`, `List`, `Dict`, etc.
- Avoid `Any`; use narrower types, `Protocol`, or `TypeVar`.
- Use `@overload` for input-dependent return types.
- Prefer `Protocol` over ABC inheritance when the interface is the contract.
- Use Python 3.12+ generic syntax: `def first[T](items: Sequence[T]) -> T`.

## 2. Model Schemas, Not Dicts

- If a `dict` has a known schema, replace it with a `dataclass`, `NamedTuple`, or Pydantic `BaseModel`.
- Never pass `dict[str, Any]` between functions as a schema.
- Use `@dataclass(frozen=True, slots=True)` for internal immutable value objects.
- Use `@dataclass` for all Temporal RPC types: workflow inputs/outputs, activity params/results, signals, and queries. Temporal `DataConverter` does not reliably round-trip Pydantic models or `NamedTuple`.
- Use Pydantic `BaseModel` at I/O boundaries: API request/response, config parsing, external data. Do not use it for Temporal RPC types.
- Use `NamedTuple` for lightweight positional records with 2-3 fields.
- Use `TypedDict` only for APIs that require raw dicts.

## 3. Use Literals, Not Magic Strings

- If a string parameter has a fixed set of valid values, use `Literal`.
- Define reusable sets with the `type` keyword: `type LoanStatus = Literal["pending", "approved", "denied"]`.
- Use `StrEnum` only when integrating with an existing enum or when runtime enum methods/iteration are required.
- Do not validate enum membership with `if x not in (...)`; let the type checker do it.

## 4. Use NewType for Domain Identifiers

- Wrap primitive identifiers (`str`, `int`, `UUID`) in `NewType` when they represent distinct domain concepts: `ApplicationId = NewType("ApplicationId", str)`.
- `NewType` uses plain assignment, not the `type` keyword. `type` creates aliases such as `type LoanStatus = Literal[...]`.
- Use `Decimal` for money and financial calculations; never `float`.
- `NewType` is zero-cost at runtime and catches misuse at type-check time.

## 5. Separate Pure Logic from Side Effects

- Pure functions take data in and return data out: no database, HTTP, file system, logging side effects, or I/O imports such as `requests`, `httpx`, `boto3`, `sqlalchemy`.
- Keep orchestrators (workflows, handlers, controllers) thin; they wire I/O to pure logic.
- If a function is hard to test without mocking 3 things, it is doing too much.

## 6. Handle Errors Intentionally

- Never use bare `except:` or `except Exception: pass`.
- Catch the narrowest exception type that makes sense.
- Raise domain-specific exceptions, not generic `ValueError`/`RuntimeError`.
- Do not return `None` to signal failure; use `T | ErrorType`, a `Result` pattern, or raise.
- Chain or intentionally suppress exceptions with `raise X from exc` or `raise X from None`; never bare `raise X`.

## 7. Use Modern Python

- Use f-strings for string formatting.
- Use `match`/`case` for dispatch over structure, status, or tagged unions; prefer it over `if`/`elif` when matching object attributes or multiple conditions.
- Use `type` for type aliases and plain assignment for `NewType`.

## 8. Maintain Async Discipline

- Never call blocking I/O (`requests`, `time.sleep`, sync file I/O) inside `async` functions.
- Use `httpx.AsyncClient`, not `requests`, in async code.
- Use `asyncio.sleep`, not `time.sleep`.
- If blocking code is unavoidable, use `anyio.to_thread.run_sync()` or `asyncio.to_thread()`.

## Code Quality Summary Checklist

For each changed function/module:
- strict mypy passes
- signatures are fully typed
- schemas are modeled
- fixed strings use `Literal`
- domain IDs use `NewType`
- money uses `Decimal`
- pure logic is separated from I/O
- exceptions are narrow and intentional
- syntax is modern
- async code does not block

# Style

- Organize each module in this order: imports, constants, type aliases and classes, then functions.
- Use blank lines to separate logical steps within functions, especially after setup and guard clauses, so each block is easy to scan.
